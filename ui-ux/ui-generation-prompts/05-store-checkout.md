# Google Stitch Prompt — ruby-veterinary Store & Checkout

> **Purpose:** Paste this prompt into Google Stitch to generate the online store and checkout surfaces for ruby-veterinary — general supplies, therapeutic diets, and prescription items behind vet authorisation.
>
> **Coverage:** 7 screens — catalog grid with categories, filter rail, product detail, cart, checkout, payment step, order confirmation + order history.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Prescription holds are informational (info icon + text) — never ruby, never styled like an error.

---

## Design System Context (Paste First)

```
I'm designing ruby-veterinary — a single veterinary clinic's website: emergency-first public site (phone, hours, address always visible), services and staff pages, clinic blog, WhatsApp triage bot with human handover, online store (general supplies, therapeutic diets, prescription items behind vet authorisation), client intake + appointment requests, and a role-scoped staff back office.

TARGET PLATFORM: Responsive web — mobile-first for pet owners, desktop-first for staff tools. Next.js 16 + React 19 + Tailwind CSS 4. Light mode AND dark mode are both first-class.

BRAND PERSONALITY: Calm, caring, trustworthy, urgent only when needed. Plain-spoken, never salesy in an emergency context.

COLOR SYSTEM (light / dark):
- Page background: #FFFFFF / #171310
- Warm alternate sections: #FAF7F5 / #1F1A16
- Cards and panels: #F6F2EF / #1F1A16
- Subtle fills: #EFE9E4 / #2A2420
- Elevated (modals, popovers): #FFFFFF / #241E1A
- Borders: #E7DFD8 / #3A322C · stronger: #D4C8BE / #4C423A
- Text primary: #1F1A17 / #F7F3F0 · secondary: #5C534C / #C4B8AE · tertiary: #8A7F77 / #94887D · inverse: #FFFFFF / #171310
- Links: #9B111E / #F53D6D
- Emergency ruby fill (Call now): #9B111E / #F53D6D with white text; hover #C00E52 / #E0115F
- Ruby bright accent and focus ring: #E0115F / #F53D6D
- Ruby soft surfaces: #FFF5F7 / #FFE4EC (light) · #1F1A16 with #F53D6D accents (dark)
- Everyday primary buttons (Book appointment, Add to cart, Submit, Continue, Save, Publish): #4A6659 with white text / #A3C4B0 with #171310 text; hover #3B5349 / #7BA88C
- Sage soft panels: #F4F8F5 and #E3EDE6 / #241E1A and #2A2420 · sage strong text and secondary links: #3B5349 / #A3C4B0
- Success: #2F7D5A on #EAF4EF / #4ADE9B on #123528
- Warning: #B7791F on #FBF3E4 / #E3B341 on #3B2E10
- Error text: #B8431F / #F0754A · destructive fill: #D65328 / #E85C30 with white text · error surfaces: #FDF0E8 / #3B1D10
- Info: #2B6CB0 on #EBF2FA / #6BA3D6 on #15273A
- Disabled: #EFE9E4 fill with #8A7F77 text / #2A2420 fill with #94887D text
- Focus ring: 3px #E0115F light / 3px rgba(245,61,109,0.4) dark

RUBY DISCIPLINE (hard rule): ruby #9B111E / #F53D6D only on emergency CTAs, "Call now" buttons, links, and documented brand accents. Everyday actions use sage #4A6659 / #A3C4B0. Errors and destructive confirms use #B8431F text / #D65328 fill plus an icon and text — never brand ruby. No full-width ruby hero backgrounds; never every button ruby.

TYPOGRAPHY:
- UI: Inter (system-ui, sans-serif) · Blog article bodies and article card titles: Merriweather (Georgia, serif) · SKUs/IDs: JetBrains Mono
- Display 48/56 · H1 36/44 · H2 28/36 · H3 22/30 · H4 18/26 · Body large 18/28 · Body 16/24 (default) · Body small 14/20 · Caption 12/16
- Headings 600 weight, -0.01em tracking · body 400 · min 16px interactive text in mobile forms

SPACING (4px base): 0, 2, 4, 6, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96

RADIUS: 4px badges · 8px buttons/inputs · 12px cards · 16px modals · 9999px avatars/pills

SHADOWS: sm 0 1px 3px rgba(31,26,23,.08) · md 0 4px 12px rgba(31,26,23,.10) · lg 0 12px 28px rgba(31,26,23,.14) · dark: raise opacity to 0.35+

Z-INDEX: base 0 · sticky 100 · header 110 · dropdown 200 · overlay 400 · modal 500 · toast 600

MOTION: 75/150/200/300ms, easing cubic-bezier(0.4, 0, 0.2, 1); respect prefers-reduced-motion

BREAKPOINTS: 640 / 768 / 1024 / 1280 · 12-col grid · max width 1280px · gutters 16/24/32

EMERGENCY PATTERN: sticky mobile tap-to-call bar with the clinic phone; phone, hours and address visible without opening a menu; phone fallback text on every degraded, empty, or error state; phone icon always paired with the visible number.

ACCESSIBILITY (WCAG 2.1 AA): keyboard operable, visible focus ring, alt text on meaningful images, ≥4.5:1 normal text contrast (both modes), tap targets ≥44px on owner-facing mobile, never colour alone — errors carry an icon and text.

Generate a DESIGN.md from this specification first, then proceed to the screen prompts below.
```

---

## Dark Mode Color Mapping (Apply to all screens)

Every screen below must be generated in BOTH light and dark mode. Same layout for both — only colours change.

- Page background: `#FFFFFF` → `#171310`
- Warm sections: `#FAF7F5` → `#1F1A16`
- Cards/panels: `#F6F2EF` → `#1F1A16` · subtle fills `#EFE9E4` → `#2A2420` · elevated `#FFFFFF` → `#241E1A`
- Borders: `#E7DFD8` → `#3A322C` · stronger `#D4C8BE` → `#4C423A`
- Text: primary `#1F1A17` → `#F7F3F0` · secondary `#5C534C` → `#C4B8AE` · tertiary `#8A7F77` → `#94887D` · inverse `#FFFFFF` → `#171310`
- Links: `#9B111E` → `#F53D6D`
- Emergency ruby CTAs ("Call now"): fill `#9B111E` → `#F53D6D`, white text in both modes · hover `#C00E52` → `#E0115F`
- Everyday primary buttons (Add to cart, Continue, Submit, Pay): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310` · hover `#3B5349` → `#7BA88C`
- Inputs/selects: fill `#FFFFFF` → `#1F1A16`, border `#E7DFD8` → `#3A322C`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Rx / prescription-hold notices: info `#2B6CB0` on `#EBF2FA` + border `#B8D0EA` → `#6BA3D6` on `#15273A` + border `#15273A`
- Rx status chips: Pending `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10` · Approved `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528` · Rejected `#B8431F` on `#FDF0E8` → `#F0754A` on `#3B1D10`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning: `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces

---

## Group 1: Browse & Discover

### Screen 1 — Store Catalog Grid with Category Filters

```
Generate the store catalog page for ruby-veterinary — the categorized inventory grid.

DESKTOP LAYOUT:
- Breadcrumb: Home / Store
- Page header on warm #FAF7F5: H1 "Clinic store" (36px), subline 16px #5C534C "Supplies, therapeutic diets, and prescription items — the same products we use in the clinic."
- Category tab bar (segmented control, 8px radius, background #F6F2EF, padding 4px):
  - Tabs: "General Pet Supplies" (active) · "Therapeutic Diets" · "Prescription Medication"
  - Active tab: white #FFFFFF fill, 1px #E7DFD8, shadow sm, text #1F1A17 weight 600
  - Inactive tab: text #5C534C; Prescription tab carries a small sage outline "Rx" badge
- Toolbar row: search input (360px, 40px height, search icon left), sort select "Featured", results count "142 products" 14px #8A7F77, and a "Filters" toggle button
- 4-column product card grid, each card white, 1px #E7DFD8, 12px radius, shadow sm:
  - Image area 1:1 on #F6F2EF, product image placeholder; badges top-left: "Rx" badge (pill, #EBF2FA background, #2B6CB0 text, 12px weight 600) on prescription items only; "In stock" green pill (#EAF4EF/#2F7D5A) bottom of image
  - Body: category caption 12px #8A7F77, title 16px weight 600 #1F1A17 (2-line clamp), variant note "3 sizes" 13px #5C534C, price 18px weight 600 #1F1A17 (or "From $24.90")
  - Card footer: sage #4A6659 outlined "Add to cart" button (40px, 8px radius) — sage, never ruby
- Empty results variant: sage-50 #F4F8F5 panel, line icon, H2 "No products match those filters", body "Try clearing a filter, or call the clinic on 010 555 0199 and we'll check for you.", outlined "Clear filters" button + ruby phone link

MOBILE LAYOUT (primary):
- H1 + subline; category tabs become a horizontally scrollable segmented row (each tab ≥48px)
- Search full width (48px); "Filters" button full width with count badge "3"
- 2-column product grid, 12px gutters, 16px page padding; cards stack images full width of column
- Add to cart button full width within card, 44px
- Empty state full width with full-width outlined button and ruby call row

ACCESSIBILITY:
- Tabs implement a tablist pattern (role="tablist", aria-selected, arrow-key navigation) OR simple links with aria-current — pick one and be consistent
- Product cards: one focus target each, accessible name includes product title and price; focus ring 3px #E0115F
- "Rx" badge and stock state are text, never colour alone; images have alt text ("Hill's Prescription Diet k/d, 2 kg bag")
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #5C534C on #F6F2EF ≈ 6.8:1 · #4A6659 border/text on white ≈ 5.9:1 for icons; dark: #6BA3D6 on #15273A ≈ 5.5:1, #4ADE9B on #123528 ≈ 8:1, #A3C4B0 on #1F1A16 ≈ 8.5:1
- Tap targets ≥44px (mobile cards' buttons 44px+); keyboard scrollable tab row without scroll traps

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 2 — Filter Rail (pet type, life stage, health condition)

```
Generate the store filter rail for ruby-veterinary — faceted filtering by pet type, life stage, and health condition.

DESKTOP LAYOUT (left rail, 260px, sticky under the header):
- Rail header: "Filters" H3 22px + "Clear all" ruby #9B111E text link (14px) + count badge
- Facet groups (accordion, 1px #E7DFD8 dividers, chevron right):
  - "Pet type": checkboxes — Cat · Dog (checkbox 18px, 1px #D4C8BE, checked fill #4A6659 with white check, 8px radius... use 4px radius for checkboxes)
  - "Life stage": Puppy/Kitten · Adult · Senior
  - "Health condition": Kidney support · Weight management · Digestive care · Joint care · Skin & coat · Dental
  - "Availability": In stock only · Prescription items
  - "Price": range slider with min/max inputs ($0 – $150)
- Each checkbox row: 44px tall, label 15px #1F1A17, count in parentheses 13px #8A7F77 (e.g. "Kidney support (24)")
- Selected filter chips appear above the grid: pill chips with × remove buttons — sage-50 #F4F8F5 fill, #3B5349 text, 44px tall
- Rail footer: sage #4A6659 filled "Show 38 products" button (44px)

MOBILE LAYOUT (primary, bottom sheet):
- Filter button in toolbar opens a bottom sheet (radius 16px top, backdrop rgba(0,0,0,0.45), z-index modal 500)
- Sheet header: "Filters" + close × button (44px) + "Clear all" link
- Facet groups expanded in one scrollable list (same checkbox rows, 48px tall)
- Sticky sheet footer: sage "Show 38 products" full width 56px
- Selected chips shown inline above the grid once the sheet closes

ACCESSIBILITY:
- Checkboxes are real inputs with labels; group headings are fieldset legends or heading + group role
- Bottom sheet: focus moves into the sheet, is trapped while open, Esc closes, focus returns to the Filters button; background inert
- Price slider has paired numeric inputs so keyboard users can type values
- Focus ring 3px #E0115F on every control; checked state shown by check glyph + fill, not fill alone
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #3B5349 on #F4F8F5 ≈ 6.9:1 · #8A7F77 counts on #FFFFFF ≈ 3.9:1 (decorative counts only) · white on #4A6659 ≈ 5.9:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1, #A3C4B0 fill/labels on #171310 ≈ 9.4:1, #94887D on #1F1A16 ≈ 4.6:1
- Tap targets ≥44px rows; sheet footer button 56px

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 3 — Product Detail Page

```
Generate the product detail page for ruby-veterinary — a variable prescription diet product with a variant picker, stock, and Rx badge.

DESKTOP LAYOUT:
- Breadcrumb: Home / Store / Therapeutic Diets / Hill's Prescription Diet k/d
- Two-column layout (7 + 5) on white:
  - Left: main image placeholder (1:1, 12px radius, on #F6F2EF), 4 thumbnail placeholders below; zoom affordance on hover
  - Right column:
    - Category caption 12px uppercase #3B5349 "Therapeutic Diets"
    - H1 "Hill's Prescription Diet k/d Kidney Care" (28px, #1F1A17)
    - Rating row (optional) + "Sold by ruby-veterinary clinic" 14px #5C534C
    - Price "From $54.90" 24px weight 600 #1F1A17 with per-unit note "2 kg bag" 14px #8A7F77
    - Rx badge row: info-style badge — icon + "Prescription required" text on #EBF2FA, border 1px #B8D0EA, text #2B6CB0, 8px radius (NOT ruby, NOT red)
    - Rx explanation panel directly under price: #EBF2FA background, info icon #2B6CB0, 14px #1F1A17 text: "This is a prescription diet. Add it to your cart as usual — a veterinarian checks your pet's file and approves it before we ship. Most orders are reviewed within one business day."
    - Variant picker "Bag size": segmented option buttons 0.5 kg / 2 kg / 5 kg — selected: 2px #4A6659 border, #F4F8F5 background, weight 600; unselected: 1px #E7DFD8. Each option shows its own price; one option shows "Out of stock" (#8A7F77 text, disabled)
    - Stock line: green dot + "In stock — 12 left" 14px #2F7D5A (out-of-stock option: "Out of stock" with #8A7F77)
    - Quantity stepper (− / 2 / +) 44px, 1px #D4C8BE, 8px radius
    - Sage #4A6659 filled "Add to cart" (48px, white text, full column width) — everyday primary, sage not ruby
    - Under button: "or" + ruby #9B111E link "Ask us about this product: 010 555 0199"
    - Trust row: 3 items with 20px icons — "Free in-clinic pickup", "Ships in 1–2 days", "Auto-refill available"
  - Below: tabbed section (Description · Feeding guide · Ingredients · FAQs), body 16px/24px; feeding guide as a real table

MOBILE LAYOUT (primary):
- Image full width 1:1, thumbnails horizontal scroll
- Stacked: caption, H1 24px, Rx badge, price, Rx explanation panel (icon above text), variant buttons wrapping 2-wide (48px tall), stock line, quantity stepper, full-width sage "Add to cart" 56px
- Trust row becomes 3 stacked rows with icons
- Tabs horizontally scrollable; feeding table gets horizontal scroll wrapper with visible hint
- Sticky bottom bar while scrolling: price left, "Add to cart" sage button right (above the tap-to-call bar)

ACCESSIBILITY:
- Variant buttons are a radiogroup (aria-checked) with keyboard arrow support; disabled/out-of-stock option has aria-disabled and a text reason
- Rx panel has an accessible heading or is associated with the price block; icon aria-hidden, text carries meaning — prescription state never conveyed by colour alone
- Quantity stepper buttons named ("Decrease quantity", "Increase quantity"); input type number with visible label
- Focus ring 3px #E0115F; contrast: #2B6CB0 on #EBF2FA ≈ 6.2:1 · #4A6659 on #F4F8F5 ≈ 5.6:1 (borders/icons) · white on #4A6659 ≈ 5.9:1 · #1F1A17 on #FFFFFF ≈ 16:1 · #2F7D5A on #FFFFFF ≈ 4.9:1; dark: #6BA3D6 on #15273A ≈ 5.5:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #4ADE9B on #171310 ≈ 9:1
- Images alt text includes product name and size; tap targets ≥44px (variant chips 48px, stepper 48px)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Cart & Checkout

### Screen 4 — Cart Drawer / Cart Page

```
Generate the shopping cart for ruby-veterinary — show it as a right-side drawer on desktop and a full page on mobile.

DESKTOP LAYOUT (drawer, 420px wide, from the right, backdrop rgba(0,0,0,0.45), z-index overlay 400):
- Drawer header: "Your cart" H3 22px + item count "3 items" 14px #8A7F77 + close × (44px)
- Line item rows (white, 1px #E7DFD8 bottom border, padding 16px):
  - Thumbnail 64px on #F6F2EF, title 15px weight 600 #1F1A17, variant 13px #5C534C ("2 kg bag"), SKU 12px JetBrains Mono #8A7F77
  - Qty stepper (44px controls), line price right-aligned 15px weight 600, remove link 13px #B8431F with trash icon (removal is destructive — error tokens, NOT ruby)
  - Rx item badge: info badge icon + "Prescription item" on #EBF2FA/#2B6CB0
- Prescription-hold notice (panel above totals): #EBF2FA background, 1px #B8D0EA border, 8px radius, info icon #2B6CB0, text 14px #1F1A17: "1 item needs veterinary approval. We'll hold the order and email you when a vet reviews it — usually within one business day. You won't be charged until it's approved." (never ruby, never styled as an error)
- Fulfilment selector (radio cards): "Ship to me — $8.90" and "Free in-clinic pickup" — selected: 2px #4A6659, #F4F8F5 bg
- Order summary: Subtotal · Tax · Shipping · Total (18px weight 600), rows 14px
- Sage #4A6659 filled "Continue to checkout" (48px, full width)
- Footer note with phone icon: "Questions about a product? Call 010 555 0199"

MOBILE LAYOUT (primary, full page):
- Header "Your cart · 3 items" + close/back
- Line items full width, thumb 72px, stepper full row below details
- Rx notice full width (icon above text if narrow)
- Fulfilment radios full width, 56px cards
- Summary table full width; total row 20px
- Sticky bottom: total left, sage "Checkout" button right (56px), above the tap-to-call bar
- Empty cart variant: sage-50 #F4F8F5 panel, cart icon, H2 "Your cart is empty", body "Browse supplies, diets, and prescription items.", outlined "Browse the store" button + ruby "Call 010 555 0199" row

ACCESSIBILITY:
- Drawer: role="dialog" aria-modal="true", labelled by "Your cart"; focus trapped, Esc closes, focus returns to cart button
- Qty steppers and remove controls named per item ("Remove Hill's k/d 2 kg from cart"); removal is a text+icon control in #B8431F, not brand ruby
- Focus ring 3px #E0115F; errors/removals use icon + text
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #B8431F on #FFFFFF ≈ 5.5:1 · white on #4A6659 ≈ 5.9:1 · #3B5349 on #F4F8F5 ≈ 6.9:1; dark: #6BA3D6 on #15273A ≈ 5.5:1 · #F0754A on #171310 ≈ 6.4:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px (mobile 48px); summary values are text; total announced as text not image

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 5 — Checkout Page

```
Generate the checkout page for ruby-veterinary — contact, patient/vet association, fulfilment, and the prescription-hold notice.

DESKTOP LAYOUT:
- Breadcrumb-free checkout header: "Checkout" H1 (36px) + step indicator "1 Details · 2 Payment · 3 Confirmation" (active sage #4A6659, upcoming #8A7F77, completed #2F7D5A checks)
- Two-column (7 + 5) on white:
  - Left column, white card sections separated by 1px #E7DFD8:
    - Section "Contact": Email, Phone fields (44px, labels above)
    - Section "Delivery": radio cards — "Ship to me" (address fields revealed) / "Free in-clinic pickup" (helper: "Ready in 2 hours during opening times")
    - Section "Pet and veterinarian" (MANDATORY, marked with text "(required)"):
      - "Which pet is this order for?" select (Biscuit — Dog · Momo — Cat) + outlined "Add a pet" link
      - "Your veterinarian on file" select (Dr Amara Okoye · Dr Ryan Meyer)
      - Helper 13px #5C534C: "We check these against your clinic record before approving prescription items."
    - Section "Prescription items in this order" — prescription-hold notice: #EBF2FA panel, 1px #B8D0EA, info icon #2B6CB0, title 15px weight 600 "1 prescription item needs vet approval", body 14px #1F1A17 "Add this order as normal. A veterinarian reviews it against your pet's file before anything ships. If it isn't suitable, we'll call you — and you won't be charged."
    - Section "Notes for the clinic" textarea (optional)
    - Sage #4A6659 filled "Continue to payment" (48px) + "Back to cart" text link
  - Right column sticky order summary card: items (thumb, title, variant, price), Rx badge, fulfilment line, subtotal/tax/shipping/total, note "Card details are entered on our secure payment provider's page — we never see or store them." with lock icon 14px #5C534C

MOBILE LAYOUT (primary):
- Step indicator compact bar at top
- Sections stack full width, 16px padding; pet/vet selects full width 48px
- Prescription-hold notice full width, icon above text if narrow, cannot be dismissed
- Summary collapses into a "Order summary (3 items)" accordion above the sticky footer
- Sticky footer: total left, sage "Continue to payment" right (56px), above the tap-to-call bar

ACCESSIBILITY:
- Fieldsets with legends per section; mandatory fields marked with text, not only an asterisk (asterisk plus "(required)" acceptable with a legend note)
- Radiocards are real radio inputs with labels; selected state has 2px border + background + check icon, not colour alone
- Prescription notice is a complementary region with a heading; icon aria-hidden, text carries meaning
- Focus ring 3px #E0115F; keyboard path reaches every control in reading order
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #5C534C on #FFFFFF ≈ 7.4:1 · white on #4A6659 ≈ 5.9:1 · #2F7D5A on #FFFFFF ≈ 4.9:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px; no colour-only required-field signalling

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 6 — Payment Step (secure gateway)

```
Generate the payment step for ruby-veterinary — hand-off to the tokenised payment gateway. No card fields exist on our page.

DESKTOP LAYOUT:
- Same checkout shell; step indicator now at "2 Payment" (step 1 completed green check, step 2 active sage)
- Left column (7 cols), white card:
  - H2 "Payment" (28px)
  - Security panel: #EBF2FA background, 8px radius, lock icon #2B6CB0, title 15px weight 600 "You're paying on our secure gateway", body 14px #1F1A17: "Card details are entered on our payment provider's hosted page over TLS. ruby-veterinary never sees or stores your card number (PCI-DSS handled by the provider)."
  - Payment method chooser (radio rows, 48px): "Pay by card (Stripe-class gateway)" with card icons · "Apple Pay / Google Pay" with wallet icon — each row: 1px #E7DFD8 border, selected 2px #4A6659 + #F4F8F5 bg
  - Sage #4A6659 filled "Pay $86.40 securely" button (48px) with small lock icon
  - Redirect note under button: "You'll be taken to a secure payment page to enter card details, then returned here." 13px #5C534C
  - Prescription reminder line: info icon + "This order still includes 1 prescription item — approval happens after payment, and we'll refund instantly if it can't be authorised." 14px #1F1A17 on #EBF2FA
  - "Back to details" text link
- Right column: locked order summary (same as Screen 5), with a caption "3 items · shipping to Rosebank"
- Gateway-down degraded state (shown as an alternate panel): warning icon #B7791F on #FBF3E4 + text "Our payment provider isn't responding right now. Nothing has been charged." + ruby phone link "Call 010 555 0199 to order by phone" + outlined "Try again" button

MOBILE LAYOUT (primary):
- Step bar compact; security panel full width with icon above text
- Method radios full width, 56px rows; card icons right-aligned
- Pay button full width 56px sage with lock icon; redirect note below in 13px
- Prescription reminder full width; degraded state full width with ruby call row as a 48px tap target
- Order summary accordion above sticky pay footer

ACCESSIBILITY:
- No card input fields on our page — show an explicit annotation "No card fields here by design"
- Payment methods are radio inputs with labels; selected state = border + background + text weight
- Pay button announces the amount ("Pay $86.40 securely"); lock icon aria-hidden
- Degraded state uses role="alert", warning icon + text, and a working phone fallback — never a dead end
- Focus ring 3px #E0115F; contrast: #2B6CB0 on #EBF2FA ≈ 6.2:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · white on #4A6659 ≈ 5.9:1 · #1F1A17 on #FFFFFF ≈ 16:1; dark: #6BA3D6 on #15273A ≈ 5.5:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px; keyboard-only path completes method selection and pay action

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 3: After the Order

### Screen 7 — Order Confirmation + Order History with Rx Status

```
Generate the order confirmation page and the account order history list for ruby-veterinary, including prescription status badges. Show both states on one canvas (confirmation on top, history below) or as two linked frames.

DESKTOP LAYOUT:
A) Confirmation (single centred column, max-width 720px):
- Success icon circle 64px (#EAF4EF / #2F7D5A check)
- H1 "Order received" (36px), paragraph 18px/28px #1F1A17: "Thanks — order RV-ORD-77123 is in. We've emailed your receipt. One item needs veterinary approval before it ships."
- Order reference card on sage-50 #F4F8F5: "Order number" caption + "RV-ORD-77123" JetBrains Mono 18px weight 600 + copy button
- Status timeline (ordered list, dots): Paid (green #2F7D5A) → Awaiting vet approval (amber #B7791F, current, larger dot with ring) → Approval due within 1 business day (grey #8A7F77)
- Prescription-hold notice: #EBF2FA panel, info icon #2B6CB0, text "We've paused the prescription item until a vet checks your pet's file. If approved, we ship in 1–2 days. If not, we'll call you and refund that line immediately."
- Fulfilment card: "Free in-clinic pickup · Ready from Wed 10:00 · 14 Maple Street" or "Ship to you · Est. Thu–Fri"
- Buttons: sage "View order" filled + outlined "Continue shopping"; ruby #9B111E phone link "Questions? Call 010 555 0199"

B) Account order history list (desktop table-like rows, 100% width):
- Columns: Order (mono ID + date) · Items (thumb cluster + "3 items") · Total · Status · Rx status · Action
- Status pills: "Fulfilled" green #2F7D5A/#EAF4EF · "Processing" amber #B7791F/#FBF3E4 · "Cancelled" grey #8A7F77/#EFE9E4 — each pill = icon + text
- Rx status pills (icon + text, never colour alone): "Rx approved" green #2F7D5A/#EAF4EF with check · "Rx pending review" amber #B7791F/#FBF3E4 with clock · "Rx declined — we'll call you" orange #B8431F/#FDF0E8 with alert icon (NOT ruby) · "No Rx items" grey #8A7F77/#EFE9E4
- Row action: outlined "View" button (40px); rows 1px #E7DFD8 bottom border, hover background #FAF7F5

MOBILE LAYOUT (primary):
A) Confirmation stacks: icon, H1, paragraph, reference card (code centred + copy), timeline vertical with dates under labels, notice panel, fulfilment card, stacked buttons (sage full width 56px, outlined full width)
B) Order history becomes cards: order ID + date row, status pills wrapping on their own lines (Rx pill visually separated below order pill), total row, "View order" full-width outlined button 48px
- Empty orders variant: sage-50 panel, icon, H2 "No orders yet", body "When you order supplies or refills, they'll appear here.", outlined "Browse the store" + ruby "Call 010 555 0199"

ACCESSIBILITY:
- Confirmation region role="status"; reference/order IDs selectable with copy buttons that announce "Copy order number"
- Timeline is a semantic ol; current step marked with text "current" as well as the ringed dot
- Status pills always contain their text label; icons aria-hidden
- Focus ring 3px #E0115F on buttons, pills-as-links, and copy controls; first focus is "View order"
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #B8431F on #FDF0E8 ≈ 5.4:1 · #8A7F77 on #EFE9E4 ≈ 3.4:1 (grey pills need ≥4.5:1 — darken text to #5C534C ≈ 5.5:1) · white on #4A6659 ≈ 5.9:1; dark: #4ADE9B on #123528 ≈ 8:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #F0754A on #3B1D10 ≈ 5.6:1 · #C4B8AE on #2A2420 ≈ 8:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px (mobile 48px); empty state keeps the phone fallback line

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Browse & Discover | 3 | ~15 |
| Cart & Checkout | 3 | ~15 |
| After the Order | 1 | ~5 |
| **Total** | **7** | **~35** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one at a time in order (catalog → filters → product → cart → checkout → payment → confirmation).
3. After each screen, check ruby discipline: only links and the "Ask us / Call" affordances are ruby; every Add-to-cart / Continue / Pay button must be sage #4A6659 (#A3C4B0 dark).
4. Useful follow-ups: "Restyle the prescription notice with the info colours, not ruby" · "Add the gateway-down degraded state" · "Show the Rx status pills in dark mode."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens (ruby discipline, error tokens)
- `06-prescription-pharmacy.md` — the staff review side of the Rx holds generated here
- `08-client-account-pets.md` — account dashboard that links to order history
