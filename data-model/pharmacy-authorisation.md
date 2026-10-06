# Pharmacy Authorisation Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Pharmacy Authorisation Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Pharmacy Authorisation Bounded Context
>
> **Classification:** Core Domain
>
> **Schema:** `pharmacy`

---

# Purpose

Defines the tables behind the clinical guardrail inside checkout: prescription authorisation requests, the mandatory patient-and-veterinarian association, veterinarian decisions (approve, reject, query), and the append-only audit trail (ADR-0012). This is the smallest schema in the product and carries the highest risk.

---

# Responsibilities

The Pharmacy Authorisation context owns:

- Prescription requests raised by checkout or staff
- Patient association (pet + primary veterinarian context at decision time)
- Decision rows: approve / reject / query with reasons
- Append-only decision audit trail

It does **not** write commerce order state; decisions flow back as events (`PrescriptionApproved`, `PrescriptionRejected`, `PrescriptionQueried`) that Commerce consumes. It reads pet/vet references by id from the Clients schema.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Prescription | `prescription_requests` | Decision history is append-only and audited |

---

# Entities

## prescription_requests

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `rx_…` |
| order_id | uuid | no | Cross-schema ref by id → `commerce.orders.id` |
| order_item_ids | uuid[] | no | Rx lines this request covers |
| user_id | uuid | no | Cross-schema ref by id → owner |
| pet_id | uuid | no | Cross-schema ref by id → `clients.pets.id` |
| veterinarian_user_id | uuid | no | Cross-schema ref by id → primary vet (denormalised at request time for queue display; link row remains authoritative) |
| status | text | no | `pending` \| `approved` \| `rejected` \| `queried` \| `cancelled` |
| owner_note | text | yes | Free text from checkout |
| requested_at | timestamptz | no | |
| decided_at | timestamptz | yes | |
| decided_by | uuid | yes | Cross-schema ref by id → staff user who decided |
| created_at | timestamptz | no | |

**Indexes**

- `ix_rx_requests_queue` on `(status, requested_at)` — staff review queue
- `ix_rx_requests_order` on `(order_id)`
- `ix_rx_requests_pet` on `(pet_id)`

**Invariants**

- Every Rx order line maps to exactly one open or decided request (idempotent per order + pet + line set at placement)
- `veterinarian_user_id` must reference a user with the veterinarian role at decision time
- Status transitions: `pending → approved | rejected | queried`; `queried → approved | rejected`; terminal states never reopen (corrections are new decisions + audit events)

## prescription_decisions

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `rxd_…` |
| prescription_request_id | uuid | no | FK → `prescription_requests.id` |
| decision | text | no | `approve` \| `reject` \| `query` |
| reason_code | text | yes | e.g. `wrong_dose`, `need_history`, `not_under_care` |
| reason_text | text | yes | Required for reject/query |
| decided_by | uuid | no | Cross-schema ref by id → veterinarian staff user |
| decided_at | timestamptz | no | |
| created_at | timestamptz | no | |

**Indexes**

- `ix_rx_decisions_request` on `(prescription_request_id, decided_at)`

**Invariants**

- Rows are insert-only; updates and deletes are prohibited (append-only, ADR-0012)
- Multiple decisions may exist for a `queried` request; the latest non-query decision wins for status projection

## rx_audit_log

Append-only audit mirroring every decision and privileged read touchpoint for prescription data.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| prescription_request_id | uuid | no | FK → `prescription_requests.id` |
| actor_user_id | uuid | no | Cross-schema ref by id |
| action | text | no | `viewed`, `decision_recorded`, `queue_claimed`, `exported` |
| detail_json | jsonb | yes | Non-sensitive context (request id, ip hash) |
| request_id | text | yes | API requestId for correlation |
| created_at | timestamptz | no | |

**Invariants**

- Insert-only; retained beyond business data per clinic policy
- Export/view actions are logged where staff access prescription content

## patient_associations

Snapshot rows recording which pet/vet context a decision relied on, for reproducibility when links change later.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| prescription_request_id | uuid | no | FK → `prescription_requests.id` |
| pet_id | uuid | no | Cross-schema ref by id |
| veterinarian_user_id | uuid | no | Cross-schema ref by id at decision time |
| client_id | uuid | yes | Cross-schema ref by id |
| snapshot_at | timestamptz | no | |
| created_at | timestamptz | no | |

**Invariants**

- One association snapshot per decided request; pending requests snapshot on first queue view

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → Commerce | `order_id` / `order_item_ids`; decisions publish events, never update orders here |
| → Client & Patient | `pet_id`, `veterinarian_user_id` |
| → Identity | veterinarian role verification at decision time |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Pharmacy Authorisation domain |
| `../domain-events.md` | `PrescriptionRequested/Approved/Rejected` |
| `../api-specification.md` | Pharmacy Authorisation API |
| `client-patient-records.md` | Primary-vet link the guardrail depends on |
| `../architectural-decision-record.md` | ADR-0012 append-only clinical decisions |

---

# Acceptance Criteria

- No approved prescription exists without a veterinarian actor recorded
- Decision tables reject UPDATE/DELETE at the database level (trigger or revoke)
- Audit rows exist for every decision and privileged view
- Commerce moves Rx orders forward only via events from this schema's transitions

---

# Guiding Principle

> **This schema is the clinic's licence to sell medicine online. Make every decision attributable, append-only, and impossible to fake after the fact.**
