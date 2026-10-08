# Domain Context Map

> **ruby-veterinary Strategic Architecture**
>
> **Document:** 01 — Domain Context Map
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary

---

# Purpose

Classifies all ten bounded contexts as Core, Supporting, or Generic, and shows upstream/downstream relationships. The authoritative ownership detail lives in `../bounded-context.md`; this document is the strategic summary used when planning features and changes.

---

# Classification

## Core Domains (differentiating value)

| Context | Strategic Importance |
|---------|---------------------|
| **Commerce** | Owns the online revenue line — all seven product types (simple, variable, grouped, external, virtual, downloadable, composite), subscriptions |
| **Pharmacy Authorisation** | Clinical safety guardrail; the clinic's licence to sell prescription items online |
| **Care Messaging** | Always-on WhatsApp triage with guaranteed human handover; after-hours presence |
| **Care Coordination** | Turns traffic into booked appointments and accepted clients |
| **Publishing** | Clinician-authored content is the local search and trust engine |

## Supporting Domains

| Context | Strategic Importance |
|---------|---------------------|
| **Clinic Content** | Public profile, hours, services, documents — the emergency handshake |
| **Client & Patient Records** | Pet and client data core domains depend on; standard record-keeping |
| **Notifications** | Delivery mechanics; provider-replaceable, contract-fixed leaf |

## Generic / Infrastructure Domains

| Context | Strategic Importance |
|---------|---------------------|
| **Identity & Access** | Standard authentication and additive role authorisation |
| **Operations** | Presentation over other contexts; owns almost no business rules |

---

# Context Map Diagram

PlantUML source: `domain-context-map.puml`.

```
                    Identity & Access  (generic upstream)
                              |
              authenticates / authorises all staff flows
                              |
   +------------+-------------+-------------+------------+------------+
   |            |             |             |            |            |
Clinic     Care Coord.   Publishing   Client & Pt.  Operations  Notifications
Content    (core)        (core)       Records       (generic)   (leaf)
   |            |             |             |            |            |
   |            +-------------+-------------+------------+------------+
   |                          |             |
   |                          | patient/vet |
   +------------------------> Commerce <----+   (core)
                                |  requires
                                v
                          Pharmacy Auth      (core, highest risk)
                                |
                        events back to Commerce

Care Messaging (core) ---> Notifications + Care Coordination (handover)
```

---

# Strategic Observations

### Identity Is The Ultimate Upstream

Every staff-facing flow authenticates through Identity. Public browsing and the emergency surface deliberately do not. This is why the emergency path works with no session (NFR: Reliability).

### Pharmacy Is The Safety Hinge

Commerce cannot complete prescription orders without Pharmacy events. The deepest dependency chain is Commerce → Pharmacy Authorisation → Client & Patient Records. Breakage at the bottom stops checkout — that is by design, not an accident.

### Notifications And Operations Are Leaves

Nothing critical depends on them. They can degrade to delay without blocking owners or clinicians.

### Publishing And Clinic Content Are Cache-Friendly Upstream Domains

Public reads here are served from Cloudflare + edge cache; their write paths are staff-only and low-frequency.

---

# Acceptance Criteria

- All ten contexts are classified exactly once
- Core context list matches `../bounded-context.md`
- The diagram matches the dependency rules in `../domain-model.md`

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `02-domain-event-flows.md` | What moves between these contexts |
| `03-context-relationships.md` | Integration pattern per pair |
| `04-ownership-matrix.md` | Who owns what |
| `../bounded-context.md` | Full context map and blast radius |

---

# Guiding Principle

> **Classification is a promise about where change hurts. Core contexts get the best tests; leaves get permission to fail politely.**
