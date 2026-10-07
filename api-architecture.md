# ruby-veterinary API Architecture

> **ruby-veterinary Documentation**
>
> **Document:** API Architecture
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Architecture Standard

---

# Purpose

This document defines how `ruby-veterinary-api` is shaped in code: the module template, the pragmatic clean-architecture/DDD/CQRS/facade/port-adapter conventions, import boundaries, persistence rules, cross-cutting behaviour, testing expectations, and implementation phasing.

It turns `architectural-decision-record.md` (ADR-0002 modular monolith, ADR-0003 schema namespaces, ADR-0006 REST, ADR-0015 Prisma) and `strategic-architecture/06-architecture-rules.md` into a concrete code contract. Where this document and the ADRs disagree, the ADRs win and this document is updated.

**Scope:** backend only. The front end follows `engineering-guidelines.md` (Next.js) and `ui-ux/`.

---

# 1. Architecture Style

One NestJS process. Ten bounded contexts, each an internal Nest module with hard import boundaries. Within a module, four layers are **folders, not packages** — pragmatic, not academic:

| Pattern | How it appears here |
|---------|---------------------|
| Clean architecture | Per-module folders: `domain/ application/ infrastructure/ presentation/`; dependencies point inward, controllers stay thin |
| DDD | Aggregates and value objects **only where invariants exist**; contexts without rich rules skip `domain/` entirely |
| CQRS | `@nestjs/cqrs` command/query split; one database; queries are plain reads through repositories |
| Facade | Every context exposes exactly one `*FacadePort` + `*Facade` — its public API for other modules |
| Port & adapter | Outbound ports for external systems (payments, WhatsApp, email, MinIO) and inbound facade ports wired through in-process adapters |
| Integration events | Cross-context collaboration on an in-process bus; past-tense, idempotent consumers |

**Deliberately not built:** event sourcing, projections, separate read stores, unit of work, outbox framework, microservices. These are absent in both the reference platform and our scale assumptions (single clinic, single VPS, hundreds of SKUs); Rule 7.4 requires an ADR before queues or replicas anyway.

---

# 2. Source Tree

```
ruby-veterinary-api/
├── prisma/
│   ├── schema.prisma               # all 10 @@schema namespaces (ADR-0003/0015)
│   ├── migrations/                 # prisma migrate output, versioned
│   └── seed.ts
├── prisma.config.ts
├── docker-compose.dev.yml          # PostgreSQL 16 for local dev
├── src/
│   ├── main.ts                     # /api/v1 prefix, ValidationPipe, Swagger, CORS
│   ├── app.module.ts               # @Global; imports all context modules
│   ├── config/                     # env schema + validated config (app, db, jwt, minio, whatsapp, payment)
│   ├── prisma/                     # prisma.module.ts (global), prisma.service.ts
│   ├── shared/
│   │   ├── kernel/                 # AggregateRoot, Entity, ValueObject, Result, PrefixedId
│   │   ├── integration/            # IntegrationEvent, IntegrationEventPublisher port,
│   │   │                           # in-process adapter, explorer service
│   │   └── cross-cutting/          # AuthGuard, RolesGuard, decorators, envelope
│   │                               # interceptor, exception filter, pagination,
│   │                               # idempotency, requestId, throttling
│   └── modules/                    # ten bounded contexts (see §3)
│       ├── identity/               # schema: identity
│       ├── clinic-content/         # schema: content
│       ├── publishing/             # schema: publishing
│       ├── care-coordination/      # schema: intake
│       ├── client-patient-records/ # schema: clients
│       ├── commerce/               # schema: commerce
│       ├── pharmacy-authorisation/ # schema: pharmacy
│       ├── care-messaging/         # schema: messaging
│       ├── notifications/          # schema: notifications
│       └── operations/             # schema: operations
└── test/                           # e2e (jest + supertest)
```

Module folder names match `bounded-context.md`; the PostgreSQL schema each owns matches `strategic-architecture/04-ownership-matrix.md`.

---

# 3. Module Template

Every context module follows this shape. Folders are omitted when the context has nothing for them — empty ceremony is not compliance.

```
modules/<ctx>/
├── <ctx>.module.ts                 # DI wiring: providers, facade binding, handler arrays, exports
├── domain/                         # ONLY where invariants exist (see §4)
│   ├── aggregates/
│   └── value-objects/
├── application/
│   ├── commands/                   # <Verb><Noun>Command + handler pair
│   ├── queries/                    # <Verb><Noun>Query + handler pair
│   ├── handlers/                   # @CommandHandler / @QueryHandler (may split commands/ queries/)
│   ├── facades/
│   │   ├── <ctx>-facade.port.ts    # abstract class = the context's public API
│   │   └── <ctx>-facade.ts         # @Injectable impl delegating to the buses
│   ├── ports/                      # outbound ports (external systems) if context-owned
│   └── dto/                        # class-validator DTOs validated at the HTTP boundary
├── infrastructure/
│   ├── repositories/               # concrete @Injectable classes wrapping PrismaService
│   ├── adapters/
│   │   ├── in-process/             # adapters implementing ANOTHER context's facade port
│   │   └── <external-system>/      # payment-gateway/, whatsapp/, email/, minio/
│   ├── mappers/                    # Prisma model → response shape (static classes)
│   └── event-consumers/            # @IntegrationEventHandler classes (idempotent)
└── presentation/
    └── *.controller.ts             # HTTP only: DTO → CommandBus/QueryBus → return
```

### Module wiring (canonical)

```ts
// modules/commerce/commerce.module.ts
const commandHandlers = [PlaceOrderHandler, AddCartItemHandler, /* ... */];
const queryHandlers = [GetCartQueryHandler, ListProductsQueryHandler, /* ... */];

@Module({
  imports: [CqrsModule],
  controllers: [CartController, CatalogController, CheckoutController, OrderController],
  providers: [
    CommerceRepository, /* ...prisma-backed repositories... */
    CommerceFacade,
    { provide: CommerceFacadePort, useExisting: CommerceFacade },
    { provide: PAYMENT_GATEWAY_PORT, useClass: PaymentGatewayRouterAdapter },
    ...commandHandlers,
    ...queryHandlers,
  ],
  exports: [CommerceFacadePort],
})
export class CommerceModule {}
```

Handler arrays are explicit constants — greppable and reviewable; no auto-discovery magic.

### Facade rules

- Exactly one facade pair per context (`<ctx>-facade.port.ts` + `<ctx>-facade.ts`), plus extra facade ports only where a context exposes genuinely distinct surfaces (e.g. a `recording-facade` in the reference platform).
- The facade body is bus delegation only — no business logic:

```ts
@Injectable()
export class IdentityFacade implements IdentityFacadePort {
  constructor(private readonly queryBus: QueryBus, private readonly tokenService: TokenService) {}
  getUserById(id: string) { return this.queryBus.execute(new GetUserByIdQuery(id)); }
}
```

- Other contexts consume the facade **only** through its port, bound with `useExisting` and injected via `@Inject(CommerceFacadePort)` or an equivalent string token.

### Controller rules

- Inject `CommandBus` / `QueryBus` (or a facade for reads owned elsewhere); never repositories.
- Map validated DTO → command/query; return the handler result; no logic beyond HTTP concerns (status codes, headers).
- Swagger decorators (`@ApiTags`, `@ApiOperation`, `@ApiResponse`) on every route — the generated spec gates against `openapi/` in CI.

---

# 4. DDD Building Blocks

## 4.1 Kernel (`shared/kernel/domain/`)

| Base class | Purpose |
|------------|---------|
| `AggregateRoot` | Extends `@nestjs/cqrs` `AggregateRoot`; abstract `id`; integration-event buffer (`addIntegrationEvent` / `pullIntegrationEvents`) |
| `Entity<T>` | Identity-bearing base (`{ id: string }`) |
| `ValueObject<T>` | Immutable, `Object.freeze`, equality by value |
| `Result<T, E>` | `ok()` / `err()` for validation paths that should not throw |
| `PrefixedId` | Generate/parse `usr_`, `ord_`, `rx_`, `apt_`, ... per `api-specification.md` |

## 4.2 Aggregate inventory (exhaustive)

Aggregates exist **only** where invariants must hold under concurrent change. Thin contexts use handlers + repositories directly — this is the reni-verse lesson: dead aggregate classes are worse than no aggregates.

| Aggregate | Context | Invariants it owns |
|-----------|---------|--------------------|
| `Order` | commerce | Status transitions; Rx lines block `paid` until Pharmacy events; money `numeric(10,2)` |
| `Cart` | commerce | One open cart per user; Rx items require `pet_id` before quote succeeds; no price snapshots |
| `Product` | commerce | Stock never negative (`available = stock_quantity - stock_reserved`); variant integrity |
| `Prescription` | pharmacy-authorisation | Vet-only decisions; append-only decision + audit rows (ADR-0012); status transitions |
| `Appointment` | care-coordination | Status transitions; requested-window validity |
| `Intake` | care-coordination | Step completion rules; upload type/size constraints |
| `Conversation` | care-messaging | Bot/handover state machine; emergency keyword detection |
| `Client` | client-patient-records | Identity uniqueness; pet ownership edges |
| `Pet` | client-patient-records | Species/breed/weight sanity; medication flags |

Contexts with **no** `domain/` layer initially: `clinic-content` (published reads), `publishing` (content CRUD), `notifications`, `operations`. They gain one only when an invariant appears.

## 4.3 Value objects

`Money`, `Email`, `PrescriptionStatus`, `OrderStatus`, `AppointmentWindow`, `FileAttachment` (PDF/JPEG, 10 MB), `BusinessHours`. VOs validate in their constructor/factory (`static create(): Result<T, string>`), then the handler converts failures into `BadRequestException` at the edge.

## 4.4 Domain events vs integration events

- **Within a context:** aggregate methods raise domain events via the kernel root when behaviour must trigger *in the same* use case; handlers process them locally. Used sparingly.
- **Across contexts:** **integration events only**, published through the `IntegrationEventPublisher` port (see §7). The event catalogue is `domain-events.md` — one publisher per event, exactly as documented.

---

# 5. CQRS Conventions

| Rule | Detail |
|------|--------|
| Naming | `PlaceOrderCommand` / `PlaceOrderHandler`; `ListProductsQuery` / `ListProductsHandler` |
| Command | Imperative, past-boundary intent; may write; idempotency-key aware when money/records move |
| Query | Read-only; same database; returns DTO/mapper shape; no side effects |
| Handler | One class per command/query; contains the use case; injects repositories + ports + publisher; throws Nest HTTP exceptions with stable error codes |
| Bus | `@nestjs/cqrs` `CommandBus` / `QueryBus` directly — no custom bus wrappers |
| Registration | Explicit arrays in `<ctx>.module.ts` |
| Validation | class-validator on **DTOs only**; commands/queries are plain constructor bags; domain VOs re-validate rules that must survive outside HTTP |

Read/write split is **logical only**: query handlers read the same Prisma-backed tables. No read replicas, no denormalised projections (see §1).

---

# 6. Port & Adapter Inventory

## 6.1 Inbound (cross-context)

| Port | Owner | Consumers (examples) |
|------|-------|----------------------|
| `IdentityFacadePort` | identity | care-coordination, care-messaging, operations |
| `ClientRecordsFacadePort` | client-patient-records | pharmacy-authorisation, commerce (pet claim) |
| `CommerceFacadePort` | commerce | pharmacy-authorisation (order context) |
| `PharmacyFacadePort` | pharmacy-authorisation | commerce (Rx status gate) |
| `CareCoordinationFacadePort` | care-coordination | operations (dashboard reads) |
| `NotificationsFacadePort` | notifications | all contexts raising alerts |

Wiring: consumer module binds `{ provide: X_FACADE_PORT, useClass: XFacadeInProcessAdapter }` where the adapter delegates to the owning context's facade. Ports live in the **consumer's** `application/ports/` or the owner's `application/facades/` as abstract classes; string tokens for multi-implementation cases, abstract classes for single-impl.

## 6.2 Outbound (external systems)

| Port | Adapters | Notes |
|------|----------|-------|
| `PaymentGatewayPort` | stripe/paystack-class adapter + router | ADR-0007; no card data; idempotent webhooks |
| `WhatsAppPort` | WhatsApp Business API adapter | ADR-0008; signature verification; 2s bot budget |
| `EmailPort` | transactional provider adapter | ADR-0013; failures alert, never fail requests |
| `ObjectStoragePort` | MinIO S3 adapter | ADR-0009; signed direct uploads, no file bytes through API |
| `IntegrationEventPublisher` | in-process adapter over `EventBus` | shared kernel port |

External-system ports are **owned where they are used** (e.g. `PaymentGatewayPort` in commerce) unless genuinely shared (`IntegrationEventPublisher`).

---

# 7. Integration Events

- Event classes are past-tense PascalCase (`OrderPlaced`, `PrescriptionApproved`) with a stable `eventName` string (`'commerce.order.placed'`), one per entry in `domain-events.md`.
- Handlers publish **after** the database transaction commits — never inside it. Consumers are **idempotent** (at-least-once semantics; dedupe by event id + handler).
- Delivery: in-process `EventBus` via `IntegrationEventPublisher` port → explorer service subscribes `@IntegrationEventHandler` classes at bootstrap. No external broker (ADR Rule 7.4 gates any queue).
- Handler registration: explicit arrays in the consuming module, same as command handlers.
- Cross-schema propagation: events carry ids and snapshots, never database rows from another schema.

---

# 8. Persistence (Prisma, ADR-0015)

| Rule | Detail |
|------|--------|
| Schema namespaces | One `schema.prisma`, ten `schemas = [...]`, `@@schema` on every model matching the ownership matrix |
| Isolation | No Prisma `@relation` across schemas; cross-context references are plain string id columns |
| Repositories | The **only** place `prisma.*` appears inside a module; concrete `@Injectable` classes |
| Transactions | `prisma.$transaction` for any multi-write touching commerce or pharmacy state; Rx decision + audit rows share one transaction (ADR-0012) |
| Money | `numeric(10,2)` in the database; `Money` VO in code; never floats |
| IDs | DB generates UUIDs; API serialises prefixed opaque strings; never sequential integers |
| Migrations | `prisma migrate` output committed, reviewed, never edited after merge (`database/migrations.md`) |
| Idempotency store | `operations.idempotency_keys` table, added to `data-model/operations.md` when Commerce lands (Phase 3) |

---

# 9. Cross-Cutting Behaviour (`shared/cross-cutting/`)

| Concern | Implementation |
|---------|----------------|
| Base path | `setGlobalPrefix('api/v1')` (Rule 5.1) |
| Validation | Global `ValidationPipe({ transform: true, whitelist: true, forbidNonWhitelisted: true })` |
| Success envelope | Interceptor → `{ data, meta: { requestId, timestamp, pagination? } }` |
| Error envelope | Exception filter → `{ error: { code, message, details[], requestId } }` with stable codes from `api-specification.md` (`VALIDATION_FAILED`, `STOCK_CHANGED`, `PRESCRIPTION_PENDING`, ...) |
| Request id | Generated per request, in logs + envelope; correlation for support |
| Auth | `AuthGuard` (JWT access token, ADR-0011) + `RolesGuard` + `@CurrentUser` / `@Requires` decorators; public routes opt out — public browsing and the emergency surface need no session |
| Idempotency | `Idempotency-Key` middleware on money-moving/record-creating POSTs; 24h replay window; conflict → `409 IDEMPOTENCY_CONFLICT` |
| Rate limiting | `@nestjs/throttler`: public reads 120 req/min/IP; forms 5/10min; auth 10/10min; webhooks exempt from limits but deduped |
| Pagination | Cursor-based for staff lists, offset for public reads; helper decorator |
| Swagger | OpenAPI export at `/docs`; CI diff against `openapi/` |
| Config | `@nestjs/config` with validated schema; no hardcoded URLs or secrets |

---

# 10. Import Boundaries

Enforced by lint (`eslint-plugin-boundaries` or `no-restricted-imports`) and review. These are the hard edges of ADR-0002.

1. **Cross-module imports of internals are forbidden.** `modules/commerce` may never import `modules/identity/domain/...` or any `presentation/`, `application/`, `infrastructure/` file of another context.
2. **Synchronous cross-context calls** go through the target's facade port (DI), never its classes.
3. **Asynchronous cross-context calls** go through integration events (§7), never direct handler-to-handler invocation.
4. **Controllers import only their own module's application layer** (commands, queries, DTOs, own facade) plus shared.
5. **Prisma appears only in `infrastructure/repositories/`** (module-owned) and `src/prisma/`.
6. **`shared/` never imports from `modules/`.** It is dependency-free infrastructure: kernel base classes, integration base types, cross-cutting.
7. **No cross-schema data reads.** A handler reads its own schema; anything else is a facade call or event.

Linter violations of 1, 5, and 6 fail CI.

---

# 11. Testing Strategy

| Level | Scope | Tools |
|-------|-------|-------|
| Unit | Aggregates and VOs (pure); handlers with mocked repositories/ports | Jest |
| Integration | Repositories + transactions against a real PostgreSQL (docker-compose.dev.yml) | Jest + test DB |
| End-to-end | HTTP journeys through the full stack | Jest + supertest |
| Contract | Generated Swagger vs `openapi/` | CI diff |

**Mandatory rejection-path tests:** every change to pharmacy, payments, or emergency-path code ships tests for what it must *refuse* (vet-only enforcement, `STOCK_CHANGED`, missing `Idempotency-Key`, Rx block on `paid`), not only the happy path. The Rx guardrail and checkout idempotency are the two named e2e critical paths.

---

# 12. Implementation Phases

| Phase | Scope | Exit criteria |
|-------|-------|---------------|
| **0 — Contract** (this document) | ADR-0015, `api-architecture.md`, guidelines/README updates | Docs committed; ADR-0004 superseded |
| **1 — Foundation** | `main.ts`, config, shared kernel + cross-cutting, Prisma + 10 namespaces, `docker-compose.dev.yml`, 10 module stubs with facade port stubs, eslint boundaries | `build`/`lint`/`test` green; health endpoint on `/api/v1` |
| **2 — Public reads** | Identity (register/login/refresh/logout/me, JWT, Argon2id, roles) + Clinic Content (clinic, services, staff, hours reads) | Auth e2e green; public reads cacheable, no session required |
| **3 — Money path** | Commerce (cart, quote, checkout, orders) + Pharmacy Authorisation (queue, decisions, audit) | Rx guardrail + checkout idempotency e2e green; OpenAPI diff clean |
| **4 — Care path** | Care Coordination (appointment requests, intake, signed uploads) + Client & Patient Records (clients, pets) | Intake upload flow; pet claim reachable from commerce |
| **5 — Async path** | Care Messaging (WhatsApp webhook, handover) + Notifications + Operations (admin inbox reads, alerts) | 2s bot budget measured; idempotent webhook dedupe tested |
| **6 — Content path** | Publishing (articles, categories, newsletter) | Public blog reads + newsletter subscribe |

Each phase ends with docs-as-contract discipline: behaviour change ⇒ spec update in the same PR (`engineering-guidelines.md`).

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `architectural-decision-record.md` | ADR-0002, ADR-0003, ADR-0006, ADR-0015 — the decisions this shapes |
| `bounded-context.md` | The ten contexts and their integration patterns |
| `strategic-architecture/06-architecture-rules.md` | MUST rules enforced by this contract |
| `strategic-architecture/04-ownership-matrix.md` | Schema ownership per context |
| `api-specification.md` | Envelopes, error codes, idempotency, pagination details |
| `domain-events.md` | Event catalogue (one publisher each) |
| `data-model/` | Tables behind each repository |
| `database/migrations.md` | Migration rules for `prisma migrate` output |
| `engineering-guidelines.md` | Daily practice, review checklist |
| `openapi/` | CI contract for the Swagger export |

---

# Acceptance Criteria

- `ruby-veterinary-api` source tree matches §2 and every context module matches §3 or is an explicit stub
- Exactly one facade pair per context; no cross-module import violates §10 (lint-enforced)
- Aggregates exist only from the §4.2 inventory; no unused aggregate classes are merged
- Repositories are the only `prisma.*` importers; Rx decision + audit share one transaction
- Controllers contain no business logic; every route has Swagger annotations
- The two named e2e critical paths (Rx guardrail, checkout idempotency) have rejection-path tests

---

# Guiding Principle

> **Frameworks are borrowed; boundaries are owned. A module that needs a new edge gets a facade or an event — never an import.**
