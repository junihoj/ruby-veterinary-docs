# Identity & Access Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Identity & Access Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Identity & Access Bounded Context
>
> **Classification:** Generic Domain
>
> **Schema:** `identity`

---

# Purpose

Defines the tables behind who anyone is and what they may do: client accounts, staff accounts, additive roles, sessions, and staff membership. This is the authentication foundation every other context trusts; it never stores pet, clinical, or order data.

---

# Responsibilities

The Identity & Access context owns:

- Client and staff user accounts
- Credentials and authentication state
- Roles (additive) and role assignment
- Sessions and refresh-token rotation state (ADR-0011)
- Staff membership linking users to the clinic

It does **not** own client profiles, pet records, orders, or any clinical data. Those live in their respective contexts and reference `users.id` by id.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| User | `users` | Roles and staff membership attach to it; sessions are children |
| Role | `roles` | Seeded once; assignments are join rows, not role copies |

---

# Entities

## users

A unique digital identity for a pet owner or a clinic staff member.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised as `usr_…` |
| email | citext | no | Unique; login identifier |
| password_hash | text | no | Argon2id; never logged, never exported |
| display_name | text | yes | Shown in back office and inbox |
| phone | text | yes | E.164; used for WhatsApp correlation when provided |
| kind | text | no | `client` \| `staff` |
| email_verified_at | timestamptz | yes | Null until verified |
| status | text | no | `active` \| `suspended` \| `deleted` |
| last_login_at | timestamptz | yes | Informational |
| created_at | timestamptz | no | Default `now()` |
| updated_at | timestamptz | no | On update trigger or ORM hook |
| deleted_at | timestamptz | yes | Soft delete for account closure |

**Indexes**

- `uq_users_email` unique on `email`
- `ix_users_kind_status` on `(kind, status)`

**Invariants**

- Exactly one row per email; email is never reused after soft delete
- `kind` is immutable after creation
- Staff users require at least one role assignment before any staff API succeeds

## roles

Seeded additive role catalogue.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| code | text | no | Unique: `client`, `receptionist`, `veterinarian`, `vet_technician`, `practice_manager` |
| name | text | no | Display label |
| created_at | timestamptz | no | Seed time |

**Invariants**

- Codes are fixed in migrations; application code never inserts ad-hoc roles
- Roles accumulate (a veterinarian who also publishes is `veterinarian` + content write grant), they do not replace

## user_roles

Many-to-many between users and roles.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| user_id | uuid | no | FK → `users.id` (same schema) |
| role_id | uuid | no | FK → `roles.id` |
| granted_at | timestamptz | no | Audit of when access appeared |
| granted_by | uuid | yes | FK → `users.id` (practice manager) |

**Indexes**

- PK `(user_id, role_id)`

## sessions

Refresh-token rotation state (ADR-0011). Access tokens are stateless JWTs; this table is what makes revocation possible.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| user_id | uuid | no | FK → `users.id` |
| refresh_token_hash | text | no | SHA-256 of the rotating refresh token |
| device_label | text | yes | Best-effort client description |
| ip | inet | yes | Last seen |
| user_agent | text | yes | Last seen |
| expires_at | timestamptz | no | Hard expiry |
| revoked_at | timestamptz | yes | Logout, compromise, or rotation replacement |
| created_at | timestamptz | no | |

**Indexes**

- `ix_sessions_user_active` on `(user_id)` where `revoked_at is null`
- `ix_sessions_expires` on `expires_at`

**Invariants**

- Refresh tokens rotate on every use; the previous row is revoked atomically
- Revoking a session revokes the refresh chain immediately
- Expired sessions are never accepted

## staff_memberships

Links a staff user to the clinic and records employment metadata used for display and inbox attribution.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| user_id | uuid | no | FK → `users.id`; one row per staff user for this single clinic |
| job_title | text | yes | e.g. `Veterinarian` |
| published | boolean | no | Whether the profile appears on the public staff grid (content context reads a projection) |
| starts_on | date | yes | |
| ends_on | date | yes | Null while active |
| created_at | timestamptz | no | |

**Invariants**

- At most one open membership per staff user (single clinic product)
- Public staff display never exposes email, phone, or session data

---

# Cross-Context References

Other schemas reference `users.id` by id only:

| Referencing context | Reference |
|---------------------|-----------|
| Client & Patient Records | `clients.user_id` |
| Publishing | article author → staff user id (via content staff profile) |
| Care Messaging | inbox agent → staff user id |
| Operations | audit actor → user id |

No other schema copies email, password hash, or role rows.

---

# Security Notes

- Passwords hashed with Argon2id; no password ever in a log line
- Refresh token hashes only; raw tokens exist solely in the client
- Role checks live in the API (nest access guards); the database does not encode authorisation
- Account deletion is soft at first, with hard purge per privacy retention schedule (`../database/backup-and-recovery.md`)

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Identity domain summary |
| `../api-specification.md` | Auth API endpoints |
| `../architectural-decision-record.md` | ADR-0011 token session model |
| `../database/database-architecture.md` | `identity` schema placement |

---

# Acceptance Criteria

- Register/login/refresh/logout flows operate entirely within this schema
- Role codes in production match the seeded migration exactly
- Session revocation is observable in `sessions.revoked_at` within one request cycle
- No other schema contains `password_hash` or refresh-token state

---

# Guiding Principle

> **Identity is boring on purpose. Everything interesting in this product happens after this schema says the actor is allowed to be there.**
