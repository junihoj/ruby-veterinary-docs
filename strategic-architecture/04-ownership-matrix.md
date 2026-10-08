# Ownership Matrix

> **ruby-veterinary Strategic Architecture**
>
> **Document:** 04 — Ownership Matrix
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary

---

# Purpose

Specifies exactly which context owns each data domain, event, API surface, database schema, and operational surface. Every piece of business data has exactly one authoritative owner. Table-level detail lives in `../data-model/`.

---

# Data Ownership

| Context | Owns Exclusively | Never Writes |
|---------|------------------|--------------|
| Identity & Access | users, roles, user_roles, sessions, staff_memberships | client profiles, pet data |
| Clinic Content | clinics, services, staff_profiles, hours, pages, documents | articles, orders |
| Publishing | articles, categories, tags, media_assets, newsletter_subscribers | staff profiles (reads them) |
| Care Coordination | appointment_requests, intake_submissions, history_uploads, form_submission_alerts | patient records, orders |
| Client & Patient Records | clients, pets, medical_history_records, primary_veterinarian_links | identity credentials, commerce state |
| Commerce | products, variants, composite slots, grouped members, download files, carts, orders, payments, subscriptions | prescription decisions, pet records |
| Pharmacy Authorisation | prescription_requests, prescription_decisions, rx_audit_log | order state (emits events) |
| Care Messaging | conversations, inbox_messages, bot_rules, menu_flows, handover_sessions | appointment records (emits events) |
| Notifications | notifications, delivery_channels, delivery_attempts, alert_subscriptions | business objects it reports on |
| Operations | dashboard_preferences, audit_log | everything else (reads through interfaces) |

---

# Event Ownership

| Event | Publisher | Primary Consumers |
|-------|-----------|-------------------|
| AppointmentRequested | Care Coordination | Notifications, Operations |
| IntakeSubmitted | Care Coordination | Notifications, Client & Patient Records (on accept) |
| ArticlePublished | Publishing | Notifications (newsletter), CDN invalidation |
| OrderPlaced | Commerce | Notifications, Pharmacy (if Rx lines) |
| PaymentCaptured / PaymentFailed | Commerce (via payments webhook) | Notifications, Operations |
| OrderFulfilled | Commerce | Notifications |
| PrescriptionRequested | Commerce / Pharmacy intake | Pharmacy (queue), Notifications |
| PrescriptionApproved / Rejected / Queried | Pharmacy Authorisation | Commerce, Notifications |
| EmergencyDetected | Care Messaging | Notifications (critical), Operations |
| HandoverRequested / HandoverAccepted | Care Messaging | Notifications, Care Coordination |
| AlertRaised | Notifications | Operations dashboard |

---

# API Surface Ownership

| Route Group (`../api-specification.md`) | Owner Context |
|------------------------------------------|---------------|
| `/auth/*` | Identity & Access |
| `/clinic/*`, `/services/*`, `/staff/*`, `/documents/*` | Clinic Content |
| `/appointment-requests`, `/clients/intake`, `/uploads/sign` | Care Coordination |
| `/articles*`, `/categories`, `/tags`, `/newsletter/*` | Publishing |
| `/products*`, `/cart*`, `/checkout*`, `/orders*`, `/subscriptions*`, `/admin/products*` | Commerce |
| `/prescriptions/*` | Pharmacy Authorisation |
| `/webhooks/whatsapp*` | Care Messaging |
| `/webhooks/payments` | Commerce |
| `/admin/alerts`, `/admin/inbox/*` | Operations (reads Care Messaging/Notifications) |

---

# Schema Ownership

| PostgreSQL Schema | Owner |
|-------------------|-------|
| `identity` | Identity & Access |
| `content` | Clinic Content |
| `publishing` | Publishing |
| `intake` | Care Coordination |
| `clients` | Client & Patient Records |
| `commerce` | Commerce |
| `pharmacy` | Pharmacy Authorisation |
| `messaging` | Care Messaging |
| `notifications` | Notifications |
| `operations` | Operations |

No context migrates another context's schema. Cross-schema foreign keys are prohibited (`../database/database-architecture.md`).

---

# Operational Surfaces

| Surface | Owner Context |
|---------|---------------|
| Admin alert dashboard view | Operations (reads Notifications) |
| Shared inbox view | Operations (reads Care Messaging) |
| Prescription review queue UI | Pharmacy Authorisation via Operations presentation |
| Catalog/CMS editors | Commerce / Publishing via Operations presentation |
| CDN + Cloudflare config | Platform (deployment-architecture.md), not a context |

---

# Acceptance Criteria

- Every business table in `../data-model/` appears under exactly one owner
- Every event in `../domain-events.md` has exactly one publisher
- No context's "writes" column contains another context's objects

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../data-model/` | Table-level ownership detail |
| `../bounded-context.md` | Ownership summary (source) |
| `06-architecture-rules.md` | Rules enforcing single ownership |
| `../database/database-architecture.md` | Schema namespaces |

---

# Guiding Principle

> **Ownership is the unit of accountability. If two contexts can write the same row, nobody is responsible for it.**
