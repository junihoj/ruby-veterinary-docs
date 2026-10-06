# Database Engineering Handbook

> **ruby-veterinary Documentation**
>
> **Document:** Database Handbook
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Architecture Standard

---

# Overview

This handbook covers everything about how ruby-veterinary persists data: architecture, schema change, performance, and recovery.

# Purpose

To give one authoritative place for persistence decisions so that migrations, indexes, and backups are handled the same way every time, by anyone.

# Database Philosophy

- **Business first** - schema follows the domain model, not the ORM's defaults
- **Data integrity above convenience** - foreign keys, constraints, and transactions guard clinical and financial correctness
- **Scalability through evolution** - design for the catalog and traffic the clinic has, with documented steps for hundreds of SKUs and beyond
- **Boring by default** - one database, one engine, no polyglot persistence until a measured need exists

# Technology Stack

| Concern | Choice | Reference |
|---------|--------|-----------|
| Primary database | PostgreSQL 16+ | ADR-0003 |
| Access layer | TypeORM behind repository interfaces | ADR-0004 |
| Migrations | TypeORM migrations, version-controlled | `migrations.md` |
| Large files | Object storage, not the database | ADR-0009 |
| Caching | CDN edge for public reads; in-process for hot queries | `performance.md` |

# Architecture Overview

One PostgreSQL database with per-bounded-context schema namespaces. Contexts own their schemas exclusively; cross-context foreign keys are prohibited (see `../bounded-context.md`). Medical-history and CMS media files live in object storage with only their references in the database.

# Handbook Structure

| Document | Contents |
|----------|----------|
| `database-architecture.md` | Ownership, consistency, transactions, scaling path |
| `migrations.md` | Schema change lifecycle and zero-downtime rules |
| `performance.md` | Indexing, caching, query budgets, monitoring |
| `backup-and-recovery.md` | Backup scope, schedules, restore procedures, DR drills |

# Related Documentation

- `../domain-model.md` - entities and ownership
- `../architectural-decision-record.md` - database decisions
- `../engineering-guidelines.md` - day-to-day standards

# Intended Audience

Backend engineers, reviewers, and whoever is paged when the site is slow.

# Change Management

Schema changes flow through `migrations.md`: branch, review with the owning context, merge, apply. No manual production DDL ever.

# Future Evolution

Background job processing (Redis plus a queue) and read replicas are explicitly deferred until measured need exists. When introduced, they require an ADR and an update to this handbook.

---

# Acceptance Criteria

- Every persistence decision on this page maps to an accepted ADR
- Every schema change enters through the documented migration path
- Backups are automated, verified, and restore-tested

---

# Guiding Principle

> **The database is the clinic's memory. Change it deliberately, index it for the queries that matter, and prove every day that it can be recovered.**
