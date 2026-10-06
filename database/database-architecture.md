# database-architecture.md

> **ruby-veterinary Documentation**
>
> **Document:** Database Architecture
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

This document defines the database architecture for ruby-veterinary: persistence philosophy, data ownership, transaction boundaries, consistency model, and scaling path.

It establishes **how** data is stored. It does not describe individual tables or columns, which belong in `data-model/` once written, or the entities themselves, which belong in `../domain-model.md`.

---

# Architectural Principles

## Domain Ownership

Every bounded context owns its data exclusively. Examples:

- Commerce owns products, carts, orders, and payments
- Pharmacy Authorisation owns prescription requests and decisions
- Client & Patient Records owns clients, pets, and history references
- Publishing owns articles, categories, and tags

No context writes to another context's tables; integration is through module interfaces or events.

## Single Source of Truth

Every piece of business data has exactly one authoritative owner. Client contact details live once (Client & Patient Records); Commerce and Care Coordination reference them by id and resolve through the owning context.

## Integrity Over Convenience

Constraints enforce invariants at the database, not only in application code: enums for status columns, `CHECK` constraints for monetary non-negativity, `NOT NULL` wherever the spec implies completeness, unique indexes for natural keys (email per client, slug per article, SKU per variant).

## Files Are Not Rows

Medical-history PDFs/JPEGs and CMS media live in object storage (ADR-0009). The database stores object keys, content type, size, and ownership - never file bytes.

---

# Storage Technology

| Store | Holds | Notes |
|-------|-------|-------|
| PostgreSQL 16+ | All structured business data | One instance on the VPS, schema-per-context |
| MinIO object storage | History uploads, CMS media | On the VPS behind the S3 API (ADR-0009/0014); private buckets, signed URLs, volume exported to the offsite backup bucket |
| Cloudflare CDN | Rendered public pages, images | Served in front of the VPS; not a data store; invalidated on publish |
| Application memory | Session state, hot catalog reads | Loss is always safe |

Deferred until measured need: Redis for caching and background queues (see `README.md`, Future Evolution).

---

# Schema Organization

```
PostgreSQL: ruby_veterinary
├── schema identity        # users, roles, sessions
├── schema content         # services, staff profiles, hours, documents
├── schema publishing      # articles, categories, tags, subscribers
├── schema intake          # appointment requests, submissions, uploads
├── schema clients         # clients, pets, history, prescriber links
├── schema commerce        # products, variants, carts, orders, payments, subscriptions
├── schema pharmacy        # prescription requests, decisions
├── schema messaging       # conversations, bot rules, menus, schedules
├── schema notifications   # notifications, delivery attempts
└── schema operations      # audit log, preferences
```

Rules:

- A context reads and writes only its own schema
- Cross-schema foreign keys are prohibited; reference by id
- Shared vocabulary (ids, timestamps) uses consistent conventions across schemas
- Migrations are scoped to one context per change wherever possible

---

# Transaction Boundaries

| Boundary | Rule |
|----------|------|
| Prescription decision | Decision row, order state change, and audit entry commit in **one** transaction - never partially |
| Order placement | Order, lines, stock reservation, and payment intent creation are atomic |
| Cart mutation | Single transaction per request |
| Cross-context workflows | **No** distributed transaction; orchestrated by the caller plus compensating events |

Cross-context consistency is eventual: Commerce waits on `PrescriptionApproved` rather than reaching into pharmacy tables. If the event never arrives, the order stays held and an alert is raised - never released silently.

## Consistency Model

| Class | Model | Examples |
|-------|-------|----------|
| Strong | Within a context, one transaction | Stock reservation, prescription release, payment capture |
| Causal | Across contexts via events | Order waiting on authorisation, intake alerting staff |
| Read-your-writes | For authenticated users | Order status immediately after checkout |

---

# Audit and Retention

Append-only audit rows record every clinical and privileged administrative action (ADR-0012): actor, action, target, timestamp, request id. Audit rows are never updated or deleted; retention extends beyond business data retention as defined in `backup-and-recovery.md`.

Soft delete applies only where history matters (client records, articles). Prescription decisions are append-only rather than soft-deleted.

---

# Scaling Strategy

The design targets the NFR requirement: scaling from initial inventory to hundreds of SKUs without page or checkout latency.

| Stage | Trigger | Action |
|-------|---------|--------|
| 1. Now | Single clinic baseline | One PostgreSQL instance, connection pooling, CDN caching |
| 2. Read pressure | Catalog queries measurable against latency budget | Add read-only replica for public catalog and article reads |
| 3. Write contention | Order or stock conflicts measurable in production | Tune indexes and isolation; consider queueing for subscription billing runs |
| 4. Growth beyond a clinic | Multi-location demand appears | Revisit scope entirely (see `../vision.md` non-goals) before restructuring |

Partitioning and sharding are explicitly **not** planned: hundreds of SKUs and a single clinic's order volume do not justify them.

---

# Security Baseline

- TLS for all connections, encrypted storage at rest (NFR: Data Encryption)
- Least-privilege application roles; no superuser in the application
- Separate credentials per environment; secrets live in the VPS `.env` outside the repository, never in code or logs
- Medical-history objects encrypted at rest with access only via signed URLs
- Backups encrypted identically to primary data (see `backup-and-recovery.md`)

---

# Acceptance Criteria

- Every context's schema is documented in `data-model/` as it is built
- No cross-context foreign key exists in any migration
- Prescription and payment transactions are provably atomic under failure
- Scaling stages have measurable triggers, not opinions

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../bounded-context.md` | Context ownership this architecture implements |
| `../domain-model.md` | Entities stored in these schemas |
| `../architectural-decision-record.md` | PostgreSQL and TypeORM decisions |
| `migrations.md` | How this schema changes |
| `performance.md` | How it stays fast |

# Guiding Principle

> **Own your data like you own your records: one place for each fact, constraints that tell the truth, and a transaction you can trust when the answer matters.**
