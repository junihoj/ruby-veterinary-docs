# Commerce Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Commerce Data Model
>
> **Version:** 1.1.0
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

Defines the tables behind the storefront and everything between browsing and fulfilment: product catalog with seven product types (WooCommerce parity), categories, search facets, carts, order lifecycle, fulfilment choice, payment references, and subscriptions/auto-refill. Money moves through a tokenised gateway; this schema stores references and state, never card data.

---

# Responsibilities

The Commerce context owns:

- Product catalog in seven types — `simple`, `variable`, `grouped`, `external`, `composite`, plus `is_virtual` and `is_downloadable` flags — and variants
- Product categories and catalog facets (pet type, life stage, condition)
- Carts and cart items (including composite selections)
- Orders, order items, fulfilment, payment references
- Subscriptions and auto-refill schedules
- Stock counts per variant
- Download files and download grants for downloadable products

It does **not** own prescription decisions (Pharmacy emits events back), pet records, or client profiles. Prescription-only items reference `pet_id` at claim time and flow through Pharmacy's hold.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Product | `products` | Variants, grouped members, composite slots, and download files belong to it; stock is per-variant |
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
| product_type | text | no | `simple` \| `variable` \| `grouped` \| `external` \| `composite` |
| is_virtual | boolean | no | Default false; excludes the product from shipping weight and physical fulfilment |
| is_downloadable | boolean | no | Default false; grants file access after payment (see `product_download_files`) |
| pricing_mode | text | yes | Composite only: `fixed` (price on parent variant) \| `sum` (sum of selected options); null for other types |
| external_url | text | yes | External only; outbound purchase link |
| button_text | text | yes | External only; CTA label, e.g. `Buy at partner` |
| requires_prescription | boolean | no | True for Rx items; pharmacy hold engages at checkout. Derived for composite (OR of slot options); never true for external |
| requires_pet_claim | boolean | no | Owner must associate a pet (links to clients.pets by id); derived for composite |
| published | boolean | no | |
| featured | boolean | no | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `uq_products_slug` unique on `(slug)`
- `ix_products_catalog` on `(published, category_id, featured)`

**Invariants**

- `variable` products have ≥ 2 variants; `simple` products have exactly 1 implied/default variant row for stock/SKU uniformity
- `grouped` products have no variants and no price of their own; members are purchased as their own cart lines
- `external` products are never purchasable on-site: no cart, no checkout, no stock tracking, no Rx, no subscriptions
- `composite` products have ≥ 1 slot; `fixed` pricing carries one default variant with the price; `sum` pricing has no purchasable variant
- `is_virtual` / `is_downloadable` are set on `simple` and `variable` products (e.g. variable + downloadable); for `composite` and `grouped`, virtual/downloadable behaviour is derived from the selected or child items at quote time; never combined with `external`
- Downloadable products reference ≥ 1 row in `product_download_files`

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
| weight_grams | integer | yes | Shipping calc; null for virtual/downloadable items |
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
- Virtual and downloadable variants carry no weight

## composite_slots

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| composite_product_id | uuid | no | FK → `products.id` (composite) |
| name | text | no | e.g. `Food bag`, `Supplement` |
| description | text | yes | |
| position | integer | no | Display order |
| min_select | integer | no | ≥ 0; 0 = optional slot |
| max_select | integer | no | ≥ `min_select` |
| created_at | timestamptz | no | |

**Indexes**

- `ix_composite_slots_product` on `(composite_product_id, position)`

**Invariants**

- A composite product has ≥ 1 slot
- `max_select` ≥ `min_select` ≥ 0

## composite_slot_options

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| slot_id | uuid | no | FK → `composite_slots.id` |
| product_id | uuid | no | FK → `products.id` (simple/variable candidate) |
| component_variant_id | uuid | yes | FK → `product_variants.id` when the option pins a specific variant |
| quantity | integer | no | ≥ 1; units per selection |
| price_override | numeric(10,2) | yes | Overrides the candidate's price within this composite |
| sort_order | integer | no | |
| created_at | timestamptz | no | |

**Indexes**

- `uq_composite_slot_options` on `(slot_id, product_id, component_variant_id)`
- `ix_composite_slot_options_product` on `(product_id)`

**Invariants**

- A slot has ≥ 1 option
- Options must be `simple` or `variable` products (no nested composites, no grouped, no external)
- `price_override` never negative

## grouped_members

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| grouped_product_id | uuid | no | FK → `products.id` (grouped) |
| member_product_id | uuid | no | FK → `products.id` (simple/variable member) |
| sort_order | integer | no | |
| created_at | timestamptz | no | |

**Indexes**

- `uq_grouped_members` on `(grouped_product_id, member_product_id)`

**Invariants**

- A grouped product has ≥ 1 member
- Members must be `simple` or `variable` products
- A grouped product must not appear as its own member

## product_download_files

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| product_id | uuid | no | FK → `products.id` (downloadable) |
| file_name | text | no | Display name |
| object_key | text | no | MinIO object key (ADR-0009); never a public URL |
| file_size_bytes | integer | no | |
| mime_type | text | no | |
| sort_order | integer | no | |
| created_at | timestamptz | no | |

**Indexes**

- `ix_product_download_files_product` on `(product_id, sort_order)`

**Invariants**

- A downloadable product references ≥ 1 file
- Files are served only through short-lived signed URLs issued by the API; bytes never pass through the API process (ADR-0009)

## product_download_grants

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `dgr_…` |
| order_item_id | uuid | no | FK → `order_items.id` |
| product_id | uuid | no | FK → `products.id` (denormalised for lookup) |
| downloads_used | integer | no | Default 0 |
| max_downloads | integer | yes | Null = unlimited |
| expires_at | timestamptz | yes | Null = never expires |
| last_download_at | timestamptz | yes | |
| revoked_at | timestamptz | yes | Staff revocation |
| created_at | timestamptz | no | |

**Indexes**

- `uq_product_download_grants` on `(order_item_id, product_id)`
- `ix_product_download_grants_expiry` on `(expires_at)` where not null

**Invariants**

- A grant exists only for a paid order line containing a downloadable product
- `downloads_used` never exceeds `max_downloads` when set
- Revoked grants stop serving immediately

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
| variant_id | uuid | yes | FK → `product_variants.id`; null for sum-priced composite lines |
| quantity | integer | no | ≥ 1 |
| pet_id | uuid | yes | Cross-schema ref by id → `clients.pets.id` for Rx items |
| added_at | timestamptz | no | |

**Indexes**

- `uq_cart_items_line` unique on `(cart_id, variant_id, pet_id)`
- `ix_cart_items_cart` on `(cart_id)`

**Invariants**

- Rx items (`requires_prescription`) must carry `pet_id` before checkout quote succeeds
- Prices are re-read from `product_variants` at quote/checkout; cart stores no price snapshot
- Composite lines carry their selection in `cart_item_components`; `variant_id` is the parent's default variant for `fixed` pricing and null for `sum` pricing
- `external` products never enter a cart

## cart_item_components

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| cart_item_id | uuid | no | FK → `cart_items.id` |
| slot_id | uuid | no | FK → `composite_slots.id` |
| product_id | uuid | no | FK → `products.id` |
| variant_id | uuid | no | FK → `product_variants.id` |
| quantity | integer | no | ≥ 1 |
| added_at | timestamptz | no | |

**Indexes**

- `uq_cart_item_components` on `(cart_item_id, slot_id, product_id, variant_id)`

**Invariants**

- Exists only for composite cart lines
- Selections satisfy the slot's `min_select` / `max_select`
- Prices are re-read from `product_variants` (or `price_override`) at quote; the cart stores no price snapshot

## orders

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `ord_…` |
| user_id | uuid | no | Cross-schema ref by id |
| client_id | uuid | yes | Cross-schema ref by id → `clients.id` when linked |
| status | text | no | `pending_payment` \| `awaiting_prescription` \| `paid` \| `fulfilling` \| `fulfilled` \| `cancelled` \| `refunded` |
| subtotal | numeric(10,2) | no | |
| tax_total | numeric(10,2) | no | |
| shipping_total | numeric(10,2) | no | 0 for pickup and digital |
| grand_total | numeric(10,2) | no | |
| currency | char(3) | no | |
| fulfilment_method | text | no | `ship` \| `pickup` \| `digital` |
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
- An order whose lines are all virtual/downloadable uses `digital` fulfilment and skips physical fulfilment; download grants issue on payment capture

## order_items

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| order_id | uuid | no | FK → `orders.id` |
| product_id | uuid | no | FK → `products.id` |
| variant_id | uuid | yes | FK → `product_variants.id`; null for sum-priced composite lines |
| quantity | integer | no | |
| unit_price | numeric(10,2) | no | Snapshot at placement |
| line_total | numeric(10,2) | no | |
| pet_id | uuid | yes | Cross-schema ref by id for Rx lines |
| prescription_request_id | uuid | yes | Cross-schema ref by id → `pharmacy.prescription_requests.id` |

**Indexes**

- `ix_order_items_order` on `(order_id)`

**Invariants**

- Composite lines snapshot their selection in `order_item_components`; one `prescription_request_id` per line covers all Rx components in it

## order_item_components

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| order_item_id | uuid | no | FK → `order_items.id` |
| slot_id | uuid | no | FK → `composite_slots.id` |
| product_id | uuid | no | FK → `products.id` |
| variant_id | uuid | no | FK → `product_variants.id` |
| quantity | integer | no | ≥ 1 |
| unit_price | numeric(10,2) | no | Snapshot at placement |
| created_at | timestamptz | no | |

**Indexes**

- `ix_order_item_components_item` on `(order_item_id)`

**Invariants**

- Snapshot of the composite selection at placement; the staff pick/pack list
- Rx components are listed here; the line's single `prescription_request_id` covers them

## fulfilments

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `ful_…` |
| order_id | uuid | no | FK → `orders.id` |
| method | text | no | `ship` \| `pickup` \| `digital` |
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
- Digital fulfilments complete automatically when payment captures; ship/pickup follow the manual flow

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

- Only physical `simple` / `variable` products may be subscribed (never virtual, downloadable, external, grouped, or composite)
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
- Every one of the seven product types is representable and purchasable (or deliberately not purchasable, for `external`) per its invariants
- Composite selections satisfy slot `min_select` / `max_select` at quote and checkout
- Download grants issue only after payment and stop serving on revocation

---

# Guiding Principle

> **Commerce is a promise about money and stock. Snapshot prices at placement, reserve honestly, and let the pharmacy guardrail decide what may actually ship.**
