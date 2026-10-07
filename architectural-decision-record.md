# architectural-decision-record.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Architecture Decision Records
>
> **Version:** 1.1.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Architecture Standard

---

# Purpose

This document records the architectural decisions that shape ruby-veterinary: what was decided, what was considered instead, and why. Decisions are recorded once, here, and never re-argued informally.

Together with `bounded-context.md` (boundaries) and `api-specification.md` (contracts), this is the technical source of truth for the implementation repositories.

---

# ADR Principles

- ADRs are immutable once accepted: supersede them with a new ADR rather than editing history
- Every ADR states context, decision, alternatives, rationale, and consequences
- Decisions with measurable reversal conditions state when to revisit them
- Product-level trade-offs belong in `prd.md`; ADRs are for technical shape

# ADR Status Definitions

| Status | Meaning |
|--------|---------|
| **Proposed** | Written, awaiting confirmation |
| **Accepted** | In force; implementation must follow |
| **Superseded** | Replaced by a later ADR (reference it) |
| **Deprecated** | No longer applicable, not yet replaced |

# ADR Template

```
# ADR-NNNN: <Decision Title>

## Context
## Decision
## Alternatives Considered
## Rationale
## Consequences
## Revisit When
```

---

# ADR Index

## ADR-0001 - Record Architecture Decisions

**Status:** Accepted

- **Decision:** Significant architecture choices are recorded as ADRs in this file using the template above.
- **Rationale:** The clinic site has a long life and will outlive the people who built it; decisions must remain explainable.

## ADR-0002 - Modular Monolith as Initial Architecture

**Status:** Accepted

- **Decision:** The API is a single NestJS application whose bounded contexts are enforced as internal modules with hard import boundaries. No microservices.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Microservices | Operational cost and distributed failure modes are unjustified for a single clinic |
| Serverless functions only | Prescription and order flows need transactional consistency across contexts |
| Monolith without module rules | Boundaries erode; extraction later becomes a rewrite |

- **Rationale:** A single deployable meets the 99.9% uptime goal with the least moving parts, while module boundaries preserve the option to split later.
- **Consequences:** All contexts share one process and one database; schema namespaces and module ownership rules in `bounded-context.md` must be enforced by lint and review.
- **Revisit When:** A context's scaling needs measurably diverge from the rest, or a module requires an independent release cadence.

## ADR-0003 - PostgreSQL 16+ as the Primary Database

**Status:** Accepted

- **Decision:** PostgreSQL 16 or later is the single system of record, with one database and per-context schema namespaces.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| MySQL | Fine technically; PostgreSQL chosen for JSONB flexibility, row-level locking, and consistency with the team's existing platform |
| SQLite | No concurrent write safety for a live storefront |
| Document store | Client, pet, order, and prescription data is relational by nature |

- **Rationale:** Relational integrity is a clinical safety requirement (pet to owner to prescriber to order), and PostgreSQL provides strong consistency for commerce plus room for JSONB where documents vary.
- **Consequences:** Migrations are version-controlled (see `database/migrations.md`); cross-context foreign keys are prohibited by convention.

## ADR-0004 - TypeORM with Repository Pattern

**Status:** Superseded by ADR-0015

- **Decision:** TypeORM is the data access layer, used only through domain repositories injected by interface (`{ provide: 'ClientRepository', useClass: TypeOrmClientRepository }`). Entities never leak past a context boundary.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Prisma | Strong DX, but the repository/injectable pattern maps more naturally onto NestJS modules and keeps SQL control for performance-critical catalog queries |
| Raw SQL | Loses type safety and migration tooling |
| Sequelize | Weaker NestJS integration |

- **Rationale:** Aligns with the existing NestJS house style already used across the organisation's services, and repository interfaces keep domains swappable.
- **Consequences:** Repositories are the only place TypeORM is imported; direct `Repository<T>` injection outside a module's own repository classes is a review failure.

## ADR-0005 - Next.js App Router on the Front End

**Status:** Accepted

- **Decision:** `ruby-veterinary-web-frontend` uses Next.js 16 App Router with React 19, Tailwind CSS 4, and TypeScript, served from the single VPS behind the Cloudflare CDN (ADR-0014). Marketing and service pages are statically rendered and cached at the edge; authenticated and commerce routes render dynamically.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| SPA only (Vite/React) | Loses SEO for the blog and service catalog, and worsens the under-3s budget on 4G |
| Separate admin SPA + public site | Doubles the build and design surface for a single clinic |

- **Rationale:** Static caching is the cheapest way to hold the sub-3s page budget, and one codebase serves public site, storefront, and staff back office.
- **Consequences:** Client-side code must never hold secrets; all privileged operations call the API with a bearer token.

## ADR-0006 - REST API First

**Status:** Accepted

- **Decision:** The front end and any future clients integrate through versioned REST endpoints over HTTPS with JSON bodies. Contracts are described in `api-specification.md` and eventually `openapi/`.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| GraphQL | Query flexibility is unnecessary; adds caching and authorisation complexity |
| tRPC | Couples the client to the framework; REST keeps third-party integrations (webhooks, future partners) simple |

- **Rationale:** REST is the lowest-friction contract for a two-consumer system and is trivially documented for external integrators.
- **Consequences:** Error model, pagination, and idempotency conventions are mandatory (defined in `api-specification.md`).

## ADR-0007 - Tokenised Payment Gateway, No Card Data On Our Side

**Status:** Accepted

- **Decision:** Checkout uses a mainstream tokenised gateway (Stripe or PayPal class) with hosted or SDK-based card collection. Card numbers never touch clinic infrastructure, keeping the platform at PCI-DSS SAQ-A scope.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Self-hosted card form posting to our API | Drives PCI-DSS Level 1 scope onto a small clinic's servers |
| Manual phone payment only | Abandons the online revenue objective |

- **Rationale:** Required by `non-functional-requirements.md` (Payment Security) and cheapest to operate safely.
- **Consequences:** Payment intents are created server-side; the API stores gateway references only; webhooks are signature-verified and idempotent.

## ADR-0008 - Verified WhatsApp Business API for Medical Context

**Status:** Accepted

- **Decision:** All WhatsApp messaging that may carry medical context or prescription requests goes through the official verified WhatsApp Business API. A multi-agent shared inbox tool fronts it for receptionists.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Unofficial personal WhatsApp automation | Violates platform terms; risk of number bans mid-emergency |
| SMS only | Loses the menu and conversation experience the requirements specify |

- **Rationale:** Compliance requirement in `non-functional-requirements.md` (WhatsApp Integrity), and the API is what makes webhooks and bot latency targets achievable.
- **Consequences:** Business verification lead time sits on the Phase 3 critical path (tracked in `prd.md` risks).

## ADR-0009 - Object Storage for Uploads and Media

**Status:** Accepted

- **Decision:** Client medical-history uploads (PDF/JPEG) and CMS media are stored in object storage, uploaded via short-lived signed URLs issued by the API, never through the API's own request body. Object storage runs as MinIO on the single VPS (ADR-0014) behind the S3 API, in private buckets.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| External S3 provider (R2/B2 class) | Monthly cost before launch; MinIO's S3-compatible API keeps this a config change later if scale or compliance demands it |
| Flat files on local disk | No bucket semantics, no signed URLs, retention unenforceable |
| Passing files through the API | Wastes bandwidth and memory; unnecessary attack surface |

- **Rationale:** Signed direct uploads keep large binaries out of the application process and make retention rules enforceable at the bucket level; the S3 API preserves an exit path without code changes.
- **Consequences:** The MinIO volume is part of the off-site backup scope in `database/backup-and-recovery.md`; buckets are private with per-object access control.

## ADR-0010 - Self-Built Staff Back Office Instead of an External CMS

**Status:** Accepted

- **Decision:** The blog CMS, catalog management, pricing tools, and prescription review queue are built as authenticated routes inside the existing Next.js app talking to the NestJS API. Content is stored in PostgreSQL. No third-party headless CMS.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Headless CMS (Sanity, Strapi class) | Recurring cost, another vendor, and prescription/catalog logic does not live there anyway |
| WordPress hybrid | Two stacks, weaker typing, security surface for a clinic |

- **Rationale:** The no-code requirement is about staff UX, not about buying a CMS; one codebase, one deploy, and data stays in the same system of record as orders and patients.
- **Consequences:** Rich text editing, image optimisation (WebP), and SEO fields are features we own and must test ourselves.

## ADR-0011 - JWT Session Tokens with Role Claims

**Status:** Accepted

- **Decision:** Authentication issues short-lived access tokens with role claims (additive staff roles plus a client role) and rotating refresh tokens. Public browsing needs no session at all.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Server-side sessions only | Extra infrastructure for a two-app architecture; acceptable alternative if rotation is complex |
| OAuth social login as primary | Pet owners expect email/password; social login may be added later as secondary |

- **Rationale:** Stateless verification suits a CDN-fronted Next.js app; role claims keep authorisation decisions cheap and centralised.
- **Consequences:** Emergency-facing pages must render without any session; token revocation relies on short lifetimes plus refresh rotation.

## ADR-0012 - Append-Only Audit Trail for Clinical Decisions

**Status:** Accepted

- **Decision:** Every prescription authorisation decision and admin action that alters clinical or financial state is written to an append-only audit log with actor, timestamp, and reference. Audit rows are never updated or deleted.

- **Rationale:** Prescription decisions carry liability; an immutable trail is the cheapest defence and the only honest answer to "who approved this".
- **Consequences:** Audit writes participate in the same transaction as the decision; retention is defined in `database/backup-and-recovery.md`.

## ADR-0013 - Third-Party Transactional Email Delivery

**Status:** Accepted

- **Decision:** All outbound email (form routing, alerts, newsletter) is sent through a dedicated transactional email provider behind a single internal notification interface.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| SMTP from the API host | Deliverability and reputation problems; no analytics |
| Build our own newsletter platform | Out of scope; embed and dispatch are enough |

- **Rationale:** The notification interface is the seam; the provider remains swappable, satisfying graceful degradation if it fails.
- **Consequences:** Failed deliveries raise alerts and surface in the admin dashboard rather than failing user requests.

## ADR-0014 - Single VPS with Docker Compose

**Status:** Accepted

- **Decision:** Everything runs on one Ubuntu VPS: Docker Compose orchestrates nginx, the Next.js frontend, the NestJS API, PostgreSQL, and MinIO. Cloudflare fronts the server for DNS, CDN caching, and edge TLS. CI deploys over SSH; PostgreSQL and MinIO volumes are backed up daily to an off-site S3-compatible bucket. Full topology in `deployment-architecture.md`.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Managed PaaS (separate frontend and API platforms) | Per-service pricing is disproportionate for a single clinic; two platforms to secure instead of one |
| Kubernetes | Operational burden with no scaling justification at this scale |
| Multiple VPS / HA pair | Cost and complexity; single-VPS risk is accepted and mitigated by off-site restore instead of failover |
| Serverless-only | Transactional checkout and prescription flows need a persistent, stateful core |

- **Rationale:** One box with self-restarting containers, external monitoring, and proven off-site recovery is the simplest topology that can credibly pursue the 99.9% NFR on a small-team budget.
- **Consequences:** Planned maintenance counts against uptime; a host failure means restore-to-new-VPS (RTO ≤ 4 hours), not automatic failover; secrets live in the VPS `.env`, never in the repository.
- **Revisit When:** Traffic outgrows vertical resizing, or availability requirements demand automatic failover.

## ADR-0015 - Prisma 7 with Driver Adapters (Supersedes ADR-0004)

**Status:** Accepted

**Supersedes:** ADR-0004

## Context

ADR-0004 chose TypeORM with string-token repositories before any API code existed. Since then three facts changed:

1. The organisation's reference platform (`reni-verse-api`) standardised on Prisma with the `@prisma/adapter-pg` driver adapter, and its architecture patterns (module-per-context, CQRS command/query split, context facades, port/adapter for external systems) are the style adopted by `api-architecture.md`.
2. Prisma's `multiSchema` feature reached General Availability (Prisma 6.13.0), so ADR-0003's per-context schema namespaces map directly to `schemas = [...]` in the datasource and `@@schema("...")` on each model — no preview flag.
3. `ruby-veterinary-api` is still a pristine NestJS 11 scaffold with zero persistence code. Switching now is free; after Phase 1 it would be a rewrite.

## Decision

- **Prisma 7** with the **`@prisma/adapter-pg`** driver adapter is the data access layer for `ruby-veterinary-api`.
- One `prisma/schema.prisma` declares **all ten per-context schema namespaces** from ADR-0003 (`schemas = ["identity", "content", "publishing", "intake", "clients", "commerce", "pharmacy", "messaging", "notifications", "operations"]`), with `@@schema(...)` on every model.
- A single `PrismaService` (module `src/prisma/`) owns the client lifecycle and is provided globally.
- **Repositories are concrete `@Injectable` classes wrapping `PrismaService`**, one per repository concern, living in `modules/<ctx>/infrastructure/repositories/`.
- Direct `prisma.*` usage outside a module's own repository classes is a review failure (the Prisma equivalent of ADR-0004's TypeORM rule).
- ADR-0004 is marked **Superseded** by this ADR. ADR-0003 is unchanged and implemented via `@@schema`.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Stay on TypeORM | Honours ADR-0004 literally, but `reni-verse-api` and this decision now share one persistence style; ADR-0004's original "Prisma is weaker" concern was pre-multiSchema-GA, and `ruby-veterinary-api` has no TypeORM code to preserve. ADR immutability is satisfied by superseding, not editing. |
| Drizzle ORM | Less mature NestJS integration; no org experience to build on |
| Raw SQL via `pg` | Loses type safety, migration tooling, and the repository/mapper patterns shared with `reni-verse-api` |
| TypeORM now, Prisma later | Two migrations instead of one; the later one would still be a rewrite |

## Rationale

One persistence stack across the organisation's NestJS services means the repository, mapper, and adapter patterns transfer directly; Prisma's generated client gives end-to-end type safety from schema to handler; `prisma migrate` covers the versioned-migration requirement in `database/migrations.md`; and multi-schema keeps the hard context boundaries of ADR-0002/0003 at the database level.

## Consequences

- The API depends on `prisma`, `@prisma/client`, and `@prisma/adapter-pg`; configuration moves into `prisma.config.ts` plus `DATABASE_URL`.
- Migrations are produced by `prisma migrate` but must still satisfy `database/migrations.md`: versioned, reviewed, never edited after merge, no manual production DDL.
- IDs: the database generates UUIDs (Prisma default); controllers serialise prefixed opaque strings (`usr_`, `ord_`, `rx_`, ...) per `api-specification.md`.
- Cross-context relations must **not** use Prisma `@relation` across schemas; they are plain string id columns per ADR-0003's no-cross-schema-FK rule.
- If a model name collides across schemas, the Prisma model name is disambiguated and mapped to the documented table with `@@map`.
- The string-token provider pattern (`{ provide: 'ClientRepository', ... }`) from ADR-0004 is no longer the convention; repositories are injected as concrete classes, facades and external-system ports keep DI tokens.

## Revisit When

A context needs sustained hand-tuned SQL beyond what `$queryRaw` inside its own repository can express, or a Prisma release regresses driver-adapter or multi-schema behaviour. Either condition warrants a new ADR, not an informal switch.

---

# ADR Governance

- New ADRs are proposed in a pull request touching only this file (and the README if the index changes)
- An ADR affecting product scope also requires `prd.md` to be updated in the same change
- Superseded ADRs remain in this document with a pointer to their replacement

---

# Acceptance Criteria

- Every technical choice listed in `prd.md` section 8 maps to an accepted ADR
- Every ADR has a status, and no accepted ADR lacks a rationale
- Any implementation deviation from an accepted ADR triggers a new ADR or a supersede

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `api-architecture.md` | How ADR-0002, ADR-0003, and ADR-0015 are shaped into code |
| `bounded-context.md` | Context boundaries this architecture enforces |
| `api-specification.md` | Contract produced by ADR-0006 |
| `engineering-guidelines.md` | How these decisions are applied in daily work |
| `database/` | Persistence, migrations, performance, and recovery consequences |
| `deployment-architecture.md` | Operational form of ADR-0014: VPS topology, CI/CD, monitoring |
| `prd.md` | Product decisions and the open VPS provider question |

# Guiding Principle

> **Write the decision down while the alternatives are still fresh. A decision that cannot be reconstructed from its context was never really made.**
