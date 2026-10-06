# Data Model Handbook

> **ruby-veterinary Documentation**
>
> **Document:** Data Model Handbook
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

This directory holds the per-context data models for ruby-veterinary: the entities each bounded context owns, their attributes and types, relationships, invariants, and indexes. It is the table-level companion to `domain-model.md` (entities) and `database/database-architecture.md` (schemas, transactions, scaling).

Each file maps to exactly one PostgreSQL schema namespace listed in `database/database-architecture.md`. A context reads and writes only its own schema; cross-context references are by opaque prefixed id, never foreign keys.

---

# How To Read These Documents

| Section | Meaning |
|---------|---------|
| Aggregate Roots | The transactional consistency boundaries in that context |
| Entities | Attributes typed for PostgreSQL 16, nullability, and notes |
| Relationships | Directional references; `→ id` means "references by id only" |
| Invariants | Business rules the schema and domain code enforce |
| Indexes | The queries each index exists to serve |
| Cross-Context References | Ids borrowed from other contexts; never duplicated data |

Attribute type conventions:

| Notation | PostgreSQL |
|----------|------------|
| `uuid` | `uuid` with `gen_random_uuid()` default |
| `text` | `text` |
| `citext` | `citext` (emails, where case-insensitive lookup matters) |
| `timestamptz` | `timestamptz` |
| `date` | `date` |
| `numeric(p,s)` | `numeric(p,s)` for money |
| `jsonb` | `jsonb` for flexible payloads that are not queried by column |
| `boolean` | `boolean` |
| `integer` | `integer` |

Identifiers are prefixed opaque strings at the API boundary (`usr_`, `ord_`, `rx_`, `pet_`, `apt_`, `art_`, `svc_`); the database stores `uuid` PKs and the API serialises them. Never expose sequential integers.

---

# File Map

| File | Schema | Context |
|------|--------|---------|
| [identity.md](identity.md) | `identity` | Identity & Access |
| [clinic-content.md](clinic-content.md) | `content` | Clinic Content |
| [publishing.md](publishing.md) | `publishing` | Publishing |
| [care-coordination.md](care-coordination.md) | `intake` | Care Coordination |
| [client-patient-records.md](client-patient-records.md) | `clients` | Client & Patient Records |
| [commerce.md](commerce.md) | `commerce` | Commerce |
| [pharmacy-authorisation.md](pharmacy-authorisation.md) | `pharmacy` | Pharmacy Authorisation |
| [care-messaging.md](care-messaging.md) | `messaging` | Care Messaging |
| [notifications.md](notifications.md) | `notifications` | Notifications |
| [operations.md](operations.md) | `operations` | Operations |

---

# Rules

1. No context table carries another context's columns; reference by id
2. Money uses `numeric`, never floats; currency is clinic base currency on `orders` and `payments`
3. Soft delete only where history matters (client records, articles); prescription decisions are append-only
4. Every business table carries `created_at` and `updated_at timestamptz`
5. Schema changes follow `../database/migrations.md`; these documents update in the same pull request as the migration
6. When an entity lands in code, this file gains a link to the migration PR

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Domains these entities belong to |
| `../bounded-context.md` | Ownership boundaries these models respect |
| `../database/database-architecture.md` | Schema namespaces, transactions, consistency |
| `../database/migrations.md` | How schema changes ship |
| `../api-specification.md` | Serialised shapes and id prefixes |

---

# Acceptance Criteria

- Every schema namespace in `database/database-architecture.md` has a data-model file
- Every entity in `domain-model.md` appears in exactly one data-model file
- No entity document contains a cross-context foreign key
- Attribute tables state type and nullability for every column

---

# Guiding Principle

> **The model is the domain made concrete. If a column cannot be traced to a rule the clinic actually has, it does not belong in the schema.**
