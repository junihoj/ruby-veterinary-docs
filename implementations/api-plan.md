# ruby-veterinary API Implementation Plan

> **ruby-veterinary Documentation**
>
> **Document:** API Implementation Plan
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Implementation Plan

---

# Purpose

This document sequences the build of `ruby-veterinary-api` from its current
pristine NestJS 11 scaffold to the full specification: seven product types,
pharmacy authorisation, intake, messaging, and publishing.

It is subordinate to `api-architecture.md` — that document is the binding code
contract; this one is the working execution plan. Where they disagree, the
contract wins and this plan is updated. Behaviour changes still land in the
specification documents first (docs-as-contract).

**Scope:** `ruby-veterinary-api` only. The web front end is a separate plan.

---

# Locked Decisions

| # | Decision | Rationale |
|---|----------|-----------|
| 1 | Persistence is **Prisma 7** + `@prisma/adapter-pg` (ADR-0015 supersedes ADR-0004) | Multi-schema GA; single schema source; typed client |
| 2 | Architecture is a **pragmatic reni-verse subset** of clean/DDD/CQRS/facade/port-adapter — no event sourcing, projections, read store, UoW, or outbox | Complexity budget of a single clinic |
| 3 | Build order: **Foundation → Public reads → Commerce → Care path → Pharmacy → Async path → Content path** | Pharmacy needs pets (Care path); PRD sequences general products (P4) before clinical commerce (P5) |
| 4 | Commerce is staged in **two milestones**: 3a (core) then 3b (composite & digital) | Composite cart lines and downloads are the heavy commerce additions |
| 5 | **All 65 Prisma tables** are created in Foundation with one initial migration | Schema completeness up front; behaviour fills in per milestone |
| 6 | Products: 5 structural types (`simple\|variable\|grouped\|external\|composite`) + `is_virtual` / `is_downloadable` flags; **bundle folded into composite** (configurable slots, WooCommerce-style) | One structural taxonomy; bundles and composites share cart mechanics |
| 7 | Subscription billing uses an **in-process `@nestjs/schedule` timer**; no ADR (Rule 7.4 covers queues/replicas, not in-process timers) | Smallest thing that works |
| 8 | Phase 3a ships a **mock payment adapter** behind `PaymentGatewayPort`; the real provider adapter lands when sandbox credentials exist | ADR-0007 was always provider-swappable |
| 9 | Status/enum-like columns are **`text` + application VOs** (not PostgreSQL enum types), following the table-level authority in `data-model/` | Value sets evolve without DDL; `database-architecture.md` guidance is satisfied by VOs at the edge (CHECK constraints can be added later if wanted) |

---

# Phase Roadmap

Mirrors `api-architecture.md` §12 (amended by Milestone 0).

| Phase | Scope | Exit criteria |
|-------|-------|---------------|
| **0 — Contract** | Amend `api-architecture.md` §12 to this table; fix three phase cross-references | Docs committed; gitlink bumped |
| **1 — Foundation** | `main.ts`, config, shared kernel + cross-cutting, Prisma + 10 namespaces + all 65 tables, `docker-compose.dev.yml` (postgres + minio), 10 module stubs with facade port stubs, eslint boundaries | `build`/`lint`/`test` green; health on `/api/v1`; fresh `migrate deploy` on empty DB |
| **2 — Public reads** | Identity (register/login/refresh/logout/me, JWT, Argon2id, roles) + Clinic Content (clinic, services, staff, hours) | Auth e2e green; public reads cacheable, no session required |
| **3a — Money path (core)** | Commerce catalog (all 7 type shapes: reads + admin CRUD), cart/quote/checkout for simple/variable/grouped/external/virtual, payments port + mock adapter, orders (ship/pickup), idempotency store; Rx items blocked at cart (`422 PRESCRIPTION_PENDING`) until Phase 5 | Store transacts end-to-end for general products; checkout idempotency e2e green; OpenAPI diff clean |
| **3b — Money path (composite & digital)** | Composite cart lines (slots, fixed/sum pricing, component stock), downloadable purchases (MinIO signed URLs, digital fulfilment, grants on payment capture), subscriptions | Composite + digital e2e green |
| **4 — Care path** | Care Coordination (appointment requests, intake, signed uploads) + Client & Patient Records (clients, pets) + `EmailPort` console adapter for form routing | Intake upload flow; pet claim reachable from commerce |
| **5 — Pharmacy** | Pharmacy Authorisation (queue, decisions, audit, patient association); **lifts the 3a Rx block**; Rx orders release to `paid` | Rx guardrail e2e green; no path moves Rx orders to `paid` without Pharmacy events |
| **6 — Async path** | Care Messaging (WhatsApp webhook, handover) + Notifications + Operations (admin inbox, alerts) | 2s bot budget measured; idempotent webhook dedupe tested |
| **7 — Content path** | Publishing (articles, categories, newsletter) | Public blog reads + newsletter subscribe |

Sequencing facts that drive this order:

- Grouped and external are catalog-only: grouped members cart as normal lines;
  external links never enter the cart. Only **composite** changes cart structure.
- The interim Rx guardrail (422 at cart-add) is a temporary build guard, not
  specified behaviour; Phase 5 replaces it with the real prescription hold.
- Pharmacy cannot start before Phase 4: patient association needs `clients.pets`.

---

# Milestone 0 — Contract Amendment

Docs repo only, one commit.

1. Replace the phase table in `api-architecture.md` §12 with the roadmap above.
2. Fix phase cross-references in the same commit:
   - `api-architecture.md` (idempotency store row): "(Phase 3)" → "(Phase 3a)"
   - `architectural-decision-record.md` (ADR-0008 consequences): "Phase 3
     critical path" → "Phase 6 critical path"
   - `prd.md` (risk table, WhatsApp row): "Phase 3 slips" → "Phase 6 slips"
3. Commit `docs(arch): reorder implementation phases (commerce before pharmacy)`,
   push, bump the parent gitlink.

---

# Milestone 1 — Foundation (`ruby-veterinary-api`)

Tasks in order:

1. **Dependencies** (pnpm): `prisma`, `@prisma/client`, `@prisma/adapter-pg`,
   `@nestjs/cqrs`, `@nestjs/config`, `@nestjs/swagger`, `@nestjs/throttler`,
   `@nestjs/jwt`, `@nestjs/schedule`, `class-validator`, `class-transformer`,
   `argon2`; dev: `eslint-plugin-boundaries`.
2. **Prisma:** `prisma.config.ts`; one `schema.prisma` with
   `schemas = ["identity", "content", "publishing", "intake",
   "clients", "commerce", "pharmacy", "messaging",
   "notifications", "operations"]` (names per `strategic-architecture/04-ownership-matrix.md`);
   all 65 models converted from
   `data-model/*.md` (`@@schema`, `@@map`, uniques, indexes, plain string ids,
   no cross-schema `@relation`); partial indexes and check constraints that
   Prisma schema cannot express go in the migration as raw SQL; initial
   migration; seed skeleton (roles, facets, clinic hours).
3. **`docker-compose.dev.yml`:** postgres:16 + MinIO, healthchecks, volumes.
4. **`src/config/`:** validated env (`DATABASE_URL`, `PORT`, JWT secrets
   required; provider keys optional).
5. **`src/prisma/`:** global `PrismaService` with lifecycle hooks.
6. **`src/shared/kernel/`:** `AggregateRoot` (integration-event buffer),
   `Entity`, `ValueObject`, `Result`, `PrefixedId` generator.
7. **`src/shared/integration/`:** `IntegrationEvent` base, publisher port +
   in-process adapter, bootstrap explorer subscribing handlers.
8. **`src/shared/cross-cutting/`:** `AuthGuard` + `@Public`, `RolesGuard` +
   `@CurrentUser`, response-envelope interceptor, exception filter with stable
   error codes, request id, pagination helper, throttler wiring.
9. **`main.ts`:** `setGlobalPrefix('api/v1')`, global `ValidationPipe`
   (transform, whitelist, forbidNonWhitelisted), Swagger at `/docs`, CORS.
10. **Ten module stubs** (`identity`, `clinic-content`, `publishing`,
    `care-coordination`, `client-patient-records`, `commerce`,
    `pharmacy-authorisation`, `care-messaging`, `notifications`, `operations`),
    each with facade port abstracts per `api-architecture.md` §6.1; wired in
    `app.module.ts`.
11. **ESLint boundaries:** no cross-module imports; Prisma only in
    `infrastructure/repositories/`; `shared/` never imports modules —
    violations fail lint.
12. **Tests:** kernel unit tests (`Result`, `PrefixedId`, value objects) +
    health e2e against the compose database.

Exit: `pnpm lint && pnpm build && pnpm test` green; fresh `migrate deploy` on
an empty database; `GET /api/v1/health` returns DB status.

---

# Milestones 2–7 — Summaries

## M2 — Public reads

Identity (5 auth ops: rotating refresh, Argon2id, additive role/permission
claims) and Clinic Content (7 sessionless reads, CDN-cacheable headers).
Rejection-path e2e: wrong password, expired token, refresh reuse.

## M3a — Money path (core)

Catalog reads + admin CRUD for all seven product-type shapes (composite PDP
renders slots; add-to-cart disabled until 3b); facets; cart/quote/checkout for
simple/variable/grouped/external/virtual lines with per-variant stock
reservation; `PaymentGatewayPort` + deterministic mock adapter + payments
webhook (signature verify, idempotent); orders, ship/pickup fulfilment;
temporary Rx block (422 `PRESCRIPTION_PENDING` at cart-add); idempotency
middleware + `operations.idempotency_keys` (table added to
`data-model/operations.md` in the same commit). Pet claim not yet required
(Rx lines blocked).

## M3b — Money path (composite & digital)

Composite cart lines: slot min/max validation, `fixed`/`sum` pricing with
`price_override`, component stock reservation, `order_item_components`
snapshot, pick list; derived Rx flags (OR of slot options — still blocked).
Downloadable purchases: staff file upload via signed URL (ADR-0009), digital
fulfilment when every line is virtual/downloadable, `product_download_grants`
issued on `PaymentCaptured`, `GET /orders/:id/downloads` +
`POST /downloads/:grantId/url`. Subscriptions: create/pause/cancel, physical
simple/variable only, in-process timer for due `next_order_at`.

## M4 — Care path

Appointment requests, intake submissions, history uploads (signed URLs),
clients/pets records with staff CRUD; `ClientRecordsFacadePort.getPet` for M5;
`EmailPort` + console adapter so form submissions and alerts route without
real SMTP. Events: `AppointmentRequested`, `IntakeSubmitted`,
`HistoryUploadReceived`.

## M5 — Pharmacy

Prescription request lifecycle and queue, veterinarian-only decisions
(`PRESCRIBER_REQUIRED`), append-only audit enforced at database level
(ADR-0012), patient association, event flow releasing Rx orders to `paid`.
Lifts the M3a Rx block. Both named e2e critical paths fully green: Rx
guardrail (rejection paths) and checkout idempotency.

## M6 — Async path

WhatsApp webhook (signature verification, message-id dedupe, keyword rules,
2s bot budget measured), human handover inbox, notifications delivery
(dashboard + email adapters), operations audit and dashboard reads.
Webhooks exempt from rate limits but deduped.

## M7 — Content path

Publishing: articles, categories, tags, newsletter subscribe; SEO metadata;
publish events. Public reads cacheable per Rule 7.2.

---

# Execution Conventions

- **Commits:** one per milestone in `ruby-veterinary-api`, conventional
  commit style, pushed to main; parent gitlink bumped (docs repo follows the
  same pattern).
- **Gates every milestone:** `pnpm lint && pnpm build && pnpm test` plus the
  milestone's e2e suite; `prisma migrate deploy` from scratch; OpenAPI export
  diffed against `openapi/` from M2 on.
- **Docs-as-contract:** any behaviour change updates the specification in the
  same commit (e.g. M3a adds the idempotency table to
  `data-model/operations.md`).
- **Rejection-path tests** are mandatory for pharmacy, payments, and
  emergency-path changes (`api-architecture.md` §11).

---

# Deferred / Out of Scope

| Item | Where it lands |
|------|----------------|
| Real payment provider adapter + sandbox credentials | After M3a, when credentials exist |
| WhatsApp Business API real credentials | M6 ships against the adapter port; live wiring behind account approval (prd.md risk) |
| Real SMTP for `EmailPort` | Console adapter until SMTP provider chosen |
| Outbound webhooks to practice-management partners | Deferred per `api-specification.md` |
| Web front end | Separate plan for `ruby-veterinary-web-frontend` |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `api-architecture.md` | Binding code contract; §12 phases are amended by M0 of this plan |
| `architectural-decision-record.md` | ADR-0002, ADR-0003, ADR-0007, ADR-0009, ADR-0011, ADR-0012, ADR-0015 |
| `api-specification.md` | Envelopes, error codes, idempotency, pagination |
| `data-model/` | Source of the 65-table Prisma conversion |
| `openapi/` | CI contract for the Swagger export |
| `database/migrations.md` | Migration rules for `prisma migrate` output |
| `engineering-guidelines.md` | Daily practice, commit conventions, review checklist |
| `prd.md` | Product phasing (P1–P5) this plan implements |

---

# Acceptance Criteria

- Every milestone's exit criteria is met before the next milestone starts
- The two named e2e critical paths (Rx guardrail, checkout idempotency) have
  rejection-path tests before Phase 5 is called done
- No milestone merges code that violates `api-architecture.md` §10 import
  boundaries
- The parent repository gitlink is bumped in the same push as each docs commit

---

# Guiding Principle

> **The contract decides what; this plan decides when. Sequence is a tool —
> safety guardrails never wait for theirs.**
