# migrations.md

> **ruby-veterinary Documentation**
>
> **Document:** Database Migrations
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

This document defines how the schema of the ruby-veterinary database changes: naming, lifecycle, zero-downtime rules, rollback strategy, testing, and CI integration.

All schema change is schema-as-code. There is no manual production DDL, ever.

---

# Guiding Principles

## Schema as Code

Migrations are version-controlled TypeScript files managed by TypeORM's migration tooling. They are reviewed in pull requests like any other code.

## Forward-Only Mindset

Migrations move forward. A merged migration is immutable; correcting a mistake means writing a new migration that supersedes it, not editing history. Down migrations are generated where the tooling allows but are **never** the default recovery path - recovery is a forward fix or a restore.

## Zero-Downtime First

Every change must be safe to apply while the previous application version is still serving traffic (deploy and migrate are decoupled).

## Small, Incremental Changes

One logical change per migration, scoped to one context schema where possible.

---

# Migration Lifecycle

1. Branch from `main`
2. Generate or write the migration against the local database
3. Apply locally and run the test suite against the migrated schema
4. Pull request: migration file + the code that depends on it in the same change
5. CI applies migrations from scratch onto an empty database, then runs the full suite
6. Merge; deploy applies migrations as a release step before the new code serves traffic
7. Post-deploy verification confirms schema version matches expectation

# Naming Convention

```
<TypeOrTimestamp>-<Verb><Thing>
```

Examples: `0001-CreateClientTable`, `0007-AddPrescriptionAuditActor`. Sequential, sortable, never reused.

# Migration Types

| Type | Example | Rules |
|------|---------|-------|
| **Schema** | Create table, add column, add index | Must be zero-downtime (see matrix below) |
| **Data** | Backfill `status` on existing rows | Batched; never a single transaction over unbounded rows |
| **Seed** | Reference rows (categories, hours) | Idempotent; never seeded in production by migration where code can do it |

---

# Zero-Downtime Strategy

| Change | Safe Pattern | Unsafe (Requires Maintenance Window or Expansion) |
|--------|--------------|---------------------------------------------------|
| Add column | Add nullable column (or with default in PG11+), deploy code reading/writing it, backfill, then add `NOT NULL` | Adding `NOT NULL` to a populated column in one step |
| Add table | Create table, deploy code | - |
| Rename table/column | Create new, deploy dual-read/write, backfill, switch, drop old later | Rename in place while old code is live |
| Drop column | Deploy code that stops using it, then drop in a later release | Dropping while code still reads it |
| Add index | `CREATE INDEX CONCURRENTLY` (outside a transaction) | Blocking `CREATE INDEX` on a hot table |
| Constraint | `NOT VALID` then `VALIDATE` (two steps) | Validating over a large table in one statement |

The expand-contract pattern is the default for every breaking shape change: expand (additive), migrate traffic, contract (remove old) in a separate release.

# Rollback Strategy

| Scenario | Response |
|----------|----------|
| Migration fails mid-apply | Fix forward with a new migration; investigate before retrying |
| Bad data written by new code | Application rollback plus compensating migration |
| Catastrophic loss | Restore from backup per `backup-and-recovery.md` |

Destructive operations (drops, narrowing types) are only ever performed after the replacement path has been live and verified.

# Transactions and Locking

- Regular migrations run in a transaction; `CONCURRENTLY` index builds do not and must be run by the migration runner's no-transaction mode
- Avoid long-held `ACCESS EXCLUSIVE` locks on hot tables (orders, conversations)
- Check `pg_locks` and migration duration budgets in review for production-sized tables

# Data Backfills

- Batched (for example, 1,000 rows per commit) with resumability
- Logged with progress; failure mid-backfill resumes, never restarts blindly
- Never combined with the deploy that introduces the reader of the new data

# Migration Testing

| Test | When |
|------|------|
| Fresh-database apply | Every CI run |
| Migrate-from-previous-release | Every CI run (applies the previous release schema, then the new migrations) |
| Suite against migrated schema | Every CI run |
| Lock duration check on representative data | Before merging migrations touching hot tables |

# CI/CD Integration

Migrations are applied by the deploy pipeline as a distinct, observable step before the new application version becomes healthy (see `../deployment-architecture.md`). Failures halt the rollout; the previous containers keep serving.

# Schema Drift

Drift detection compares the live schema against the migration history on a schedule. Detected drift raises an alert and blocks deploys until reconciled - drift means someone changed production by hand.

---

# Anti-Patterns

- Editing a merged migration
- Manual `ALTER` on production
- Mixing two contexts' changes in one migration
- Backfills inside the migration that adds the reading code
- Dropping anything in the same release that stops using it

---

# Acceptance Criteria

- CI proves fresh apply and previous-release upgrade paths before merge
- Every breaking change follows expand-contract with a stated contract phase
- No production schema change exists that is not represented in migration history
- Lock durations on hot tables are measured, not assumed

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `database-architecture.md` | The schema this process maintains |
| `../engineering-guidelines.md` | Review and CI standards |
| `backup-and-recovery.md` | What to do when a migration destroys data |
| `../architectural-decision-record.md` | TypeORM decision (ADR-0004) |

# Guiding Principle

> **A migration is a promise that tomorrow's schema can be built from yesterday's. Write it once, prove it upgrades, and never touch it again.**
