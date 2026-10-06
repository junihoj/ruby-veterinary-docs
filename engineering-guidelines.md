# engineering-guidelines.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Engineering Guidelines
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Engineering Standard

---

# Purpose

This document defines how software is written, reviewed, tested, and released across the ruby-veterinary repositories.

It turns the architectural decisions in `architectural-decision-record.md` into daily practice. Where this document and the ADRs disagree, the ADRs win and this document is updated.

---

# Engineering Philosophy

- Small, reversible changes over large speculative ones
- The checklist in `functional-requirements.md` is the definition of "done", not the ticket text
- Safety rails (prescription authorisation, payment integrity, emergency visibility) are never traded for speed
- Documentation is part of the change, not a follow-up

# Repository Structure

Three sibling repositories under the `ruby-veterinary-service` parent, tracked as git submodules:

| Repository | Contents | Toolchain |
|------------|----------|-----------|
| `ruby-veterinary-web-frontend` | Public site, storefront, staff back office | Next.js 16, React 19, Tailwind 4, TypeScript, pnpm |
| `ruby-veterinary-api` | REST API and third-party integrations | NestJS 11, TypeScript, Jest 30, ESLint 9, Prettier |
| `ruby-veterinary-docs` | This specification set | Markdown, UTF-8 |

Parent repository pins all three as submodules. A change spanning repositories is merged in dependency order: docs first, then API, then front end.

# Development Lifecycle

1. Read the relevant specification section before writing code
2. Implement behind a feature flag or additive endpoint where risk warrants
3. Tests pass locally with `pnpm lint`, `pnpm test` (API) or `pnpm lint` (front end)
4. Pull request referencing the requirement or ADR it satisfies
5. Review against the checklist below; merge with squash
6. Deploy through CI to the managed platform

# Branching Strategy

| Branch | Rule |
|--------|------|
| `main` | Always deployable; protected, no direct pushes |
| `feat/<ticket>-<slug>` | Branch from `main`, short-lived |
| `fix/<slug>` | Branch from `main` |
| `hotfix/<slug>` | Cut from `main` for production incidents, merged back immediately |

One logical change per branch. No long-running feature branches.

# Commit Convention

Conventional Commits:

```
feat(cart): reserve stock per variant at checkout
fix(rx): block release until prescriber decision recorded
docs(api): document idempotency keys for checkout
chore(deps): bump next to 16.3.8
```

Types: `feat`, `fix`, `docs`, `test`, `chore`, `refactor`, `perf`, `ci`. Scope is the module or repository area. The body explains why, not what.

# Pull Request Standards

- Title states the behaviour change in plain language
- Description links the requirement checklist item or ADR
- Screenshots for any visual change; recording for flows
- No unrelated formatting churn in the diff
- Checklist:

  - [ ] Spec section read and reflected
  - [ ] Tests added or updated for changed behaviour
  - [ ] No secrets, keys, or card data in code or logs
  - [ ] Error paths handled with stable `code` values (see `api-specification.md`)
  - [ ] Accessibility unaffected (keyboard path, alt text, contrast)
  - [ ] Docs updated when behaviour or contract changed

# Definition of Ready (DoR)

A task is ready when acceptance is testable against `functional-requirements.md`, dependencies (gateway, WhatsApp approval, design) are identified, and out-of-scope items are named.

# Definition of Done (DoD)

- Acceptance criteria in the checklist are demonstrably met
- Automated tests cover the changed behaviour
- Lint and type check pass in CI
- No regression in the sub-3s or 99.9% commitments
- Documentation updated in the same change
- Rolled out with a way to revert

---

# Code Organization

**API (NestJS)** - one module per bounded context, matching `bounded-context.md`:

```
src/
├── modules/
│   ├── commerce/            # controller, service, repository, entities, dto
│   ├── pharmacy/
│   ├── intake/
│   └── ...
├── shared/                  # cross-cutting: auth guard, error codes, config
└── main.ts
```

Rules: controllers handle HTTP only, services hold business rules, repositories are the sole importers of TypeORM (ADR-0004), DTOs validate at the boundary.

**Front end (Next.js)** - App Router with route groups:

```
src/
├── app/
│   ├── (public)/            # services, staff, articles - statically rendered
│   ├── (store)/             # catalog, cart, checkout - dynamic
│   ├── (account)/           # intake, orders - authenticated
│   └── (admin)/             # back office - staff roles only
├── components/              # shared UI primitives
└── lib/                     # api client, types, utilities
```

# Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files | kebab-case | `prescription-queue.component.tsx` |
| Classes | PascalCase | `PrescriptionService` |
| Functions/vars | camelCase | `buildFulfilmentQuote` |
| DTO / entity fields | camelCase in code, snake_case accepted at edge | |
| Database columns | snake_case | `prescriber_id` |
| Tables | plural snake_case | `prescription_requests` |
| API routes | plural kebab | `/api/v1/appointment-requests` |
| Env vars | SCREAMING_SNAKE | `STRIPE_SECRET_KEY` |
| Domain events | past-tense PascalCase | `PrescriptionApproved` |

# Backend Guidelines (NestJS)

- Module boundaries mirror bounded contexts; cross-module imports go through the public service interface only
- Validate every input with class-validator DTOs; reject unknown fields on writes
- Authorisation via guards reading role claims; never trust a field supplied by the client to assert identity (pet and prescriber associations are looked up, not accepted)
- Transactions wrap any multi-write operation touching commerce or pharmacy state
- Configuration through `@nestjs/config` with validated schema; no hardcoded URLs or keys
- Audit writes are part of the same transaction as the decision they describe

# Frontend Guidelines (Next.js)

- Server components by default; client components only where interaction requires it
- Public pages statically rendered and cached; never block rendering on a third-party call
- No secrets in client code; all privileged calls go through the API with a bearer token
- Every interactive control reachable by keyboard, with visible focus
- Images pass through the Next.js image pipeline for automatic WebP conversion (NFR media optimisation)
- Emergency contact block is a shared layout component rendered on every page - never duplicated per page

# API Development

Follow `api-specification.md` exactly: envelope, error `code` values, idempotency keys, pagination. New endpoints require the OpenAPI spec update in the same PR.

# Error Handling

- Return stable machine-readable `code` values; never leak stack traces or SQL to clients
- Log server-side with `requestId` correlation
- Expected failures (validation, conflicts) are logged at info; unexpected at error with alerting
- Front end renders plain-language fallbacks with the clinic phone number for `5xx` responses (graceful degradation)

# Testing Strategy

| Level | Tool | Expectation |
|-------|------|-------------|
| Unit | Jest | Business rules in services; prescription guardrails fully covered |
| Integration | Jest + Testcontainers-style DB | Repository and transaction behaviour |
| End-to-end | Jest + supertest (API), Playwright (front end) | Happy path plus one failure path per user journey |
| Contract | OpenAPI diff in CI | Spec drift fails the build |

Minimum gate: any change to pharmacy, payments, or emergency-path code requires tests for its rejection paths, not only its success path.

# Security Practices

- HTTPS everywhere; HSTS enabled (NFR: Data Encryption)
- Card data never accepted, stored, or logged (ADR-0007)
- Uploads restricted by type and size, stored in private object storage, served via short-lived signed URLs (ADR-0009)
- Webhook signatures verified before any parsing
- Secrets in the platform secret store only; never in the repository or `.env` committed files
- Rate limiting on auth and submission endpoints
- Dependency audit in CI; cookie consent and privacy policy pages present before any tracking (NFR: Privacy Compliance)

# Logging & Observability

Structured JSON logs with `requestId`, user id when authenticated, and domain event names. Alerting on: webhook failures, payment failures spike, intake submission failures, uptime and latency budget breaches. The admin alert dashboard consumes the same event stream.

# Performance Guidelines

- Public pages must fit the sub-3s-on-4G budget; measure, do not assume
- Cache public reads aggressively at the CDN edge
- Optimistic UI only where the server confirms quickly; never optimistically for payment or prescription state
- Catalog queries paginated and indexed to hold the hundreds-of-SKUs scalability target

# Database Standards

See `database/` for architecture, migrations, performance, and recovery. In practice: migrations are versioned and reviewed, never edited after merge; every table carries `created_at`/`updated_at`; soft delete only where the spec requires history (prescription decisions are append-only, ADR-0012).

# CI/CD Pipeline

| Stage | Gate |
|-------|------|
| Lint & type check | ESLint + `tsc --noEmit` |
| Unit & integration tests | Jest |
| OpenAPI drift | Generated spec matches `openapi/` |
| Build | `next build` / `nest build` |
| Deploy | Trunk-based to the managed platform; rollback is one command |
| Post-deploy smoke | Health check, homepage fetch, form submission probe |

# Documentation Standards

Specifications live in `ruby-veterinary-docs` and follow its README conventions: header block, `# Purpose`, cross-references, semantic versioning in headers. Behaviour changes update the relevant spec in the same pull request; contract changes update `api-specification.md` and `openapi/` together.

# Code Review Checklist

- Correctness against the linked checklist item
- Guardrails intact: no path releases an Rx order or captures payment without its required steps
- Boundary validation present on every new endpoint
- Tests for failure paths, not just success
- Performance implications for public reads
- No unrelated changes

# Technical Debt

Debt is recorded as an issue with a cost statement ("what does this cost us each month?") and an owner. Debt touching security, payments, or prescription handling is repaid before feature work.

# Release Management

Trunk-based continuous delivery. Changes behind flags are released dark and activated deliberately. Any release affecting payments or prescription flow is staged: sandbox first, then a manual end-to-end verification with a real veterinarian and a test order.

# Engineering Culture

- Ask which spec section governs this before arguing preference
- Blameless post-incident review for any user-facing outage
- Small pull requests, kind reviews, fast feedback

---

# Acceptance Criteria

- Every repository lints, type-checks, and tests green in CI
- New endpoints appear in generated OpenAPI in the same PR
- Guardrail changes ship with rejection-path tests
- The DoD checklist is verifiable on every merged pull request

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `architectural-decision-record.md` | Decisions these guidelines operationalise |
| `api-specification.md` | Contract conventions enforced in review |
| `bounded-context.md` | Module boundaries mirrored in code organization |
| `database/` | Persistence standards referenced here |
| `functional-requirements.md` | Definition of done |

# Guiding Principle

> **Write the code as if the next person to read it is the one who will be paged at 2am. Boring conventions, explicit guardrails, and a green test suite are how we let them sleep.**
