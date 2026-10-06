# OpenAPI Specifications

> **ruby-veterinary Documentation**
>
> **Document:** OpenAPI Specifications
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** API Contract

---

# Purpose

Machine-readable OpenAPI 3.1 definitions for the ruby-veterinary REST API, mirroring `../api-specification.md`. These files are the committed contract surface: generated or verified in CI against the NestJS controllers (Swagger), with drift failing the build.

---

# Specifications

| File | Domain | Description |
|------|--------|-------------|
| `components.yaml` | Shared | Envelope, errors, pagination, security schemes, common parameters |
| `auth.yaml` | Auth | Register, login, refresh, logout, current identity |
| `clinic-content.yaml` | Clinic Content | Hours, contact, services, staff, documents |
| `intake.yaml` | Care Coordination | Appointment requests, client intake, signed uploads |
| `publishing.yaml` | Publishing | Articles, taxonomy, newsletter subscribe |
| `commerce.yaml` | Commerce | Products, cart, checkout, orders, subscriptions |
| `pharmacy.yaml` | Pharmacy Authorisation | Review queue and decision endpoints |
| `admin-operations.yaml` | Operations | Admin alerts, shared inbox |
| `webhooks.yaml` | Webhooks | Inbound WhatsApp and payment webhooks |

Each domain file is self-contained (local `components`) so generators and reviewers can open one file in isolation. `components.yaml` is the canonical reference for shared shapes.

---

# API Conventions

| Convention | Value |
|------------|-------|
| Base URL (production) | `https://api.ruby-veterinary.example.com/api/v1` *(placeholder until the domain is selected — PRD §14 open question)* |
| Base URL (local) | `http://localhost:3000/api/v1` |
| Auth | `Authorization: Bearer <access token>` |
| Success envelope | `{ "data": ..., "meta": { "requestId": "...", "timestamp": "..." } }` |
| Error envelope | `{ "error": { "code": "...", "message": "...", "details": [...], "requestId": "..." } }` |
| Pagination | `meta.pagination` with `limit`, `cursor`, `hasMore` |
| Idempotency | `Idempotency-Key` header on money-moving / record-creating POSTs |
| Webhooks | Provider signature headers, not bearer tokens |

Error `code` values are stable API surface: clients branch on `code`, never on `message`.

---

# Usage

```bash
# Validate
npx @redocly/cli lint openapi/*.yaml

# Swagger UI locally
npx @redocly/cli preview-docs openapi/commerce.yaml

# Generate a TypeScript client
npx openapi-generator-cli generate -i openapi/commerce.yaml -g typescript-fetch -o ./sdk/commerce
```

---

# Drift Policy

1. Controllers in `ruby-veterinary-api` emit Swagger during build
2. CI exports the spec and compares with these files
3. Drift fails the build; the fix is regenerate-and-commit or change `api-specification.md` in the same PR

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../api-specification.md` | Human-readable source of truth for the same contract |
| `../engineering-guidelines.md` | CI drift gate |
| `../architectural-decision-record.md` | ADR-0006 REST-first |
| `../bounded-context.md` | Context behind each route group |

---

# Acceptance Criteria

- Every endpoint group in `api-specification.md` has a corresponding YAML file
- Shared envelope and error codes match `components.yaml` and `api-specification.md`
- All files parse as OpenAPI 3.1
- CI fails on drift between NestJS export and these files

---

# Guiding Principle

> **The contract is code you can diff. If the YAML and the controllers disagree, one of them is lying — and CI has to decide which.**
