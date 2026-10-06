# Commerce Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Commerce Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Commerce Bounded Context
>
> **Classification:** Core Domain
>
> **Schema:** `commerce`

---

# Purpose

Defines the tables behind the storefront and everything between browsing and fulfilment: product catalog with three product types, categories, search facets, carts, order lifecycle, fulfilment choice, payment references, and subscriptions/auto-refill. Money moves through a tokenised gateway; this schema stores references and state, never card data.

---

# Responsibilities

The Commerce context owns:

- Product catalog (single, variable, bundle) and variants
- Product categories and catalog facets (pet type, life stage, condition)
- Carts and cart items
- Orders, order items, fulfilment, payment references
- Subscriptions and auto-refill schedules
- Stock counts per variant

It does **not** own prescription decisions (Pharmacy emits events back), pet records, or client profiles. Prescription-only items reference `pet_id` at claim time and flow through Pharmacy's hold.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Product | `products` | Variants and bundle components belong to it; stock is per-variant |
| Cart | `carts` | Items move with the cart; cart is user-scoped |
| Order | `orders` | Lines, fulfilment, payment reference; strongly consistent |

---

# Entities

## product_categories

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | e.g. `Prescription Medication`, `Therapeutic Diets`, `General Pet Supplies` |
| slug | text | no | Unique |
| description | text | yes | |
| sort_order | integer | no | |
| created_at | timestamptz | no | |

**Indexes**

- `uq_product_categories_slug` unique on `(slug)`

## products

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `prd_…` |
| category_id | uuid | no | FK → `product_categories.id` |
| name | text | no | |
| slug | text | no | Unique |
| summary | text | no | |
| description_md | text | yes | |
| product_type | text | no | `single` \| `variable` \| `bundle` |
| requires_prescription | boolean | no | True for Rx items; pharmacy hold engages at checkout |
| requires_pet_claim | boolean | no | Owner must associate a pet (links to clients.pets by id) |
| published | boolean | no | |
| featured | boolean | no | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_products_slug` unique on `(slug)`
- `ix_products_catalog` on `(published, category_id, featured)`

**Invariants**

- `variable` products have ≥ 2 variants; `single` products have exactly 1 implied/default variant row for stock/SKU uniformity
- `bundle` products reference components; bundles never stock independently of components unless policy says otherwise (default: component stock governs)

## product_variants

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `var_…` |
| product_id | uuid | no | FK → `products.id` |
| name | text | no | e.g. `0–5 kg`, `12 pack` |
| sku | text | no | Unique |
| price | numeric(10,2) | no | |
| compare_at_price | numeric(10,2) | yes | Strikethrough display |
| currency | char(3) | no | ISO 4217 |
| stock_quantity | integer | no | Default 0 |
| stock_reserved | integer | no | Default 0; held during checkout |
| weight_grams | integer | yes | Shipping calc |
| is_default | boolean | no | Default selection on PDP |
| active | boolean | no | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_product_variants_sku` unique on `(sku)`
- `ix_product_variants_product` on `(product_id, active)`

**Invariants**

- Available stock = `stock_quantity - stock_reserved`; never negative
- `STOCK_CHANGED` at checkout when available stock dropped since quote

## bundle_components

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| bundle_product_id | uuid | no | FK → `products.id` (bundle) |
| component_product_id | uuid | no | FK → `products.id` (single/variable) |
| component_variant_id | uuid | yes | FK → `product_variants.id` when fixed variant |
| quantity | integer | no | ≥ 1 |
| sort_order | integer | no | |

**Invariants**

- A bundle product must not appear as its own component
- Pricing: bundle price is explicit on the bundle's variant, not summed from components

## catalog_facets

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| facet_type | text | no | `pet_type`, `life_stage`, `health_condition` |
| value | text | no | e.g. `dog`, `puppy`, `joint_support` |
| slug | text | no | Unique per type |
| sort_order | integer | no | |

## product_facets

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| product_id | uuid | no | FK → `products.id` |
| facet_id | uuid | no | FK → `catalog_facets.id` |

**Indexes**

- PK `(product_id, facet_id)`

## carts

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `crt_…` |
| user_id | uuid | no | Cross-schema ref by id → `identity.users.id` |
| status | text | no | `open` \| `converted` \| `abandoned` |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_carts_open_user` unique on `(user_id)` where `status = 'open'`

## cart_items

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| cart_id | uuid | no | FK → `carts.id` |
| product_id | uuid | no | FK → `products.id` |
| variant_id | uuid | no | FK → `product_variants.id` |
| quantity | integer | no | ≥ 1 |
| pet_id | uuid | yes | Cross-schema ref by id → `clients.pets.id` for Rx items |
| added_at | timestamptz | no | |

**Indexes**

- `uq_cart_items_line` unique on `(cart_id, variant_id, pet_id)`
- `ix_cart_items_cart` on `(cart_id)`

**Invariants**

- Rx items (`requires_prescription`) must carry `pet_id` before checkout quote succeeds
- Prices are re-read from `product_variants` at quote/checkout; cart stores no price snapshot

## orders

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `ord_…` |
| user_id | uuid | no | Cross-schema ref by id |
| client_id | uuid | yes | Cross-schema ref by id → `clients.id` when linked |
| status | text | no | `pending_payment` \| `awaiting_prescription` \| `paid` \| `fulfilling` \| `fulfilled` \| `cancelled` \| `refunded` |
| subtotal | numeric(10,2) | no | |
| tax_total | numeric(10,2) | no | |
| shipping_total | numeric(10,2) | no | 0 for pickup |
| grand_total | numeric(10,2) | no | |
| currency | char(3) | no | |
| fulfilment_method | text | no | `ship` \| `pickup` |
| shipping_address_json | jsonb | yes | Snapshot at order time |
| idempotency_key | text | yes | |
| placed_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_orders_idempotency` on `(idempotency_key)` where not null
- `ix_orders_user_placed` on `(user_id, placed_at desc)`
- `ix_orders_status` on `(status, placed_at desc)`

**Invariants**

- `awaiting_prescription` is set when any line `requires_prescription`; order cannot move to `paid` until Pharmacy events approve all Rx lines (or staff override per policy)
- Totals are recomputed server-side; client totals are advisory

## order_items

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| order_id | uuid | no | FK → `orders.id` |
| product_id | uuid | no | FK → `products.id` |
| variant_id | uuid | no | FK → `product_variants.id` |
| quantity | integer | no | |
| unit_price | numeric(10,2) | no | Snapshot at placement |
| line_total | numeric(10,2) | no | |
| pet_id | uuid | yes | Cross-schema ref by id for Rx lines |
| prescription_request_id | uuid | yes | Cross-schema ref by id → `pharmacy.prescription_requests.id` |

**Indexes**

- `ix_order_items_order` on `(order_id)`

## fulfilments

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `ful_…` |
| order_id | uuid | no | FK → `orders.id` |
| method | text | no | `ship` \| `pickup` |
| status | text | no | `pending` \| `ready` \| `handed_over` \| `shipped` \| `delivered` \| `cancelled` |
| carrier | text | yes | |
| tracking_number | text | yes | |
| pickup_code | text | yes | For curbside |
| estimated_ready_at | timestamptz | yes | |
| completed_at | timestamptz | yes | |
| created_at | timestamptz | no | |

**Invariants**

- One fulfilment row per order for the v1 single-choice model; split shipment requires a new ADR
- Pickup never ships; ship never hands over in person

## payments

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `pay_…` |
| order_id | uuid | no | FK → `orders.id` |
| gateway | text | no | e.g. `stripe`, `paypal` |
| gateway_payment_ref | text | no | Tokenised reference; **never PAN data** |
| status | text | no | `pending` \| `captured` \| `failed` \| `refunded` \| `partially_refunded` |
| amount | numeric(10,2) | no | |
| currency | char(3) | no | |
| failure_code | text | yes | |
| idempotency_key | text | yes | |
| captured_at | timestamptz | yes | |
| created_at | timestamptz | no | |

**Indexes**

- `uq_payments_gateway_ref` on `(gateway, gateway_payment_ref)`
- `ix_payments_order` on `(order_id)`

**Invariants**

- Card numbers, CVVs, and expiry never touch this schema or the API process (ADR-0007)
- Webhook id (`payments` webhook) is idempotent; duplicate capture events update status, never double-fulfil

## subscriptions

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `sub_…` |
| user_id | uuid | no | Cross-schema ref by id |
| product_id | uuid | no | FK → `products.id` |
| variant_id | uuid | no | FK → `product_variants.id` |
| pet_id | uuid | yes | Cross-schema ref by id for Rx auto-refill |
| frequency | text | no | `monthly`, `quarterly`, etc. |
| status | text | no | `active` \| `paused` \| `cancelled` |
| next_order_at | timestamptz | no | |
| cancel_reason | text | yes | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `ix_subscriptions_due` on `(status, next_order_at)`
- `ix_subscriptions_user` on `(user_id)`

**Invariants**

- Rx subscriptions require an approved prescription state before each auto-order; otherwise they pause and notify
- Cancellation stops future orders; existing orders are unaffected

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → Pharmacy | Rx lines raise `PrescriptionRequested`; decisions arrive as events |
| → Client & Patient | `pet_id` claims for Rx items |
| → Notifications | order/Rx events drive owner emails |
| → Operations | catalog maintenance writes through this context |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Commerce domain |
| `../domain-events.md` | `OrderPlaced`, `PaymentCaptured`, `OrderFulfilled` |
| `../api-specification.md` | Catalog, cart, checkout, orders APIs |
| `pharmacy-authorisation.md` | Rx hold coupling |
| `../architectural-decision-record.md` | ADR-0007 tokenised payments |

---

# Acceptance Criteria

- Checkout is idempotent; retries never double-charge or double-order
- No schema column stores card data
- Rx orders cannot reach `paid` without Pharmacy approval events
- Stock reserved at quote is visible in `product_variants.stock_reserved`

---

# Guiding Principle

> **Commerce is a promise about money and stock. Snapshot prices at placement, reserve honestly, and let the pharmacy guardrail decide what may actually ship.**
