# Domain Event Flows

> **ruby-veterinary Strategic Architecture**
>
> **Document:** 02 — Domain Event Flows
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary

---

# Purpose

Maps the critical business flows as sequences of domain events: what happened, what triggered the next step, and which contexts reacted. The full event catalog with publisher/subscriber matrix lives in `../domain-events.md`; this document is the narrative layer for planning and incident reasoning.

---

# Event Flow Principles

- Events are facts in the past tense: `OrderPlaced`, never `OrderPlacing`
- Publishers never block on consumers; a consumer outage never fails the publisher
- Every money-moving or clinically sensitive flow has at least one audit-visible consequence
- Idempotency keys at the API boundary prevent duplicate events from client retries

---

# Flow 1 — Appointment Request

```
Owner submits appointment form
  → Care Coordination creates AppointmentRequest (status=received)
  → publishes AppointmentRequested
  → Notifications enqueues admin alert (email + dashboard)
  → Operations dashboard shows new intake
Staff triages
  → status received → triaged → scheduled | declined
```

Trigger: website form. No account required. Degradation: if alert delivery fails, dashboard still shows the request.

---

# Flow 2 — New Client Intake

```
Owner completes intake form, optionally signs upload
  → POST /uploads/sign (Intake) → owner PUTs to MinIO
  → IntakeSubmission created (historyUploadIds attached)
  → publishes IntakeSubmitted
  → Notifications alerts staff
Staff reviews, accepts
  → Client & Patient Records module creates Client + Pet rows
  → status accepted → owner receives confirmation email
```

Acceptance writes never happen from the intake schema directly — the Clients context module owns the write.

---

# Flow 3 — Order Placement Without Prescription Items

```
Owner checks out
  → Commerce revalidates stock + totals (STOCK_CHANGED on conflict)
  → Order created (pending_payment), stock reserved
  → publishes OrderPlaced
  → payment token captured via gateway
Payment webhook arrives
  → Payments webhook verified + deduped
  → PaymentCaptured published
  → Order status → paid
Fulfilment completes
  → OrderFulfilled published → Notifications informs owner
```

---

# Flow 4 — Order With Prescription Items (The Guardrail)

```
Owner checks out Rx-containing order
  → Commerce creates Order (awaiting_prescription)
  → publishes OrderPlaced + PrescriptionRequested (one per Rx order line set)
  → Pharmacy stores PrescriptionRequest (pending)
  → owner sees PRESCRIPTION_PENDING on order status (never a blank error)
Veterinarian reviews queue
  → approve → PrescriptionApproved + rx_audit_log row
  → reject → PrescriptionRejected + reason
  → query → PrescriptionQueried (owner/staff loop)
On approval
  → Commerce listens, moves order → paid (after payment) / fulfilling
  → stock released for fulfilment
  → Notifications informs owner of Rx decision
```

Invariant: no code path moves an Rx order to `paid` without Pharmacy events. Manual offline fallback exists for outages (NFR graceful degradation) and is itself audited.

---

# Flow 5 — WhatsApp Emergency Escalation

```
Owner sends WhatsApp message
  → Meta webhook verified, immediate 200, async process
  → dedupe by provider message id
  → Care Messaging matches bot rules
Panic keyword hit
  → EmergencyDetected published (highest priority)
  → conversation.emergency_flag = true
  → Notifications pages staff (WhatsApp + dashboard)
  → bot pauses; handover session requested
Receptionist claims conversation
  → HandoverAccepted; bot stays paused
  → agent replies via admin inbox
```

Latency budget: keyword rules fire within 2s. Fallback if messaging is down: website sticky call bar remains the emergency path.

---

# Flow 6 — Article Publishing

```
Staff author writes draft
  → Article saved (status=draft); invisible to public
Staff publishes
  → ArticlePublished published
  → CDN cache invalidated for blog routes
  → Notifications may dispatch newsletter (if scheduled)
  → SEO metadata shipped with article
```

---

# Flow 7 — Auto-Refill Subscription

```
Scheduler finds subscriptions due
  → Commerce revalidates stock + Rx state
  → if Rx active: place order like Flow 4
  → if Rx missing/expired: pause subscription + notify owner
  → next_order_at advanced
```

---

# Acceptance Criteria

- Every event in `../domain-events.md` appears in at least one flow above or is explicitly marked as low-frequency utility
- Flows 4 and 5 have documented degradation behaviour
- No flow requires a consumer to be online for the publisher to succeed

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-events.md` | Canonical event catalog |
| `03-context-relationships.md` | Pattern used by each hop above |
| `05-communication-matrix.md` | Failure handling per channel |
| `../database/backup-and-recovery.md` | Recovery of flows after data loss |

---

# Guiding Principle

> **If you cannot narrate a feature as a chain of events with named publishers, it is not designed yet — it is only described.**
