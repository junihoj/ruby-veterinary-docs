# Client & Patient Records Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Client & Patient Records Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Client & Patient Records Bounded Context
>
> **Classification:** Supporting Domain
>
> **Schema:** `clients`

---

# Purpose

Defines the tables behind the relationship data that makes care personal: client records linked to identity accounts, pets and their basic profile, medical-history references, and each pet's primary veterinarian on file. Pharmacy authorisation depends on this schema for the patient-and-prescriber association.

---

# Responsibilities

The Client & Patient Records context owns:

- Client records (owner accounts bound to a user id or intake identity)
- Pets and their basic profile
- Medical history record references (object keys; clinical review stays with staff)
- Primary veterinarian links per pet

It does **not** own identity credentials, appointment requests, orders, or prescription decisions. Pet data is referenced by id from Pharmacy and Commerce; it is never copied into those schemas.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Client | `clients` | Pets and history belong to the client relationship |
| Pet | `pets` | Profile + primary-vet link move together |

---

# Entities

## clients

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `cli_…` |
| user_id | uuid | yes | Cross-schema ref by id → `identity.users.id`; null for phone-only clients |
| display_name | text | no | Owner name as recorded |
| email | citext | yes | |
| phone | text | no | E.164 |
| address_line1 | text | yes | |
| address_line2 | text | yes | |
| city | text | yes | |
| postal_code | text | yes | |
| country | text | yes | |
| preferred_contact | text | no | `phone` \| `email` \| `whatsapp` |
| source | text | no | `intake` \| `walk_in` \| `existing_import` |
| status | text | no | `active` \| `inactive` \| `deleted` |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |
| deleted_at | timestamptz | yes | Soft delete; privacy purge path documented in backup handbook |

**Indexes**

- `uq_clients_user` unique on `(user_id)` where not null
- `ix_clients_phone` on `(phone)`
- `ix_clients_status` on `(status)`

**Invariants**

- At most one active client per identity user (single-clinic product)
- Deleting a client never hard-deletes pets while clinical history must be retained per policy; retention schedule in `../database/backup-and-recovery.md` governs purge

## pets

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `pet_…` |
| client_id | uuid | no | FK → `clients.id` |
| name | text | no | |
| species | text | no | `dog`, `cat`, `bird`, `rabbit`, `other` |
| breed | text | yes | |
| sex | text | yes | `female`, `male`, `unknown` |
| neutered | boolean | yes | |
| date_of_birth | date | yes | Best effort |
| weight_kg | numeric(6,2) | yes | Updated at visits |
| microchip_id | text | yes | |
| photo_object_key | text | yes | MinIO key |
| notes | text | yes | Owner-visible notes field only; clinical detail stays in history records |
| status | text | no | `active` \| `deceased` \| `deleted` |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `ix_pets_client` on `(client_id, status)`
- `ix_pets_name` on `(name)`

**Invariants**

- A pet always belongs to exactly one client
- Deceased pets remain queryable for history; they cannot be added to new carts' prescription claims without staff override

## medical_history_records

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `mhr_…` |
| pet_id | uuid | no | FK → `pets.id` |
| title | text | no | e.g. `2025 vaccination card` |
| object_key | text | no | MinIO key (owner upload or staff upload) |
| mime_type | text | no | `application/pdf`, `image/jpeg` |
| byte_size | integer | no | |
| source | text | no | `owner_upload` \| `staff_upload` \| `import` |
| uploaded_by_user_id | uuid | yes | Cross-schema ref by id |
| visit_date | date | yes | |
| summary | text | yes | Short clinical summary visible to authorised staff |
| created_at | timestamptz | no | |

**Indexes**

- `ix_medical_history_records_pet` on `(pet_id, created_at desc)`

**Invariants**

- Records are reference objects; bytes never live in PostgreSQL
- Owner-visible metadata never includes free-text clinical notes not intended for the owner

## primary_veterinarian_links

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| pet_id | uuid | no | FK → `pets.id` |
| veterinarian_user_id | uuid | no | Cross-schema ref by id → staff user with veterinarian role |
| effective_from | date | no | |
| effective_to | date | yes | Null = current |
| created_at | timestamptz | no | |

**Indexes**

- `ix_primary_vet_current` unique on `(pet_id)` where `effective_to is null`

**Invariants**

- Exactly one current primary vet per pet (Pharmacy guardrail depends on this)
- Changing the link closes the previous row; history is preserved

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → Identity | `clients.user_id`; credentials never stored here |
| → Pharmacy | `pets.id`, `primary_veterinarian_links` for Rx authorisation |
| → Commerce | `pets.id` for prescription-item claims on checkout |
| → Care Coordination | accepted intake may create clients/pets via this context's module |
| ← Publishing | reads nothing here |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Client & Patient Records domain |
| `../data-model/pharmacy-authorisation.md` | How Rx uses patient + primary vet |
| `../api-specification.md` | Client-facing status reads |
| `../database/backup-and-recovery.md` | Retention and privacy purge |

---

# Acceptance Criteria

- Every pet has a client; no orphan pets exist in production
- A pet with an active prescription request always has a current primary-vet link
- Soft-deleted clients disappear from active queries but retain audit/history pointers
- No other schema stores pet demographics in duplicate

---

# Guiding Principle

> **The patient record is the clinic's memory of a living animal. Keep it accurate, keep ownership obvious, and never scatter copies of it across the system.**
