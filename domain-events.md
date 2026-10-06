# domain-events.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Domain Events
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

This document is the catalog of domain events published across ruby-veterinary and the matrix of who publishes and consumes each one.

Events are the only sanctioned mechanism for one domain to inform another. A domain that needs data another domain owns subscribes to its events or calls a public interface; it never reads foreign tables. See `domain-model.md` for ownership.

---

# Principles

- Events describe facts that have already happened, never instructions
- Publishing is fire-and-forget within the publishing transaction boundary; consumers must tolerate retries
- Event names are past-tense and domain-scoped
- Ordering is guaranteed only per aggregate (one aggregate's events arrive in order)
- Failed consumers raise an operational alert rather than blocking the publisher

---

# Event Catalog

## Care Coordination

| Event | Payload Essentials | Raised When |
|-------|--------------------|-------------|
| `AppointmentRequested` | request id, owner contact, pet, reason, preferred windows | Appointment form submitted |
| `IntakeSubmitted` | submission id, owner, pets, uploaded file references | New-client registration submitted |
| `IntakeComplete` | submission id | Staff accepted and filed an intake |
| `HistoryUploadReceived` | file id, submission id, format | PDF/JPEG stored successfully |

## Commerce

| Event | Payload Essentials | Raised When |
|-------|--------------------|-------------|
| `CartAbandoned` | cart id, items, owner reference | Cart idle past threshold |
| `OrderPlaced` | order id, items, fulfilment method | Checkout completed |
| `PaymentCaptured` | order id, gateway reference, amount | Gateway confirmed capture |
| `PaymentFailed` | order id, reason code | Gateway declined or errored |
| `OrderFulfilled` | order id, method (ship / pickup) | Order handed to customer or carrier |
| `SubscriptionActivated` | subscription id, cadence, next billing date | Auto-refill enrolment confirmed |
| `SubscriptionCancelled` | subscription id, reason | Owner or staff cancelled |

## Pharmacy Authorisation

| Event | Payload Essentials | Raised When |
|-------|--------------------|-------------|
| `PrescriptionRequested` | request id, order id, pet, prescriber, items | Order containing Rx items reached review |
| `PrescriptionApproved` | request id, veterinarian id, decision note | Veterinarian authorised |
| `PrescriptionRejected` | request id, veterinarian id, reason | Veterinarian declined |
| `PrescriptionQueried` | request id, question | Veterinarian requested more information |

## Care Messaging

| Event | Payload Essentials | Raised When |
|-------|--------------------|-------------|
| `ConversationStarted` | conversation id, channel, contact | First inbound WhatsApp message |
| `EmergencyDetected` | conversation id, matched keyword, surfaced number | Panic keyword matched |
| `HandoverRequested` | conversation id, trigger (user request or bot limit) | Bot paused, receptionist paged |
| `HandoverAccepted` | conversation id, agent id | Receptionist claimed the conversation |
| `OutOfHoursResponderActivated` | schedule id | Business hours profile flipped |

## Clinic Content, Publishing, Identity

| Event | Payload Essentials | Raised When |
|-------|--------------------|-------------|
| `ArticlePublished` | article id, author id, category, slug | Staff published an article |
| `ArticleUnpublished` | article id, reason | Article withdrawn |
| `NewsletterSubscribed` | subscriber id, source | Email captured |
| `ClientRegistered` | client id, user id | New client account created |
| `StaffRoleGranted` | user id, role | Practice manager granted a role |

## Notifications (raised, not a consumer of itself)

| Event | Payload Essentials | Raised When |
|-------|--------------------|-------------|
| `AlertRaised` | source, severity, reference | Any form submission or bot escalation needing a human |
| `NotificationDelivered` | notification id, channel, status | Email or alert delivery confirmed |

---

# Publisher / Subscriber Matrix

| Event | Publisher | Subscribers |
|-------|-----------|-------------|
| `AppointmentRequested`, `IntakeSubmitted` | Care Coordination | Notifications, Operations |
| `OrderPlaced` | Commerce | Pharmacy Authorisation (if Rx items), Notifications |
| `PaymentCaptured`, `PaymentFailed` | Commerce | Notifications, Operations |
| `SubscriptionActivated` | Commerce | Notifications |
| `PrescriptionRequested` | Pharmacy Authorisation | Notifications (veterinarian alert), Operations |
| `PrescriptionApproved` / `PrescriptionRejected` | Pharmacy Authorisation | Commerce (release or release-hold), Notifications |
| `EmergencyDetected` | Care Messaging | Notifications (high severity), Operations |
| `HandoverRequested` | Care Messaging | Operations (shared inbox), Notifications (receptionist page) |
| `ArticlePublished` | Publishing | Clinic Content (render), Notifications (digest) |
| `ClientRegistered` | Identity & Access | Care Coordination, Notifications |
| `AlertRaised` | any domain | Notifications, Operations dashboard |

---

# Acceptance Criteria

The event catalog is considered complete when:

- Every cross-domain dependency drawn in `domain-model.md` has at least one corresponding event
- Every event has exactly one publisher
- Every subscriber can be built without querying the publisher's tables
- Emergency-path events (`EmergencyDetected`, `HandoverRequested`) have retry and alerting behaviour defined

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `domain-model.md` | Domain ownership that this catalog reflects |
| `bounded-context.md` | Integration patterns applied to these events |
| `api-specification.md` | HTTP endpoints that raise the same facts for external callers (webhooks) |
| `functional-requirements.md` | Feature behaviours these events implement |

# Guiding Principle

> **Domains talk through facts, not through each other's tables. If a behaviour cannot be expressed as something that happened, it is not yet designed.**
