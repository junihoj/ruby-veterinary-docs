# Architecture Rules

> **ruby-veterinary Strategic Architecture**
>
> **Document:** 06 — Architecture Rules
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary

---

# Purpose

The non-negotiable constraints that preserve context isolation, clinical safety, and operability of the modular monolith. Violations of MUST rules are blocking review failures.

---

# Rule Classification

| Classification | Meaning | Enforcement |
|----------------|---------|-------------|
| **MUST** | Mandatory; violation blocks merge | Code review, CI checks |
| **SHOULD** | Strongly recommended; exceptions need a documented reason | Code review |
| **MAY** | Team discretion | Advisory |

---

## Rule 1 — Bounded Context Isolation

**MUST**

| # | Rule |
|---|------|
| 1.1 | No context reads or writes another context's database schema |
| 1.2 | Cross-context references are by opaque id, never cross-schema foreign keys |
| 1.3 | Cross-context imports of domain entities are prohibited; use module interfaces |
| 1.4 | Each context owns its schema namespace and migrations |
| 1.5 | New contexts require updates to `../bounded-context.md`, `../domain-model.md`, and `04-ownership-matrix.md` in the same change |

---

## Rule 2 — Event-Driven Collaboration

**MUST**

| # | Rule |
|---|------|
| 2.1 | Cross-context notifications use domain events, not shared tables |
| 2.2 | Event names are past-tense facts; schemas evolve additively |
| 2.3 | Event handlers are idempotent; duplicate delivery is assumed |
| 2.4 | Publishers never invoke consumer business logic synchronously to "make sure" it ran |

---

## Rule 3 — Clinical Safety Rails

**MUST**

| # | Rule |
|---|------|
| 3.1 | Prescription decisions are made only in the Pharmacy context, only by veterinarian-role actors |
| 3.2 | Rx order state transitions require Pharmacy events; UI cannot bypass |
| 3.3 | Decision records are append-only; UPDATE/DELETE are prohibited at the database level |
| 3.4 | Every decision and privileged prescription read writes an audit row |
| 3.5 | The emergency surface never depends on bot, payment, or CMS availability |

---

## Rule 4 — Commerce Integrity

**MUST**

| # | Rule |
|---|------|
| 4.1 | Card data never touches our systems; tokenised gateway only (ADR-0007) |
| 4.2 | Money-moving POSTs accept `Idempotency-Key`; replays return the original response |
| 4.3 | Prices are revalidated server-side at quote and checkout |
| 4.4 | Stock is reserved per variant; conflicts return `STOCK_CHANGED`, never silent oversell |

---

## Rule 5 — API Contract Discipline

**MUST**

| # | Rule |
|---|------|
| 5.1 | All routes live under `/api/v1`; breaking changes require a new version prefix |
| 5.2 | Response and error envelopes match `../openapi/components.yaml` |
| 5.3 | Clients branch on error `code`, never on `message` |
| 5.4 | OpenAPI files in `../openapi/` are CI-verified against NestJS Swagger exports |
| 5.5 | Emergency and degradation responses include plain-language fallback plus clinic phone |

---

## Rule 6 — Security and Privacy

**MUST**

| # | Rule |
|---|------|
| 6.1 | Secrets live in the VPS `.env` and CI deploy secrets — never in the repository or logs |
| 6.2 | Passwords hashed with Argon2id; tokens rotated per ADR-0011 |
| 6.3 | Object storage buckets are private; access is by short-lived signed URL |
| 6.4 | Staff endpoints enforce additive role claims; public endpoints never trust client-supplied roles |
| 6.5 | Audit logs redact secrets and never store PANs or password hashes |

---

## Rule 7 — Performance and Availability

**MUST / SHOULD**

| # | Rule | Class |
|---|------|-------|
| 7.1 | Public pages fit the sub-3s budget on 4G; measure in CI where possible | MUST |
| 7.2 | Public reads are cacheable at Cloudflare; cache keys exclude private data | MUST |
| 7.3 | Bot keyword handling completes within 2s | MUST |
| 7.4 | New background work (queues, replicas) requires an ADR before implementation | SHOULD |

---

## Rule 8 — Deployment and Evolution

**MUST / SHOULD**

| # | Rule | Class |
|---|------|-------|
| 8.1 | Everything runs on the single VPS stack in `../deployment-architecture.md` | MUST |
| 8.2 | Deploys are CI-driven over SSH; manual production DDL is prohibited | MUST |
| 8.3 | Migrations ship through `../database/migrations.md` zero-downtime rules | MUST |
| 8.4 | Extracting a context into a service requires a new ADR and re-evaluation of sync pairs in `03-context-relationships.md` | SHOULD |

---

## Rule 9 — Documentation as Contract

**MUST**

| # | Rule |
|---|------|
| 9.1 | Behaviour changes update the relevant spec in the same pull request |
| 9.2 | ADRs record decisions; accepted ADRs are never re-argued informally |
| 9.3 | Data model files update alongside migrations that implement them |
| 9.4 | Design tokens and UI patterns in `../ui-ux/` are the source of truth for frontend theming |

---

# Acceptance Criteria

- Every rule traces to an ADR, NFR, or domain safety requirement
- Review checklist in `../engineering-guidelines.md` references these rules
- New MUST rules are impossible to satisfy by accident — each maps to an automated or review gate

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../bounded-context.md` | Rules 1 source |
| `../architectural-decision-record.md` | ADRs behind MUSTs |
| `../api-architecture.md` | Code-level enforcement of these rules |
| `../engineering-guidelines.md` | Day-to-day review gates |
| `../non-functional-requirements.md` | Performance/security targets |
| `../ui-ux/accessibility.md` | Accessibility MUSTs |

---

# Guiding Principle

> **Rules exist so that 2am decisions are already made. Follow them by default; break them only with a written exception and a date to revisit.**
