# performance.md

> **ruby-veterinary Documentation**
>
> **Document:** Database Performance
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

This document defines how the ruby-veterinary database stays fast: objectives, indexing standards, caching, query budgets, monitoring, and load testing.

It operationalises the NFR commitments in `../non-functional-requirements.md` - most importantly sub-3-second page loads on 4G and a catalog that scales to hundreds of SKUs without checkout delays.

---

# Performance Philosophy

- Measure before optimising; the budget is the arbiter, not intuition
- Index for the queries the product actually runs
- Cache at the outermost layer that can be invalidated safely (CDN first)
- Design for the clinic's scale; document the step beyond it rather than building it early

# Performance Objectives

| Objective | Target | Source |
|-----------|--------|--------|
| Homepage and emergency pages | < 3s on standard 4G mobile | NFR: Page Load Time |
| WhatsApp bot response | < 2s including DB work | NFR: Bot Response Latency |
| Catalog listing API | p95 < 300 ms server-side | Budget supporting the 3s page goal |
| Checkout quote | p95 < 500 ms | Budget supporting no checkout delays |
| Uptime | 99.9% | NFR: Website Uptime |
| Catalog scale target | Hundreds of SKUs with no latency regression | NFR: Catalog Scalability |

Server-side budgets are sized so the network and rendering remainder still fit the 3s page target.

# Indexing Standards

## Principles

- Every foreign key used for joins or filters is indexed
- Indexes serve measured queries; speculative indexes are reviewed out
- Composite index column order follows equality, then range, then sort
- Selective predicates first where cardinality differs sharply

## Required Patterns (per context schema)

| Table (example) | Index | Driven By |
|-----------------|-------|-----------|
| `commerce.products` | `(category_id, is_active)`, GIN on filter facets where used | Catalog listing filters |
| `commerce.product_variants` | Unique `(sku)`, `(product_id)` | Checkout and stock checks |
| `commerce.orders` | `(client_id, created_at desc)`, `(status)` | Order history, admin queue |
| `pharmacy.prescription_requests` | Partial index on `(status) WHERE status = 'pending'` | Review queue |
| `publishing.articles` | Unique `(slug)`, `(published_at desc)`, `(category_id)` | Public article queries |
| `messaging.conversations` | `(status, updated_at desc)` | Shared inbox ordering |
| `intake.appointment_requests` | `(created_at desc)`, `(status)` | Admin alerts |
| `clients.pets` | `(client_id)`, `(primary_veterinarian_id)` | Patient association at checkout |

## Conventions

- Named: `idx_<context>_<table>_<columns>`, unique: `uq_...`
- Created `CONCURRENTLY` in production (see `migrations.md`)
- Partial indexes for status queues where one state dominates
- Never index everything "just in case" - each index costs writes on orders and conversations

# Query Patterns to Avoid

- `SELECT *` over wide media rows on list endpoints
- N+1 queries from repository loops (fetch in batches or join)
- Unbounded `WHERE ... IN` with client-supplied id lists
- Cross-schema joins (prohibited by `database-architecture.md`)
- String-built filters without parameterisation

# Caching Strategy

| Layer | What | Invalidation |
|-------|------|--------------|
| Cloudflare CDN edge (in front of the VPS) | Public pages, images, public read APIs with short TTL | Purge on publish; short TTL as safety net |
| Application in-process | Hot reference data (hours, categories) | TTL seconds; safe because data is rarely written |
| HTTP revalidation | Articles and service pages | On `ArticlePublished` / service edit events |
| Never cached | Cart, checkout quote, order status, prescription state, shared inbox | Always live |

Caching must never mask prescription or payment state: stale reads are unacceptable where clinical or financial correctness is involved.

# Connection Management

- Connection pooling between the application and PostgreSQL; pool size sized to the VPS, not maxed
- Pool exhaustion alerts at threshold
- Long-running analytical queries (if ever added) go to a replica, not the primary

# Monitoring

| Signal | Why |
|--------|-----|
| Query latency p50/p95/p99 by statement | Finds regressions before users do |
| Slow query log with threshold (for example > 200 ms) | Tuning queue |
| Lock waits and deadlocks | Contention on orders and conversations |
| Cache hit ratio | Index and memory health |
| Connection saturation | Capacity warning |
| Replication lag (if a replica is added) | Read-your-writes safety |

Monitoring feeds the admin alert dashboard concept from `../functional-requirements.md` (Notifications & Management) for operational signals.

# Load Testing

- Baseline before each commerce phase (P4, P5 in `../prd.md`)
- Scenarios: public article browsing, catalog filter + detail, checkout burst, prescription queue concurrent decisions, WhatsApp inbound spike
- Verified that catalog filters stay within budget at 10x expected SKU count
- Results recorded; regressions block release of the phase

# Performance Reviews

- Review index usage (`pg_stat_user_indexes`) quarterly; drop unused indexes
- Revisit budgets whenever a new page template ships
- Analyse slow query trends after every traffic milestone

---

# Anti-Patterns

- Optimising without a failing measurement
- Adding a cache in front of a wrong query
- Indexing a table no one reads
- Treating the 3s budget as a frontend-only concern

---

# Acceptance Criteria

- Every endpoint in `api-specification.md` has a stated budget and monitoring
- Required indexes from this document exist and are used
- Load test passes at 10x catalog size before commerce phases ship
- Slow query log is empty of unresolved entries at release time

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../non-functional-requirements.md` | Performance commitments these budgets serve |
| `database-architecture.md` | Schema and ownership the indexes respect |
| `migrations.md` | Safe index creation |
| `../api-specification.md` | Endpoints carrying these budgets |

# Guiding Principle

> **Speed is a feature users feel before they read a word. Budget it, measure it, and let the p95 settle every argument.**
