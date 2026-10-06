# architectural-decision-record.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Architecture Decision Records
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

**Status:** Accepted

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

- **Decision:** `ruby-veterinary-web-frontend` uses Next.js 16 App Router with React 19, Tailwind CSS 4, and TypeScript, deployed as an edge/CDN-rendered application. Marketing and service pages are statically rendered and cached; authenticated and commerce routes render dynamically.

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

- **Decision:** Client medical-history uploads (PDF/JPEG) and CMS media are stored in object storage, uploaded via short-lived signed URLs issued by the API, never through the API's own request body.

### Alternatives Considered

| Option | Why Not |
|--------|---------|
| Local disk on the API host | Lost on redeploy, unscalable, complicates backups |
| Passing files through the API | Wastes bandwidth and memory; unnecessary attack surface |

- **Rationale:** Signed direct uploads keep large binaries out of the application process and make retention rules enforceable at the bucket level.
- **Consequences:** Buckets are private with per-object access control; retention and deletion policies are defined in `database/backup-and-recovery.md`.

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

## ADR-0014 - Provider-Agnostic Managed Hosting with SLA

**Status:** Proposed

- **Decision:** The frontend deploys to a managed Next.js platform and the API to a managed container platform, both with a 99.9% availability commitment, infrastructure as code, and CI-driven deploys. The specific providers are an open question in `prd.md` section 14.

- **Rationale:** Uptime is a contractual NFR; managed platforms are the only realistic way a small team meets it.
- **Consequences:** Environment parity through containers; secrets live in the platform's secret store, never in the repository.
- **Revisit When:** Provider selection is confirmed by the practice manager.

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
| `bounded-context.md` | Context boundaries this architecture enforces |
| `api-specification.md` | Contract produced by ADR-0006 |
| `engineering-guidelines.md` | How these decisions are applied in daily work |
| `database/` | Persistence, migrations, performance, and recovery consequences |
| `prd.md` | Product decisions and the hosting open question |

# Guiding Principle

> **Write the decision down while the alternatives are still fresh. A decision that cannot be reconstructed from its context was never really made.**
