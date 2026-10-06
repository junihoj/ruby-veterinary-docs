# Operations Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Operations Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Operations Bounded Context
>
> **Classification:** Generic Domain
>
> **Schema:** `operations`

---

# Purpose

Defines the tables behind the staff-facing control surface: admin dashboard preferences and the global audit log. Operations reads every domain through public interfaces; this schema owns almost nothing of the business itself — which is the point.

---

# Responsibilities

The Operations context owns:

- Dashboard view preferences per staff user
- The platform-wide audit log (privileged actions beyond context-local trails)
- Operational counters/snapshots if later required for dashboard performance

It does **not** own business objects. Orders, prescriptions, articles, and products are never duplicated here. Prescription-specific audit rows live in the Pharmacy schema; this log records cross-cutting administrative actions.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Audit | `audit_log` | Append-only |
| Dashboard | `dashboard_preferences` | Per-user UI state |

---

# Entities

## dashboard_preferences

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| staff_user_id | uuid | no | Cross-schema ref by id → `identity.users.id` |
| preferences_json | jsonb | no | Pinned widgets, inbox density, table columns |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_dashboard_prefs_user` unique on `(staff_user_id)`

## audit_log

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| actor_user_id | uuid | no | Cross-schema ref by id |
| actor_role | text | yes | Role snapshot at action time |
| action | text | no | e.g. `catalog.price_changed`, `article.published`, `role.granted`, `settings.changed` |
| entity_type | text | no | e.g. `product`, `article`, `user` |
| entity_id | uuid | no | Id of affected object (any schema) |
| before_json | jsonb | yes | Redacted snapshot |
| after_json | jsonb | yes | Redacted snapshot |
| request_id | text | yes | Correlation with API logs |
| ip_hash | text | yes | Salted hash; raw IPs not retained here |
| created_at | timestamptz | no | |

**Indexes**

- `ix_audit_log_actor_time` on `(actor_user_id, created_at desc)`
- `ix_audit_log_entity` on `(entity_type, entity_id, created_at desc)`
- `ix_audit_log_action` on `(action, created_at desc)`

**Invariants**

- Insert-only; retention per clinic policy in `../database/backup-and-recovery.md`
- `before_json`/`after_json` never contain passwords, tokens, or card data
- Every write through a privileged admin endpoint emits exactly one row

## dashboard_snapshots (deferred)

Reserved for pre-aggregated daily metrics if dashboard queries ever breach latency budgets. Empty until measured need; creation requires an ADR touch in `dashboard` docs and this file.

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → All contexts | Reads through public modules only |
| → Identity | `actor_user_id`, `staff_user_id` |
| ← Notifications | Alert stream consumed for admin alert dashboard |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Operations domain |
| `../bounded-context.md` | Operations reads everything, owns almost nothing |
| `../api-specification.md` | Admin alerts and inbox APIs |
| `pharmacy-authorisation.md` | Clinical audit lives there, not here |

---

# Acceptance Criteria

- No business table exists in `operations` that another context owns
- Audit rows are produced for all privileged admin writes
- Dashboard preferences are per-user and fail closed to defaults

---

# Guiding Principle

> **Operations is a mirror, not a warehouse. If the same fact is stored twice, one of the copies will lie.**
