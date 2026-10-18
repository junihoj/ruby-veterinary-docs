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
| `401` | `TOKEN_EXPIRED`, `INVALID_CREDENTIALS`, `INVALID_SIGNATURE` | Missing or invalid authentication |
| `403` | `STAFF_ONLY`, `PRESCRIBER_REQUIRED`, `DOWNLOAD_NOT_AVAILABLE` | Authenticated but not permitted for this action |
| `404` | `RESOURCE_NOT_FOUND` | Absent or hidden from this caller |
| `409` | `IDEMPOTENCY_CONFLICT`, `STOCK_CHANGED`, `EMAIL_TAKEN` | State conflict; safe to re-read and retry |
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

Auth failure semantics (stable `code` surface):

- `401 TOKEN_EXPIRED` — access token past its TTL, or refresh session past `expires_at`.
- `401 INVALID_CREDENTIALS` — wrong password, unknown email, revoked or reused refresh token.
- `409 EMAIL_TAKEN` — register with an email that already has an account.
- Refresh-token reuse (a revoked/rotated token presented again) is treated as compromise: **all of that user's sessions are revoked** and the request fails `401 INVALID_CREDENTIALS`.

Cache policy: public reads (clinic content, later catalog/articles) answer `Cache-Control: public, max-age=60`; all auth responses answer `Cache-Control: no-store`.

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
| `POST` | `/api/v1/uploads/sign` | Bearer | Issue signed upload URL for PDF/JPEG history (`medical_history` records an `upl_…` row; `cms_media` presigns only, reserved for M7) |
| `GET` | `/api/v1/clients/intake/:id/status` | Bearer | Submission status |
| `GET` | `/api/v1/admin/appointment-requests` | Staff (`care:write`) | Triage queue, status filter |
| `PATCH` | `/api/v1/admin/appointment-requests/:id` | Staff (`care:write`) | Triage: `received → triaged → scheduled \| declined` (`409 STATE_CONFLICT` on invalid transitions) |
| `GET` | `/api/v1/admin/clients/intake` | Staff (`care:write`) | Intake queue |
| `PATCH` | `/api/v1/admin/clients/intake/:id` | Staff (`care:write`) | Move to `in_review` or `rejected` |
| `POST` | `/api/v1/admin/clients/intake/:id/accept` | Staff (`care:write`) | File the Client + Pets through the Clients context module (Flow 2); `422 INTAKE_NOT_ACCEPTABLE` when `existingClient` lacks `clientId` |
| `GET` | `/api/v1/admin/uploads` | Staff (`care:write`) | Upload review queue |
| `GET` | `/api/v1/admin/uploads/:id/url` | Staff (`care:write`) | Signed download URL (ADR-0016) |
| `PATCH` | `/api/v1/admin/uploads/:id` | Staff (`care:write`) | Finalise (object must exist — `422 UPLOAD_MISSING`) or reject with reason |

`POST` bodies are idempotency-safe: retries with an `Idempotency-Key` header return the original result.

Form submissions route through `EmailPort` (console adapter until an SMTP provider is chosen, ADR-0013): each request records dashboard + email alert rows and attempts delivery — an alert failure never fails the owner's submission. Submission publishes `AppointmentRequested` / `IntakeSubmitted`; acceptance publishes `IntakeComplete`; upload finalisation publishes `HistoryUploadReceived`.

## Client & Patient Records API

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `GET` | `/api/v1/pets` | Bearer | The caller's pets (pet-claim surface for commerce prefill; empty list until an intake is accepted) |
| `GET`/`POST` | `/api/v1/admin/clients` | Staff (`care:write`) | Client list (`q` over name/phone) and create (`409 ALREADY_LINKED` on a second active user link) |
| `GET`/`PATCH` | `/api/v1/admin/clients/:id` | Staff (`care:write`) | Client record (GET includes pets); soft-delete via status |
| `GET`/`POST` | `/api/v1/admin/clients/:id/pets` | Staff (`care:write`) | List and add pets |
| `GET`/`PATCH` | `/api/v1/admin/pets/:id` | Staff (`care:write`) | Pet record; `status: deceased` keeps history queryable |
| `GET`/`POST` | `/api/v1/admin/pets/:id/medical-history` | Staff (`care:write`) | History records (reference objects, ADR-0016); POST creates the row and returns a presigned PUT |
| `GET`/`POST` | `/api/v1/admin/pets/:id/primary-vet` | Staff (`care:write`) | Primary-vet link history; POST starts a new current link and closes the previous one in the same transaction (`422 NOT_A_VETERINARIAN` unless the target holds the veterinarian role) |

The one-current-primary-vet-per-pet partial unique index is the pharmacy guardrail (M5) — link changes must preserve it. `care:write` is granted to every staff role; client identities are 403 `STAFF_ONLY` on all `/admin` routes.

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
| `GET` | `/api/v1/products/:slug` | Public | Product detail; payload varies by type: variants (simple/variable), members (grouped), slots + options (composite), external URL/button (external), virtual/downloadable flags |
| `GET`/`POST`/`PATCH`/`DELETE` | `/api/v1/cart`, `/api/v1/cart/items` | Bearer | Persistent cart; composite lines carry a `components` selection |
| `POST` | `/api/v1/checkout/quote` | Bearer | Totals: tax, shipping, pickup; composite lines re-priced from selected options |
| `POST` | `/api/v1/checkout` | Bearer | Place order; returns `PRESCRIPTION_PENDING` when Rx items present |
| `GET` | `/api/v1/orders` | Bearer | Order history with fulfilment state |
| `GET` | `/api/v1/orders/:id` | Bearer | Single order incl. Rx status and download grants |
| `GET` | `/api/v1/orders/:id/downloads` | Bearer | Downloadable files granted on the order, with remaining counts and expiry |
| `POST` | `/api/v1/downloads/:grantId/url` | Bearer | Issue a short-lived signed URL for one granted file (ADR-0009) |
| `POST`/`PATCH`/`DELETE` | `/api/v1/subscriptions` | Bearer | Auto-refill management (physical simple/variable products only) |
| `POST` | `/api/v1/admin/products` etc. | Staff (`catalog:write`) | Catalog maintenance across all seven product types |
| `POST` | `/api/v1/admin/products/:id/download-files` | Staff (`catalog:write`) | Create a downloadable file record and receive a presigned PUT upload URL |

Checkout accepts an `Idempotency-Key`; stock is reserved per variant or composite component (`STOCK_CHANGED` on conflict). `fulfilmentMethod` is `ship`, `pickup`, or `digital` (digital when every line is virtual/downloadable). Quote totals are deterministic: `taxTotal` is 20% of `subtotal`, `shippingTotal` is a flat £4.99 for `ship` and `0` for `pickup`/`digital`, and `grandTotal = subtotal + taxTotal + shippingTotal`. Prescription-only lines are rejected at add-to-cart (`422 PRESCRIPTION_PENDING`) until the prescription flow releases them.

Composite lines carry a `components` selection validated against the product's slots (unknown slot/option or a per-slot count outside `minSelect`/`maxSelect` fails `400`); prices are re-read at quote — `fixed` from the parent's default variant, `sum` as the total of selected options with `price_override` substituting the candidate price. Derived flags apply to composite lines: prescription if any selected option product requires one (`422` at add), digital eligibility only when every selected component is virtual/downloadable.

On payment capture the order is paid, component/variant stock settles, a `product_download_grants` row is issued for each paid line whose product is downloadable (idempotent per line), and a fulfilment whose lines are all virtual/downloadable completes immediately. `GET /orders/:id/downloads` lists the grants; `POST /downloads/:grantId/url` answers `404` for unknown or foreign grants and `403 DOWNLOAD_NOT_AVAILABLE` when the grant is revoked, expired, or exhausted.

Subscriptions are physical `simple`/`variable` only (anything else `422`); an in-process timer picks `active` rows whose `next_order_at` is due, places a system order through the same idempotent checkout path (`Idempotency-Key` = `sub:<id>:<due-date>`), auto-captures it via the shared capture path, and advances `next_order_at`; a subscription that cannot reserve stock is paused instead of ordered.

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

Signature: the `X-Provider-Signature` header carries the hex HMAC-SHA256 of the raw request body under the shared provider secret; verification happens before parsing, and a mismatch answers `401 INVALID_SIGNATURE`.

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
2. Client uploads directly to object storage (ADR-0016)
3. A `medical_history` sign records a pending upload row, linked to the submission at intake time and finalised (object existence verified) or rejected by staff

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
