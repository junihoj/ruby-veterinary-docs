# bounded-context.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Bounded Contexts
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

This document defines the strategic architecture of ruby-veterinary: how the platform is carved into bounded contexts, which of them are core versus supporting, who owns what, how contexts integrate, and what breaks when a context fails.

It turns the domains in `domain-model.md` into deployable and ownership boundaries inside the modular NestJS monolith. Context boundaries are enforced by module boundaries in code; the pattern of communication between them is fixed here.

---

# Context Map

```
                        +---------------------------+
                        |      Identity & Access    |
                        |  (accounts, roles, auth)  |
                        +-------------+-------------+
                                      |
                       authenticates / authorises
                                      |
   +-------------------+--------------+---------------+--------------------+
   |                   |              |               |                    |
   v                   v              v               v                    v
+--------------------+  +------------------+  +-----------------+  +-------------------+
| Clinic Content     |  | Care Coordination|  | Publishing      |  | Operations        |
| (site, services,   |  | (forms, intake,  |  | (blog, CMS,     |  | (back office,     |
|  staff, docs)      |  |  appointments)   |  |  newsletter)    |  |  dashboards)      |
+---------+----------+  +--------+---------+  +--------+--------+  +---------+---------+
          |                      |                     |                     |
          | read-only            | uses                | author byline       | reads all
          | staff profiles       v                     v                     |
          |            +------------------+   +-----------------+           |
          |            | Client & Patient |   | Notifications   |<----------+
          |            | Records          |   | (delivery leaf) |
          |            +--------+---------+   +-----------------+
          |                     |
          |                     | patient / prescriber
          |                     v
          |      +------------------------+       +----------------------+
          +----->| Commerce               |------>| Pharmacy             |
                 | (catalog, cart, orders,| holds  | Authorisation        |
                 |  payments, subs)       |       | (vet review, audit)  |
                 +-----------+------------+       +----------------------+
                             |
                             | order notifications
                             v
                 +------------------------+
                 | Care Messaging         |-----> Care Coordination (handover)
                 | (WhatsApp bot, inbox)  |-----> Notifications (alerts)
                 +------------------------+

   Notification delivery: outbound email / WhatsApp via third-party providers
```

---

# Context Classification

## Core Domains (differentiating value)

| Context | Why It Is Core |
|---------|----------------|
| **Commerce** | Owns the online revenue line, including variable products, bundles, and subscriptions |
| **Pharmacy Authorisation** | The clinical safety guardrail; the clinic's licence to sell prescription items online |
| **Care Messaging** | Always-on triage with a guaranteed human path; the clinic's after-hours presence |
| **Care Coordination** | Turns traffic into booked appointments and accepted clients |
| **Publishing** | Clinician-authored content is the local search and trust engine |

## Supporting Domains (necessary, not differentiating)

| Context | Why It Is Supporting |
|---------|----------------------|
| **Clinic Content** | Static presentation of a fixed clinic profile |
| **Client & Patient Records** | Needed by core domains but standard record-keeping |
| **Notifications** | Delivery mechanics; replaceable provider, fixed contract |

## Generic / Infrastructure Domains

| Context | Why It Is Generic |
|---------|-------------------|
| **Identity & Access** | Standard authentication and role-based authorisation |
| **Operations** | Presentation over other domains; owns no business rules |

---

# Context Dependency Depth

| Depth | Context | Depends On |
|-------|---------|------------|
| 0 (leaf) | Notifications, Operations | nothing depends on them internally |
| 1 | Identity & Access | nothing (authenticates everyone else) |
| 2 | Clinic Content, Publishing, Client & Patient Records | Identity |
| 3 | Care Coordination, Care Messaging | Client & Patient Records, Notifications |
| 4 | Commerce | Client & Patient Records, Pharmacy Authorisation |
| 5 | Pharmacy Authorisation | Client & Patient Records, Identity (veterinarian role) |

The deepest chain is Commerce -> Pharmacy Authorisation -> Client & Patient Records. Failures at the bottom of this chain stop checkout, which is why patient data access is treated as a critical path in blast-radius analysis.

---

# Integration Patterns

| Pattern | Where It Is Used | Why |
|---------|------------------|-----|
| **Direct module call (synchronous)** | Commerce -> Pharmacy Authorisation to place a hold; Operations -> anything for read models | Same process, strong consistency required |
| **Domain events (asynchronous)** | Every entry in `domain-events.md` | Loose coupling, retry tolerance, no foreign-table reads |
| **Webhook (external inbound)** | WhatsApp Business API and payment gateway callbacks | Third parties cannot call inside the process |
| **Outbound adapter** | Notifications -> email/WhatsApp providers; Commerce -> payment gateway | External variability isolated behind an interface |

### Where Each Pattern Is Used

- Synchronous calls are permitted only for operations that must succeed or fail in the user's request
- Asynchronous events carry every cross-context notification; a consumer outage never fails the publisher
- Webhooks are validated by signature, idempotent by event id, and always produce an alert when rejected
- Outbound adapters implement a narrow port so a provider can be swapped (for example, a second payment gateway) without touching domain code

---

# Context Ownership Summary

| Context | Owns Exclusively | Never Writes |
|---------|------------------|--------------|
| Identity & Access | users, roles, sessions | client profiles, pet data |
| Clinic Content | services, staff profiles, hours, pages, documents | articles, orders |
| Publishing | articles, categories, tags, newsletter subscribers | staff profiles (reads them) |
| Care Coordination | appointment requests, intake submissions, uploads | patient records, orders |
| Client & Patient Records | clients, pets, history, primary-vet links | identity credentials, commerce state |
| Commerce | products, variants, bundles, carts, orders, payments, subscriptions | prescription decisions, pet records |
| Pharmacy Authorisation | prescription requests, decisions, audit trail | order state (emits events instead) |
| Care Messaging | conversations, bot rules, menus, handover, schedules | appointment records (emits events) |
| Notifications | notifications, delivery attempts, alert subscriptions | business objects it only reports on |
| Operations | audit log, dashboard preferences | everything else (reads through interfaces) |

---

# Rules

1. No context reads another context's tables; integration is via public module interfaces or events
2. Every context owns its migrations and schema namespace within the shared PostgreSQL database
3. A context may be extracted into its own service only when the modular monolith's limits are measured, not assumed (see `architectural-decision-record.md`)
4. Core contexts cannot be degraded silently; supporting and generic contexts degrade to defined fallbacks
5. Cross-context foreign keys are prohibited; reference by id and resolve through the owning context
6. New contexts require an update to this document and to `domain-model.md` in the same change

---

# Blast Radius Analysis

| Impact | Context Down | Consequence |
|--------|--------------|-------------|
| **Critical** | Identity & Access | No one can sign in; public browsing still works, everything behind a login stops |
| **Critical** | Client & Patient Records | Checkout and intake blocked at the patient association step |
| **High (clinical safety)** | Pharmacy Authorisation | Rx orders cannot be released; manual offline authorisation fallback engages |
| **High (revenue)** | Commerce | Storefront purchasing unavailable; site, CMS, and booking unaffected |
| **Medium** | Care Coordination | Appointment and intake forms degrade to a phone number fallback (never a dead end) |
| **Medium** | Care Messaging | Bot unavailable; click-to-chat and phone fallback shown, WhatsApp direct still reachable |
| **Low** | Publishing | Existing articles stay served from cache; new publishing unavailable |
| **Low** | Clinic Content | Cached static pages served; edits unavailable |
| **Low** | Notifications | Alerts delayed; the dashboard still shows submissions |
| **Low** | Operations | Staff lose dashboards temporarily; underlying flows continue |

Graceful degradation for every medium-or-higher entry is a binding requirement in `non-functional-requirements.md` (Reliability & Availability).

---

# Acceptance Criteria

The context map is considered correct when:

- Every domain in `domain-model.md` maps to exactly one context
- Every integration listed uses one of the four sanctioned patterns
- Every core context has an explicitly documented blast-radius consequence
- No context in the ownership table has a "writes" entry that belongs to another context

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `domain-model.md` | Domains and entities these contexts own |
| `domain-events.md` | Events exchanged across context boundaries |
| `architectural-decision-record.md` | Decisions on monolith, database, and extraction criteria |
| `strategic-architecture/` | Detailed context maps and architecture rules (planned) |

# Guiding Principle

> **A bounded context is a promise that change stays local. Draw the line where the vocabulary diverges, let each context speak its own language, and when in doubt keep the boundary and simplify the code inside it.**
