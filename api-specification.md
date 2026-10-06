# api-specification.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** API Specification
>
> **Version:** 1.0.0
>
> **Status:** Living Specification
>
> **Owner:** ruby-veterinary
>
> **Classification:** API Contract

---

# Purpose

This document defines the REST contract between `ruby-veterinary-web-frontend` and `ruby-veterinary-api`, and the inbound webhooks the API accepts from third parties.

It is the source of truth for resources, endpoints, payloads, authentication, and error behaviour. Engineering standards live in `engineering-guidelines.md`; machine-readable definitions are generated into `openapi/`.

---

# API Design Principles

- Consistent regardless of which bounded context owns the resource
- Predictable: plural nouns, standard verbs, standard envelopes
- Secure: least privilege by default, no secrets in URLs, no card data ever accepted
- Versioned: `/api/v1` prefix; breaking changes require a new major prefix
- Backward compatible: fields are added, never repurposed, within a version
- Fast: every public read must be cacheable and well within the sub-3s page budget

# Versioning Strategy

| Element | Rule |
|---------|------|
| Base path | `/api/v1` |
| Additive change | Same version; unknown response fields must be ignored by clients |
| Breaking change | New version prefix; old version supported for one release cycle |
| Deprecation | `Deprecation` and `Sunset` headers before removal |

# Authentication

| Surface | Mechanism |
|---------|-----------|
| Public reads (services, staff, hours, articles, catalog) | No authentication |
| Client actions (intake, orders, subscriptions) | `Authorization: Bearer <access token>` with client role claim |
| Staff back office, prescription decisions, catalog writes | Bearer token with additive staff role claims |
| Inbound webhooks (WhatsApp, payments) | Provider signature verification, not bearer tokens |
| Emergency surface | Must function with no session; never redirect to login |

Tokens: short-lived access token plus rotating refresh token (ADR-0011). Public browsing never requires a session.

# Common Response Format

```json
{
  "data": { "id": "apt_01H...", "status": "received" },
  "meta": { "requestId": "req_01H...", "timestamp": "2026-10-06T10:00:00Z" }
}
```

| Convention | Rule |
|------------|------|
| Collections | `data` is an array; pagination under `meta.pagination` (`limit`, `cursor` or `total`) |
| Creation | `201` with the created resource |
| No content | `204` with empty body |
| Idempotent retries | Same response as the original success |
| Identifiers | Prefixed opaque strings (`ord_`, `pet_`, `rx_`), never sequential integers |

# Error Model

```json
{
  "error": {
    "code": "PRESCRIPTION_REQUIRED",
    "message": "This order requires veterinary authorisation.",
    "details": [{ "field": "items.0.productId", "issue": "rx_only" }],
    "requestId": "req_01H..."
  }
}
```

| HTTP | `code` Examples | Meaning |
|------|-----------------|---------|
| `400` | `VALIDATION_FAILED` | Input failed schema or field validation; `details` lists fields |
| `401` | `TOKEN_EXPIRED`, `INVALID_CREDENTIALS` | Missing or invalid authentication |
| `403` | `STAFF_ONLY`, `PRESCRIBER_REQUIRED` | Authenticated but not permitted for this action |
| `404` | `RESOURCE_NOT_FOUND` | Absent or hidden from this caller |
| `409` | `IDEMPOTENCY_CONFLICT`, `STOCK_CHANGED` | State conflict; safe to re-read and retry |
| `413` | `UPLOAD_TOO_LARGE` | Upload exceeds size limits |
| `422` | `PRESCRIPTION_PENDING` | Business rule blocks completion (Rx hold) |
| `429` | `RATE_LIMITED` | Throttled; `Retry-After` header set |
| `500` | `INTERNAL_ERROR` | Unexpected; request id logged for correlation |
| `503` | `DEPENDENCY_UNAVAILABLE` | External provider down; graceful fallback applies |

Error `code` values are stable API surface: clients branch on `code`, never on `message`.

---

# API Domains

## Auth API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `POST` | `/api/v1/auth/register` | Public | Create a client account |
| `POST` | `/api/v1/auth/login` | Public | Issue access + refresh tokens |
| `POST` | `/api/v1/auth/refresh` | Refresh token | Rotate tokens |
| `POST` | `/api/v1/auth/logout` | Bearer | Revoke refresh token |
| `GET` | `/api/v1/auth/me` | Bearer | Current identity and roles |

## Clinic Content API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `GET` | `/api/v1/clinic/hours` | Public | Standard and holiday hours |
| `GET` | `/api/v1/clinic/contact` | Public | Phone, address, emergency numbers |
| `GET` | `/api/v1/services` | Public | Service catalog with baseline pricing |
| `GET` | `/api/v1/services/:slug` | Public | Single service page |
| `GET` | `/api/v1/staff` | Public | Staff profiles grid |
| `GET` | `/api/v1/documents` | Public | Downloadable care sheets, waivers, travel certificates |
| `GET` | `/api/v1/documents/:id` | Public | Signed download link |

## Intake & Appointments API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `POST` | `/api/v1/appointment-requests` | Optional | Submit intake form; email routing fires |
| `POST` | `/api/v1/clients/intake` | Public or Bearer | New-client registration with history files |
| `POST` | `/api/v1/uploads/sign` | Bearer | Issue signed upload URL for PDF/JPEG history |
| `GET` | `/api/v1/clients/intake/:id/status` | Bearer | Submission status |

`POST` bodies are idempotency-safe: retries with an `Idempotency-Key` header return the original result.

## Publishing API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `GET` | `/api/v1/articles` | Public | List with category/tag filters and pagination |
| `GET` | `/api/v1/articles/:slug` | Public | Article with author, related feed, SEO metadata |
| `POST` | `/api/v1/articles` | Staff (`content:write`) | Create draft |
| `PATCH` | `/api/v1/articles/:id` | Staff (`content:write`) | Edit; carries meta title, description, slug |
| `POST` | `/api/v1/articles/:id/publish` | Staff (`content:write`) | Publish; raises `ArticlePublished` |
| `GET`/`POST`/`PATCH` | `/api/v1/categories`, `/api/v1/tags` | Staff | Taxonomy management |
| `POST` | `/api/v1/newsletter/subscribe` | Public | Email capture (rate limited) |

## Catalog & Commerce API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `GET` | `/api/v1/products` | Public | Filter by pet type, life stage, condition; category facets |
| `GET` | `/api/v1/products/:slug` | Public | Product detail incl. variants and bundle contents |
| `GET`/`POST`/`PATCH`/`DELETE` | `/api/v1/cart`, `/api/v1/cart/items` | Bearer | Persistent cart |
| `POST` | `/api/v1/checkout/quote` | Bearer | Totals: tax, shipping, pickup |
| `POST` | `/api/v1/checkout` | Bearer | Place order; returns `PRESCRIPTION_PENDING` when Rx items present |
| `GET` | `/api/v1/orders` | Bearer | Order history with fulfilment state |
| `GET` | `/api/v1/orders/:id` | Bearer | Single order incl. Rx status |
| `POST`/`PATCH`/`DELETE` | `/api/v1/subscriptions` | Bearer | Auto-refill management |
| `POST` | `/api/v1/admin/products` etc. | Staff (`catalog:write`) | Catalog maintenance |

Checkout accepts an `Idempotency-Key`; stock is reserved per variant (`STOCK_CHANGED` on conflict).

## Pharmacy Authorisation API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `GET` | `/api/v1/prescriptions/queue` | Staff (`rx:review`) | Review queue |
| `GET` | `/api/v1/prescriptions/:id` | Staff (`rx:review`) | Request with patient history and prescriber context |
| `POST` | `/api/v1/prescriptions/:id/approve` | Staff (`rx:review`) | Approve; raises `PrescriptionApproved`, writes audit row |
| `POST` | `/api/v1/prescriptions/:id/reject` | Staff (`rx:review`) | Reject with reason |
| `POST` | `/api/v1/prescriptions/:id/query` | Staff (`rx:review`) | Request more information |

Owner-facing status is read-only through `GET /orders/:id`. The decision endpoint is the only place an Rx order can be released, and it always writes the append-only audit trail (ADR-0012).

## Care Messaging Webhooks

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `GET` | `/api/v1/webhooks/whatsapp/verify` | Signature | Meta webhook verification handshake |
| `POST` | `/api/v1/webhooks/whatsapp` | Signature | Inbound messages, status callbacks |

Processing contract:

1. Verify provider signature before parsing
2. Acknowledge within provider timeout (immediate `200`), process asynchronously
3. Deduplicate by provider message id (at-least-once delivery)
4. Keyword rules fire within the 2s bot latency budget; panic keywords raise `EmergencyDetected` with highest priority
5. `HandoverRequested` pages the shared inbox; bot stays paused until `HandoverAccepted`

## Payments Webhooks

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `POST` | `/api/v1/webhooks/payments` | Signature | Capture confirmations, failures, refunds |

Idempotent by provider event id; maps to `PaymentCaptured` / `PaymentFailed`. Rejections alert Operations.

## Notifications & Operations API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `GET` | `/api/v1/admin/alerts` | Staff (`ops:read`) | Form submissions and bot escalations |
| `GET` | `/api/v1/admin/inbox/conversations` | Staff (`ops:inbox`) | Shared inbox state, concurrent agents supported |
| `POST` | `/api/v1/admin/inbox/conversations/:id/claim` | Staff (`ops:inbox`) | Accept handover |
| `POST` | `/api/v1/admin/inbox/conversations/:id/messages` | Staff (`ops:inbox`) | Agent reply |

---

# Rate Limits

| Surface | Limit | Behaviour |
|---------|-------|-----------|
| Public reads | 120 req/min per IP | Cached upstream; `429` with `Retry-After` beyond |
| Form and newsletter submits | 5 per 10 min per IP | Spam control |
| Auth endpoints | 10 per 10 min per IP | Brute-force control |
| Webhooks | Provider retry schedule | Deduplicated, never rate-limited into data loss |
| Staff writes | Generous | Throttled only to protect shared inboxes |

# Idempotency

Every money-moving or record-creating POST (`checkout`, `checkout/quote` is read-only, `prescriptions/:id/approve`, intake submissions) accepts an `Idempotency-Key` header. Replayed keys within 24 hours return the original response; conflicting bodies under the same key return `409 IDEMPOTENCY_CONFLICT`.

# File Upload API

Uploads (medical history PDF/JPEG, CMS media) use signed direct-to-object-storage URLs:

1. `POST /api/v1/uploads/sign` returns a short-lived signed URL and object key
2. Client uploads directly to object storage (ADR-0009)
3. API validates type and size, then links the object to the owning record

No file bytes pass through the API process. Accepted types: `application/pdf`, `image/jpeg`; default size cap 10 MB.

# Webhooks (Outbound, Future)

Outbound webhooks for future partners (practice management systems) are deferred. When introduced they will follow the same signature, retry, and idempotency contract as inbound webhooks.

# OpenAPI

Machine-readable specifications are generated from the NestJS controllers with Swagger and exported to `openapi/`. Generated files are committed and verified in CI against this document; a drift check fails the build.

# Graceful Degradation Contract

When an external dependency fails, endpoints degrade rather than erroring into a blank screen: catalog and articles serve cached responses, checkout returns `503 DEPENDENCY_UNAVAILABLE` with a plain-language fallback instruction and the clinic phone number, and intake falls back to email-only submission. This satisfies the Reliability requirement in `non-functional-requirements.md`.

---

# Acceptance Criteria

- Every resource in `domain-model.md` has documented endpoints or is explicitly internal
- Every endpoint specifies method, path, auth, and error codes
- Every money-moving endpoint specifies idempotency behaviour
- OpenAPI generation is enforced by CI against this document
- The emergency path works with no session and no third-party dependency

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `domain-model.md` | Resources these endpoints expose |
| `domain-events.md` | Facts raised when mutations succeed |
| `bounded-context.md` | Context boundaries behind each route group |
| `architectural-decision-record.md` | REST-first decision (ADR-0006) and tokenisation (ADR-0007) |
| `engineering-guidelines.md` | Implementation standards for these contracts |

# Guiding Principle

> **The API is a promise made to code you have not written yet. Be boring, be explicit, and never break a promise without a new version.**
