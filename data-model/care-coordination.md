# Care Coordination Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Care Coordination Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Care Coordination Bounded Context
>
> **Classification:** Core Domain
>
> **Schema:** `intake`

---

# Purpose

Defines the tables behind the request surface between owners and the clinic: appointment requests, new-client intake submissions, uploaded medical-history files, and the alerts raised when a form arrives. This is where website traffic becomes scheduled care.

---

# Responsibilities

The Care Coordination context owns:

- Appointment requests (owner details, pet details, reason, preferred windows)
- New-client intake submissions
- History upload references (object keys; bytes live in MinIO)
- Form submission alerts routing into Notifications/Operations views

It does **not** own client or pet records (those are written after acceptance into `clients`), identity accounts (registered users may pre-fill from Identity), or commerce state.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Appointment | `appointment_requests` | Status lifecycle from `received` to triaged/scheduled |
| Intake | `intake_submissions` | Client onboarding payload with linked uploads |

---

# Entities

## appointment_requests

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `apt_…` |
| user_id | uuid | yes | Cross-schema ref by id → `identity.users.id`; null for anonymous requests |
| contact_name | text | no | Always captured, even when signed in |
| contact_phone | text | no | E.164 |
| contact_email | citext | no | Confirmation goes here |
| pet_name | text | no | Free text at request time |
| pet_species | text | yes | `dog`, `cat`, `other` |
| reason | text | no | Free text; staff triages |
| preferred_window_start | timestamptz | yes | |
| preferred_window_end | timestamptz | yes | |
| urgency | text | no | `routine` \| `soon` \| `urgent` — owner-selected hint, not a clinical decision |
| status | text | no | `received` \| `triaged` \| `scheduled` \| `declined` \| `cancelled` |
| idempotency_key | text | yes | Client-supplied; unique per submission window |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `ix_appointment_requests_status_created` on `(status, created_at desc)`
- `uq_appointment_requests_idempotency` on `(idempotency_key)` where not null
- `ix_appointment_requests_user` on `(user_id)`

**Invariants**

- `urgency` never overrides triage; it only prioritises the staff queue view
- Emergency language in `reason` does not auto-book; staff contact path is telephone (per graceful degradation rules)
- Idempotent retries with the same `Idempotency-Key` return the original request

## intake_submissions

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `intake_…` |
| user_id | uuid | yes | Cross-schema ref by id → `identity.users.id` |
| contact_name | text | no | |
| contact_phone | text | no | E.164 |
| contact_email | citext | no | |
| address_line1 | text | yes | Collected for client onboarding |
| address_line2 | text | yes | |
| city | text | yes | |
| postal_code | text | yes | |
| country | text | yes | ISO 3166-1 alpha-2 |
| existing_client | boolean | no | True if owner believes they already have a file |
| preferred_vet_user_id | uuid | yes | Cross-schema ref by id → staff user (optional) |
| notes | text | yes | Owner-supplied history narrative |
| status | text | no | `received` \| `in_review` \| `accepted` \| `rejected` |
| idempotency_key | text | yes | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `ix_intake_submissions_status_created` on `(status, created_at desc)`
- `uq_intake_submissions_idempotency` on `(idempotency_key)` where not null

**Invariants**

- Acceptance (status → `accepted`) may trigger creation of `clients`/`pets` rows in the Clients schema via the owning context's module, never by direct write here
- Rejection records reason for staff dashboard; owner receives plain-language confirmation

## history_uploads

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `upl_…` |
| appointment_request_id | uuid | yes | FK → `appointment_requests.id` (same schema) |
| intake_submission_id | uuid | yes | FK → `intake_submissions.id` |
| object_key | text | no | MinIO key from signed upload (ADR-0009) |
| original_filename | text | yes | Display only; not a path |
| mime_type | text | no | `application/pdf`, `image/jpeg` |
| byte_size | integer | no | ≤ 10 MB enforced at sign time and finalise |
| uploaded_by_user_id | uuid | yes | Cross-schema ref by id |
| status | text | no | `pending` \| `finalised` \| `rejected` |
| rejection_reason | text | yes | Type/size/virus scan failure |
| created_at | timestamptz | no | |

**Indexes**

- `ix_history_uploads_appointment` on `(appointment_request_id)`
- `ix_history_uploads_intake` on `(intake_submission_id)`

**Invariants**

- Exactly one of the two parent ids is set
- Files are private in MinIO; clinic staff reach them through signed URLs issued by this context
- Rejected uploads never finalise into a parent record

## form_submission_alerts

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| source_type | text | no | `appointment_request` \| `intake_submission` |
| source_id | uuid | no | Id of the originating row |
| channel | text | no | `admin_dashboard` \| `email` \| `whatsapp` |
| destination | text | yes | Email address or inbox queue name |
| status | text | no | `queued` \| `delivered` \| `failed` |
| attempts | smallint | no | Default 0 |
| last_error | text | yes | |
| created_at | timestamptz | no | |
| delivered_at | timestamptz | yes | |

**Invariants**

- Alert failures never fail the owner's submission; they raise `AlertRaised` for Operations
- Dashboard delivery is the system of record for staff triage

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → Identity | optional `user_id` for pre-filled and signed-in flows |
| → Client & Patient Records | accepted intake may create clients/pets through that context's module |
| → Notifications | alerts consume this context's events; delivery rows may live here for routing state |
| → Care Messaging | handover may reference an appointment request id in conversation metadata |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Care Coordination domain |
| `../domain-events.md` | `AppointmentRequested`, `IntakeSubmitted` |
| `../api-specification.md` | Intake & Appointments API, upload signing |
| `../non-functional-requirements.md` | Phone fallback, never a dead end |

---

# Acceptance Criteria

- Anonymous appointment requests work with no account and no session
- Idempotent POSTs never create duplicate requests
- Every uploaded file is either finalised against a parent record or explicitly rejected
- Alert failure degrades to dashboard visibility, not lost submissions

---

# Guiding Principle

> **A form that arrives is a responsibility. Capture it completely, confirm it plainly, and make sure someone at the clinic sees it before the owner loses patience.**
