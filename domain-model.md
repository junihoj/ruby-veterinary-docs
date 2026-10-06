# domain-model.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Domain Model
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

This document defines the business domains of the ruby-veterinary platform, the entities each domain owns, and how the domains depend on one another.

It is the bridge between the feature checklists in `functional-requirements.md` and the technical decomposition in `bounded-context.md`. It deliberately stops short of table and column design, which belongs in `data-model/`.

---

# Core Domains

- Clinic Content
- Publishing
- Care Coordination
- Commerce
- Pharmacy Authorisation
- Care Messaging
- Client & Patient Records
- Identity & Access
- Notifications
- Operations

---

# Guiding Principles

- **Modular monolith first** - one NestJS application with hard module boundaries; split only when justified
- **Domain-Driven Design** - each domain owns its entities and business rules
- **Strong consistency for commerce and pharmacy** - orders, payments, and prescription decisions never rely on eventual consistency
- **Event-driven collaboration** - domains notify each other through domain events, not by reaching into each other's tables
- **Safety rails are domain rules** - the prescription guardrail lives in the Pharmacy domain, enforced in code, not in UI logic

---

# Domain Summary

### Clinic Content

The public-facing description of the clinic: what it does, what it costs, who works there, when it is open, and how to reach it. This domain owns the three-second emergency clarity requirement.

**Owns:** Service catalog pages with baseline pricing, staff profiles with credentials and accreditations, clinic hours and holiday hours, physical address and phone numbers, static information pages, downloadable care documents (post-op sheets, travel certificates, liability waivers), privacy policy and cookie consent pages.

**Key Entities:** Service, StaffProfile, ClinicLocation, BusinessHours, InformationPage, DownloadableDocument

### Publishing

The educational content engine: clinician-authored articles designed to build trust and local search authority.

**Owns:** Articles and their rich text bodies, categories (for example Dog Care, Cat Care, Puppy/Kitten Tips), tags (for example Nutrition, Seasonal Safety), article authors as references to staff profiles, embedded media, social sharing configuration, newsletter subscriber list, per-article SEO metadata (meta title, description, clean URL), related-article associations.

**Key Entities:** Article, Category, Tag, ArticleAuthor, MediaAsset, NewsletterSubscriber, SeoMetadata

### Care Coordination

The request surface between owners and the clinic: turning a website visit into scheduled care.

**Owns:** Appointment requests (owner details, pet details, reason for visit, preferred time windows), new-client registration submissions, uploaded medical history files, submission routing to clinic email, admin alerts raised when a form arrives.

**Key Entities:** AppointmentRequest, IntakeSubmission, HistoryUpload, FormSubmissionAlert

### Commerce

The storefront and everything between browsing and fulfilment.

**Owns:** Product catalog with the three product types (single, variable with per-variant price/SKU/stock, composite bundles), product categories (Prescription Medication, Therapeutic Diets, General Pet Supplies), search and filters (pet type, life stage, health condition), shopping cart, order lifecycle, tax and shipping calculation, fulfilment choice (home shipping or free in-clinic/curbside pickup), payment capture through the tokenised gateway, auto-refill and subscription billing.

**Key Entities:** Product, ProductVariant, Bundle, ProductCategory, Cart, CartItem, Order, OrderItem, Fulfilment, Payment, Subscription

### Pharmacy Authorisation

The clinical guardrail that sits inside checkout. This domain is small but carries the highest risk: no prescription-only product may leave without a veterinarian's decision.

**Owns:** Prescription authorisation requests raised by checkout, the mandatory patient-and-vet association (pet name and primary veterinarian on file), the review queue, veterinarian decisions (approve, reject, query), decision audit trail.

**Key Entities:** PrescriptionRequest, PrescriptionDecision, PatientAssociation

### Care Messaging

The WhatsApp front door: automated triage with a guaranteed path to a human.

**Owns:** WhatsApp conversations through the verified Business API, keyword auto-responder rules, panic-keyword detection for emergency routing, numbered pre-screening menus, bot-to-human handover state, out-of-hours schedules and away-message flows, the central shared inbox used by multiple receptionists.

**Key Entities:** Conversation, BotRule, MenuFlow, HandoverSession, BusinessHoursProfile, InboxMessage

### Client & Patient Records

The relationship data that makes care personal and that pharmacy authorisation depends on.

**Owns:** Client records (linked to identity accounts), pets and their basic profile, medical history references, each pet's primary veterinarian on file.

**Key Entities:** Client, Pet, MedicalHistoryRecord, PrimaryVeterinarianLink

### Identity & Access

Who anyone is, and what they are allowed to do.

**Owns:** Client accounts, staff accounts, roles (receptionist, veterinarian, vet technician, practice manager) treated additively, sessions, authentication, staff-only access policies.

**Key Entities:** User, Role, Session, StaffMembership

### Notifications

The delivery layer for everything the system needs to tell someone.

**Owns:** Email routing for form submissions, admin alerts for form submissions and bot escalations, newsletter delivery, out-of-hours auto-responses triggered by messaging.

**Key Entities:** Notification, DeliveryChannel, AlertSubscription

### Operations

The staff-facing control surface.

**Owns:** Admin alert dashboard, shared inbox presentation (consuming Care Messaging), catalog and price maintenance tools, content publishing tools, prescription review queue presentation, reporting views.

**Key Entities:** DashboardView, AuditLog

---

# Cross-Domain Dependencies

```
                    +------------------+
                    |    Identity &    |
                    |      Access      |
                    +--------+---------+
                             | authorises
        +--------------------+--------------------+
        |                    |                    |
+-------v--------+  +--------v---------+  +-------v--------+
| Clinic Content |  | Care Coordination|  |  Publishing    |
+-------+--------+  +--------+---------+  +-------+--------+
        |                    |                    |
        |             +------v-------+            |
        |             | Client &     |            |
        +------------>| Patient      |<-----------+ (author byline)
                      | Records      |
                      +------+-------+
                             |
              +--------------+--------------+
              |                             |
      +-------v--------+            +-------v--------+
      |   Commerce     +----------->|   Pharmacy     |
      +-------+--------+  requires  | Authorisation  |
              |                     +----------------+
      +-------v--------+
      | Notifications  |<---- (alerts from every domain)
      +----------------+

      Care Messaging  --->  Notifications + Care Coordination (human handover)
      Operations      --->  reads every domain, writes only through them
```

Dependency rules:

- Commerce may depend on Client & Patient Records and Pharmacy Authorisation, never the reverse
- Pharmacy Authorisation reads patient data but never writes commerce state; the decision flows back as an event
- Publishing references Clinic Content (staff profiles) read-only
- Notifications is a leaf: nothing depends on it, and it may be degraded without blocking a core flow
- Operations reads through each domain's public interface; it is never a data owner

---

# Key Domain Events

The full catalog with publisher/subscriber assignments lives in `domain-events.md`. The events that matter most to this model:

- `AppointmentRequested`, `IntakeSubmitted` (Care Coordination)
- `OrderPlaced`, `PaymentCaptured`, `OrderFulfilled` (Commerce)
- `PrescriptionRequested`, `PrescriptionApproved`, `PrescriptionRejected` (Pharmacy Authorisation)
- `EmergencyDetected`, `HandoverRequested`, `HandoverAccepted` (Care Messaging)
- `AlertRaised` (Notifications, consumed by Operations)

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Order | `Order` | Contains order lines, fulfilment, payment reference; strongly consistent |
| Prescription | `PrescriptionRequest` | Decision history is append-only and audited |
| Appointment | `AppointmentRequest` | Status lifecycle from submitted to triaged |
| Conversation | `Conversation` | Messages, menu state, and handover state move together |
| Product | `Product` | Variants and bundle components belong to it; stock is per-variant |
| Client | `Client` | Pets and history belong to it |
| Article | `Article` | Body, SEO metadata, and category links publish together |

---

# Acceptance Criteria

The domain model is considered correct when:

- Every checklist item in `functional-requirements.md` maps to exactly one owning domain
- No domain reads another domain's tables directly
- Every cross-domain interaction is either an inbound request through a public interface or a published domain event
- Pharmacy guardrail rules are expressed as domain rules, not UI behaviour

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `functional-requirements.md` | Feature source of truth this model must cover |
| `domain-events.md` | Event catalog for cross-domain communication |
| `bounded-context.md` | Context map, ownership, and integration patterns |
| `data-model/` | Table-level design derived from these entities |
| `architectural-decision-record.md` | Why a modular monolith with PostgreSQL was chosen |

# Guiding Principle

> **Every domain owns its rules and nothing borrows another domain's tables. A feature that cannot be explained as a domain owning an entity and publishing an event is a feature that will not survive contact with the second release.**
