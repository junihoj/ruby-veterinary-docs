# Communication Matrix

> **ruby-veterinary Strategic Architecture**
>
> **Document:** 05 — Communication Matrix
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary

---

# Purpose

Documents every communication path between contexts and external systems: mechanism, protocol, consistency, delivery guarantee, and failure handling. This is the operational companion to `03-context-relationships.md`.

---

# Internal Paths (Monolith)

| Path | Mechanism | Guarantee | Failure Handling |
|------|-----------|-----------|------------------|
| API guards → Identity | In-process read | Strong | 401/403; no retry |
| Checkout → Pharmacy hold | Sync module call | Strong | Request fails with clear Rx message; no partial order |
| Pharmacy decision → Commerce | Domain event | At-least-once | Idempotent handler; status projection tolerates duplicates |
| Any context → Notifications | Domain event | At-least-once | Notifications failure never rolls back publisher |
| Operations inbox claim | Sync module call | Strong (first claim wins) | 409 CONFLICT to loser; UI re-reads |
| Bot rules → conversation state | Sync write in messaging schema | Strong | Bot degrades to fallback reply with phone number |

Event delivery inside the process uses the in-process event bus with durable handler retries where the business consequence demands it (prescription, order status). Handlers must be idempotent by design; event ids are unique and stored where replay matters.

---

# External Paths

| Path | Protocol | Signature / Auth | Delivery | Failure Handling |
|------|----------|------------------|----------|------------------|
| Meta WhatsApp inbound | HTTPS webhook | `X-Provider-Signature` | At-least-once | Verify → 200 immediately → async; dedupe by provider message id; rejections alert Operations |
| Payment gateway webhook | HTTPS webhook | Provider signature | At-least-once | Idempotent by provider event id; failures surface in dashboard; checkout degrades to 503 with phone fallback |
| Email delivery | SMTP/API via adapter | Provider creds in VPS env | At-least-once with bounded retries | Failed attempts alert; dashboard remains system of record |
| WhatsApp outbound (staff/bot) | Business Cloud API | Provider creds in VPS env | Provider-defined | Retry then mark failed; conversation shows send failure |
| MinIO object storage | S3 API (local) | Access keys in VPS env | Request/response | Uploads fail closed; signed URLs short-lived |
| Cloudflare CDN | HTTPS edge | Zone API token | n/a | Origin serve-through; stale-while-revalidate for public cache |

---

# Delivery Guarantees Summary

| Class | Guarantee | Consumer Requirement |
|-------|-----------|---------------------|
| Domain events (in-process) | At-least-once | Idempotent by event id |
| Inbound webhooks | At-least-once | Dedupe by provider id |
| Checkout / Rx decisions | Exactly-once **effect** via idempotency keys + status machines | Never assume single delivery |
| Outbound notifications | At-least-once attempts, bounded | Surface terminal failures |

---

# Degradation Ladder (By Channel Loss)

| Lost channel | What still works | Owner-visible fallback |
|--------------|------------------|------------------------|
| WhatsApp bot | Website, phone | Sticky call bar + direct WhatsApp still reachable |
| Payment gateway | Browse, cart | 503 with plain-language instruction + clinic phone |
| Email | Dashboard alerts, WhatsApp alerts | Staff see submissions in admin UI |
| CDN | Origin serves slowly | Emergency pages prioritised at origin |
| Entire VPS | Offsite backup restore path | Emergency content from CDN cache; phone published in DNS TXT if needed |

---

# Observability Requirements

| Signal | Purpose |
|--------|---------|
| Event publish/consume counters by type | Detect silent subscriber loss |
| Webhook rejection alerts | Security + provider misconfig |
| Delivery attempt failure rate | Provider health |
| Handler retry depth | Poison-event detection |
| Correlation `requestId` across API + events + logs | Incident reconstruction |

---

# Acceptance Criteria

- Every cross-context and external path in `../bounded-context.md` and `../api-specification.md` appears in this matrix
- Every external path documents signature verification
- Every degradation row is reflected in UI requirements (`../ui-ux/patterns-emergency-first.md`)

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `03-context-relationships.md` | Pattern per pair |
| `../api-specification.md` | Webhook processing contracts |
| `../deployment-architecture.md` | Where paths run |
| `../ui-ux/patterns-emergency-first.md` | UI fallbacks for each lost channel |

---

# Guiding Principle

> **Assume every dependency will fail at 2am. The matrix is how you prove the clinic can still answer the phone.**
