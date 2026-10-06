# Clinic Content Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Clinic Content Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Clinic Content Bounded Context
>
> **Classification:** Supporting Domain
>
> **Schema:** `content`

---

# Purpose

Defines the tables behind the public clinic profile: services and baseline pricing, staff profiles for the public grid, hours and holiday hours, location/contact data, static information pages, and downloadable care documents. This schema carries the three-second emergency clarity requirement at the data level (hours and contact are always current).

---

# Responsibilities

The Clinic Content context owns:

- Service catalog with baseline pricing and slugs
- Staff profiles for public display (name, credentials, accreditations, published flag)
- Physical location and contact numbers (including emergency numbers)
- Standard and holiday business hours
- Static information pages
- Downloadable care documents (post-op sheets, waivers, travel certificates)

It does **not** own articles (Publishing), appointment requests (Care Coordination), or staff credentials for authentication (Identity). Publishing reads staff profiles read-only by id.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Service | `services` | Slug-driven public pages; baseline pricing is display-only |
| StaffProfile | `staff_profiles` | Published projection for the public grid |
| ClinicProfile | `clinics` | One row: location, phones, emergency numbers |

---

# Entities

## clinics

One row for the single clinic this product serves.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | Display name |
| slug | text | no | Unique |
| address_line1 | text | no | |
| address_line2 | text | yes | |
| city | text | no | |
| region | text | yes | State/province |
| postal_code | text | yes | |
| country | text | no | ISO 3166-1 alpha-2 |
| latitude | numeric(9,6) | yes | For map embeds |
| longitude | numeric(9,6) | yes | |
| main_phone | text | no | E.164; tap-to-call source |
| emergency_phone | text | no | E.164; highest-priority display everywhere |
| whatsapp_display | text | yes | Display-only if chat CTA exists |
| timezone | text | no | IANA, e.g. `Africa/Lagos`; governs hours logic |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Invariants**

- Exactly one active `clinics` row
- `emergency_phone` is never null and never hidden behind client-side filtering

## services

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `svc_…` |
| clinic_id | uuid | no | FK → `clinics.id` (same schema) |
| name | text | no | |
| slug | text | no | Unique |
| summary | text | no | One-line description for cards and SEO |
| description_md | text | yes | Markdown body for the service page |
| baseline_price | numeric(10,2) | yes | Display-only "from" price; not a quote |
| currency | char(3) | yes | ISO 4217; defaults to clinic base |
| category | text | no | e.g. `consultation`, `surgery`, `dentistry`, `vaccination`, `grooming` |
| featured | boolean | no | Highlight on home/services index |
| requires_appointment | boolean | no | |
| published | boolean | no | Unpublished rows are invisible to public API |
| sort_order | integer | no | Default 0 |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_services_slug` unique on `(slug)`
- `ix_services_published_category` on `(published, category, sort_order)`

**Invariants**

- `baseline_price` is indicative only; checkout never uses this table for money
- Unpublished services never appear in public reads or sitemap

## staff_profiles

Public projection of staff members. Authentication identity lives in `identity.users`; this table owns presentation fields only.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `stf_…` |
| user_id | uuid | no | Cross-schema reference by id → `identity.users.id` (no FK across schemas) |
| display_name | text | no | |
| title | text | no | e.g. `Dr.`, `Veterinary Nurse` |
| credentials | text[] | no | Qualifications list |
| accreditations | text[] | yes | Registry memberships |
| bio_md | text | yes | Short professional bio |
| photo_object_key | text | yes | MinIO key; served via signed/CDN URL |
| specialisms | text[] | yes | Display tags |
| published | boolean | no | |
| sort_order | integer | no | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_staff_profiles_user` unique on `(user_id)`
- `ix_staff_profiles_published` on `(published, sort_order)`

**Invariants**

- One profile per staff user
- Publishing never exposes email, phone, or internal role rows

## business_hours

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| clinic_id | uuid | no | FK → `clinics.id` |
| weekday | smallint | no | 0=Sunday … 6=Saturday |
| opens_at | time | no | Clinic-local time |
| closes_at | time | no | Clinic-local time |
| closed | boolean | no | True overrides open/close for that day |
| created_at | timestamptz | no | |

**Indexes**

- `uq_business_hours_day` unique on `(clinic_id, weekday)`

## holiday_hours

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| clinic_id | uuid | no | FK → `clinics.id` |
| holiday_date | date | no | |
| opens_at | time | yes | Null when fully closed |
| closes_at | time | yes | Null when fully closed |
| closed | boolean | no | |
| label | text | yes | e.g. `Public Holiday` |
| created_at | timestamptz | no | |

**Indexes**

- `uq_holiday_hours_date` unique on `(clinic_id, holiday_date)`

**Invariants**

- Open-state API derives: holiday row wins over `business_hours` for that date
- Hours are the clinic's own definition; no third-party hours feed

## information_pages

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| slug | text | no | Unique, e.g. `about`, `privacy`, `cookie-policy` |
| title | text | no | |
| body_md | text | no | |
| seo_title | text | yes | |
| seo_description | text | yes | |
| published | boolean | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_information_pages_slug` unique on `(slug)`

## downloadable_documents

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `doc_…` |
| title | text | no | e.g. `Post-operative care sheet` |
| slug | text | no | Unique |
| description | text | yes | |
| object_key | text | no | MinIO key (ADR-0009) |
| mime_type | text | no | `application/pdf` |
| byte_size | integer | no | |
| published | boolean | no | |
| created_at | timestamptz | no | |

**Indexes**

- `uq_downloadable_documents_slug` unique on `(slug)`

**Invariants**

- Public download issues a short-lived signed URL; the bucket stays private
- Document removal is soft (`published=false` first) so historical links fail gracefully with a phone fallback

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| Publishing → this schema | `staff_profiles.user_id` read-only for author bylines |
| Care Coordination | Reads `business_hours`/`holiday_hours` to label "open now" on intake forms |
| Commerce | Reads nothing here; pricing lives on products |
| Operations | Writes via this context's public module only |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Clinic Content domain |
| `../api-specification.md` | Clinic Content API endpoints |
| `../prd.md` | 3-second emergency clarity requirement |
| `publishing.md` | Staff byline source |

---

# Acceptance Criteria

- Public clinic page renders hours, both phones, and address from this schema alone
- Service slugs are unique and stable; unpublished rows are invisible
- Holiday overrides beat standard hours in every "open now" computation
- Downloads never expose raw bucket paths without a signed URL

---

# Guiding Principle

> **The clinic profile is the product's handshake. Keep it accurate, keep it fast, and keep the emergency number one tap away from every page.**
