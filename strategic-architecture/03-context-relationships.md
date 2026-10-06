# Context Relationships

> **ruby-veterinary Strategic Architecture**
>
> **Document:** 03 — Context Relationships
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary

---

# Purpose

Defines the sanctioned integration pattern for every context pair that talks, and analyses the blast radius when a context changes or fails. Patterns are drawn from `../bounded-context.md` (Integration Patterns) and elaborated here for planning.

---

# Integration Patterns

| Pattern | Mechanism | Consistency |
|---------|-----------|-------------|
| **Synchronous module call** | In-process NestJS module interface | Strong; fails the request |
| **Domain event** | In-process event bus → handlers | Eventual; at-least-once with idempotent handlers |
| **Inbound webhook** | HTTP + signature verification | At-least-once from provider; dedupe required |
| **Outbound adapter** | Narrow port → provider SDK | Provider-defined; timeouts map to graceful degradation |

Synchronous calls are permitted only when the operation must succeed or fail inside the user's request (checkout quote, prescription hold placement, inbox claim).

---

# Context Pair Matrix

| From → To | Pattern | Notes |
|-----------|---------|-------|
| All → Identity | Sync read (guards) | Never writes identity tables |
| Commerce → Pharmacy | Sync hold + events back | Strong consistency for Rx gate |
| Pharmacy → Commerce | Events only | Never writes order state |
| Commerce → Client & Patient | Sync read (pet claim) | Id references only |
| Pharmacy → Client & Patient | Sync read (pet + primary vet) | Guardrail input |
| Care Coordination → Notifications | Events | Alert never blocks submission |
| Care Messaging → Notifications | Events | Emergency priority |
| Care Messaging → Care Coordination | Events after handover | Booking intent |
| Publishing → Notifications | Events | Newsletter on publish |
| Publishing → Clinic Content | Sync read | Staff bylines |
| Operations → All | Sync read via public modules | Never direct table reads |
| Commerce → Payments gateway | Outbound adapter | Tokenised; no card data |
| Care Messaging → WhatsApp | Outbound + inbound webhook | Signature-verified |
| Notifications → Email/WhatsApp | Outbound adapter | Retry + alert on failure |

---

# Blast Radius Of Change

| If this context changes | Who feels it | Mitigation |
|-------------------------|--------------|------------|
| Identity | Everything staff-facing | Public emergency surface unaffected; feature-flag auth changes |
| Client & Patient Records | Checkout, Rx, intake acceptance | Contract tests on id references; no schema drift |
| Pharmacy Authorisation | Checkout completion | Manual offline authorisation fallback + audit |
| Commerce | Storefront revenue | CDN keeps catalog browsable; checkout shows phone fallback |
| Care Messaging | Bot, inbox | Sticky call bar + direct WhatsApp still reachable |
| Care Coordination | Forms | Phone fallback always present |
| Publishing | Blog | Cached articles stay served |
| Clinic Content | Public profile | Cached pages; edits unavailable |
| Notifications | Delivery timing | Dashboard remains system of record |
| Operations | Staff dashboards | Underlying flows continue |

---

# Coupling Rules

1. A context may depend on another's **published language** (events, module interfaces) — never its tables
2. New event subscribers must be idempotent; duplicate delivery is normal
3. Shared vocabulary (ids, timestamps) is consistent; shared mutable state is forbidden
4. Extracting a context to a remote service converts sync module calls into network calls — every pair listed as "sync" must be re-evaluated first (ADR required)

---

# Acceptance Criteria

- Every cross-context call in code maps to a pattern in the matrix
- No undocumented context pair appears in production traffic
- Blast radius table covers all ten contexts

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../bounded-context.md` | Source pattern list and dependency depth |
| `02-domain-event-flows.md` | Concrete sequences using these pairs |
| `05-communication-matrix.md` | Failure semantics per channel |
| `06-architecture-rules.md` | MUST rules enforcing this matrix |

---

# Guiding Principle

> **Relationships are contracts. Write the pattern down before you write the import, because imports never stay local.**
