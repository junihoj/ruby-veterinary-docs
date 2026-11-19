# Product Creation Coverage

> **ruby-veterinary Documentation**
>
> **Document:** Product Creation Coverage — `ruby-veterinary-api`
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Commerce Bounded Context
>
> **Classification:** Implementation Coverage

---

# Purpose

Records what "product creation" currently means in `ruby-veterinary-api`:
which route exists, what it persists, what it validates, what is tested, and
which parts of a sellable product are still out of scope.

This document describes the code as it exists today. The binding contract
remains `openapi/commerce.yaml` and `data-model/commerce.md`; where this
document disagrees, the contract wins.

---

# Verdict

Product creation **is covered**, but only for the **product shell**.

`POST /api/v1/admin/products` creates a draft product row. It does **not**
create variants, SKUs, prices, stock, images, composite slots, grouped
membership, or facets. A newly created product therefore cannot be sold until
those records are added by another path.

The current handler states this explicitly:

> Creates a draft (published defaults to false) — variant seeding for the new
> product is out-of-band until an admin variants surface exists.

---

# Endpoint

| Item | Value |
|---|---|
| Method | `POST` |
| Path | `/api/v1/admin/products` |
| Operation ID | `adminCreateProduct` |
| Controller | `src/modules/commerce/presentation/admin-products.controller.ts` |
| Handler | `src/modules/commerce/application/handlers/admin-create-product.handler.ts` |
| Repository write | `src/modules/commerce/infrastructure/repositories/product.repository.ts` (`createProduct`) |
| Permission | `catalog:write` |
| Current holder | `practice_manager` (`src/modules/identity/domain/role-permissions.ts`) |
| Success status | `201 Created` |
| Contract | `openapi/commerce.yaml` → `/admin/products` |

The route is staff-only. A non-staff caller receives `403 STAFF_ONLY`.

---

# What the Create Call Persists

The request body is `AdminCreateProductDto`
(`src/modules/commerce/application/dto/commerce.requests.dto.ts`).

| Field | Required | Notes |
|---|---|---|
| `name` | Yes | Max 200 characters |
| `categoryId` | Yes | Must resolve to an existing `commerce.product_categories` row |
| `productType` | Yes | `simple`, `variable`, `grouped`, `external`, or `composite` |
| `slug` | No | Lowercase kebab-case; generated from `name` when omitted |
| `summary` | No | Defaults to an empty string |
| `descriptionMd` | No | Max 2,000 characters |
| `isVirtual` | No | Defaults to `false` |
| `isDownloadable` | No | Defaults to `false` |
| `pricingMode` | No | `fixed` or `sum`; composite products only |
| `externalUrl` | No | Absolute URL; external products only |
| `buttonText` | No | Max 120 characters; external products only |
| `requiresPrescription` | No | Defaults to `false` |
| `requiresPetClaim` | No | Defaults to `false` |
| `published` | No | Defaults to `false` |
| `featured` | No | Defaults to `false` |

The response is a `ProductDetailDto`. For a newly created product,
`variants` is empty.

---

# Validation and Invariants

The handler performs these checks before writing:

1. **Category parsing** — `categoryId` is parsed with `PrefixedId.tryAnyToUuid`
   before touching Prisma so a malformed id returns `400`, not a database
   error.
2. **Category existence** — unknown categories return `400 VALIDATION_FAILED`.
3. **Slug uniqueness** — a generated or supplied slug already in use returns
   `400 VALIDATION_FAILED`.
4. **Shape invariants** (`src/modules/commerce/application/product-shape.ts`):
   - `external` requires `externalUrl`.
   - `externalUrl` is rejected for every other product type.
   - `composite` requires `pricingMode`.
   - `pricingMode` is rejected for every other product type.

The global validation pipe whitelists request bodies strictly, so unknown
fields return `400 VALIDATION_FAILED`.

---

# Test Coverage

Covered by `test/commerce.e2e-spec.ts` under `admin products (catalog:write)`:

| Behaviour | Asserted |
|---|---|
| Non-staff create | `403` |
| Non-staff update | `403` |
| Draft create | `201`, `published: false`, prefixed `prd_` id, generated slug |
| Empty variants on create | `variants: []` |
| Duplicate slug | `400 VALIDATION_FAILED` |
| Unknown category | `400 VALIDATION_FAILED` |
| Partial product update | `200`, fields merged, publication unchanged |
| Unknown product update | `404 RESOURCE_NOT_FOUND` |
| Variant stock update | `200`, stock/available quantity updated |
| Variant/product mismatch | `404 RESOURCE_NOT_FOUND` |

The route is also part of the OpenAPI contract diff gate.

---

# Not Covered Yet

The following are **not** part of product creation today:

| Area | Status |
|---|---|
| Variant creation (SKU, price, currency, stock, weight, default flag) | No endpoint or create-time seeding |
| Product images / media | No product media column or table |
| Composite slot and option setup | Not created with the parent product |
| Grouped product membership | Not created with the parent product |
| Facets / filters | Not created with the parent product |
| Category creation | No admin categories endpoint; categories must already exist |
| Publish workflow | Only the `published` boolean; no review/approval transition |
| Admin UI | API only |

The only variant write surface is
`PATCH /api/v1/admin/products/{id}` with a `variantId`, which **updates an
existing variant** (stock/price maintenance). It does not create one.

Downloadable products follow a separate path:
`POST /api/v1/admin/products/{id}/download-files` creates a file record and a
presigned upload URL.

---

# Practical Implication

A successful `POST /admin/products` produces a valid catalog record that can be
read by staff and, once `published`, by public catalog reads. It has no variant,
so it contributes no SKU or price and cannot be added to a cart.

A complete "create a sellable product" flow requires either:

1. A future admin variant-creation surface, or
2. A controlled seed/import that creates variants out-of-band.

Until then, product creation should be treated as **draft metadata creation**,
not catalog onboarding end to end.

---

# Related Documents

| Document | Relationship |
|---|---|
| `openapi/commerce.yaml` | Binding create/update contract |
| `data-model/commerce.md` | `products`, `product_variants`, and related tables |
| `implementations/api-plan.md` | Milestone sequencing (M3a commerce core, M3b composite/digital) |
| `api-specification.md` | Error envelope and admin route conventions |
