# Google Stitch Prompt — ruby-veterinary Staff Back Office

> **Purpose:** Paste this prompt into Google Stitch to generate the role-scoped staff back office for ruby-veterinary — login, alerts, catalogue, content, and prescription shortcuts.
>
> **Coverage:** 7 screens — staff login, admin alerts dashboard, catalog product table, product editor, article editor, prescription queue shortcut card, shared inbox embedded view note.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Desktop-first layouts are fine here; every screen still carries a mobile note and a light+dark pair.

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
- App shell sidebar/chrome: `#F6F2EF` → `#1F1A16` · active nav item `#E3EDE6` → `#2A2420` with text `#3B5349` → `#A3C4B0`, active left border `#4A6659` → `#A3C4B0`
- Cards/tables: `#FFFFFF` → `#1F1A16` · table header `#F6F2EF` → `#2A2420` · row hover `#FAF7F5` → `#241E1A` · subtle fills `#EFE9E4` → `#2A2420`
- Elevated (modals, dropdowns): `#FFFFFF` → `#241E1A`
- Borders: `#E7DFD8` → `#3A322C` · stronger `#D4C8BE` → `#4C423A`
- Text: primary `#1F1A17` → `#F7F3F0` · secondary `#5C534C` → `#C4B8AE` · tertiary `#8A7F77` → `#94887D` · inverse `#FFFFFF` → `#171310`
- Links: `#9B111E` → `#F53D6D`
- Emergency ruby CTAs ("Call now"): fill `#9B111E` → `#F53D6D`, white text in both modes
- Everyday primary buttons (Save, Publish, Sign in, Continue): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310` · hover `#3B5349` → `#7BA88C`
- Destructive (delete, unpublish confirm): fill `#D65328` → `#E85C30` with white text — never ruby
- Inputs/selects/textareas: fill `#FFFFFF` → `#1F1A16`, border `#E7DFD8` → `#3A322C`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning: `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Unread count badges: `#9B111E` fill white text → `#F53D6D` fill `#171310` text (alerts only)
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces

---

## Group 1: Access & Triage

### Screen 1 — Staff Login

```
Generate the staff login page for ruby-veterinary — role-scoped back-office entry (staff accounts only; no public registration).

DESKTOP LAYOUT:
- Split layout: left panel on warm #FAF7F5 (60%) with clinic branding — text logo "ruby-veterinary · back office", H1 "Staff sign-in" (36px), subline 16px #5C534C "Internal use only. Every action here is logged against your name."
  - Security notes list with 20px sage icons: "HTTPS everywhere", "Role-based access — reception, veterinary, and manager permissions", "Sessions expire after 30 minutes idle"
  - Persistent emergency line at panel bottom: ruby #9B111E phone link "Clinic line: 010 555 0199" (staff can always reach the floor)
- Right card (white, 1px #E7DFD8, 12px radius, shadow md, padding 32px, max-width 400px):
  - H2 "Sign in" 22px, fields: Staff email (44px), Password with show/hide toggle (44px), "Remember this device" checkbox
  - Sage #4A6659 filled "Sign in" (48px, full width)
  - Text links: "Forgot password?" #9B111E · "Need access? Ask the practice manager." 13px #8A7F77
  - Error state: panel #FDF0E8, error icon + text #B8431F "Email or password is incorrect. After 5 attempts, sign-in pauses for 15 minutes." + per-field border #B8431F
  - Locked state variant: amber panel #FBF3E4, warning icon #B7791F, "Too many attempts. Try again in 14:32 or call the practice manager."

MOBILE LAYOUT (mobile note):
- Single column: logo, H1, card full width 16px padding, fields 48px, sign-in full width 56px sage, links stacked, emergency line full-width 48px tap row at the bottom

ACCESSIBILITY:
- Form fields labelled persistently; autocomplete="username"/"current-password"; show/hide toggle is a labelled button announcing state
- Error/locked messages in role="alert" and receive focus after failed submit; icon + text, never colour alone
- Focus ring 3px #E0115F; no captcha-only or colour-only affordances
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FAF7F5 ≈ 7.1:1 · white on #4A6659 ≈ 5.9:1 · #B8431F on #FDF0E8 ≈ 5.4:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #9B111E on #FAF7F5 ≈ 8.0:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #F0754A on #3B1D10 ≈ 5.6:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #F53D6D on #171310 ≈ 6.2:1
- Tap targets ≥44px (48–56px mobile); keyboard-only sign-in path

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 2 — Admin Alerts Dashboard

```
Generate the admin alerts dashboard for ruby-veterinary — form submissions and bot escalations in one triage list with unread counts.

DESKTOP LAYOUT (back-office shell: 240px sidebar + content, max-width 1200px):
- Sidebar nav: Dashboard (active), Alerts, Catalog, Content, Prescriptions, Inbox, Staff, Settings — active item #E3EDE6/#3B5349 with 4px left border #4A6659
- Content header: H1 "Alerts" (36px) + subline "New form submissions and WhatsApp escalations. Clear the queue, don't let anything go unanswered." + right: sage outlined "Mark all read" (40px) + filter select
- Stat cards row (4): "Unread 12" (unread badge uses ruby #9B111E fill, white text) · "Bot escalations 3" (amber) · "Appointment requests 5" (info) · "Avg first response 8m" (green)
- Alert list (white card, 1px #E7DFD8):
  - Grouped by source with section headers 12px uppercase #8A7F77: "ESCALATIONS FROM THE BOT" · "FORM SUBMISSIONS"
  - Escalation rows (unread): 4px left border ruby #9B111E, unread dot ruby, source chip "WhatsApp · bot" (#EBF2FA/#2B6CB0), title 15px weight 600 #1F1A17 ("Emergency keyword: 'bleeding' — Naledi Dlamini"), snippet 13px #5C534C, timestamp 12px #8A7F77, right: ruby text link "Open in inbox →" (escalation link only) + unread count badge
  - Form rows (unread): 4px left border #4A6659 (sage), source chip "Appointment request", title "New request — Biscuit (dog), urgent today", meta "Received 6m ago · routed to reception@", actions: outlined "Open" (40px) + "Mark read" text
  - Read rows: no left border, grey unread state, muted title #5C534C, source chip stays
  - Priority sort: emergency escalations pinned to the top of the list
- Empty state: sage-50 #F4F8F5 panel, check icon, H2 "You're all caught up", body "New submissions and escalations land here the moment they arrive."

MOBILE LAYOUT (mobile note):
- Sidebar collapses to drawer; stats 2×2; alert rows stack (source chip + time on line 1, title, snippet, actions full width 48px); unread rows keep left border; pinned escalations first; empty state full width

ACCESSIBILITY:
- List is a list of links/buttons; unread state conveyed by text ("unread") in the accessible name plus the visual dot — never colour alone
- Stat cards announce number + label as text; badges include "unread" wording
- Focus ring 3px #E0115F; keyboard reaches every row action; "Mark all read" confirms via role="status"
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (timestamps/group labels — use #5C534C if load-bearing) · white on #9B111E ≈ 8.4:1 (unread badge) · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #3B5349 on #E3EDE6 ≈ 6.4:1 · white on #4A6659 ≈ 5.9:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #2F7D5A on #EAF4EF ≈ 4.6:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE ≈ 9:1 · #94887D ≈ 4.6:1 · #F53D6D fill with #171310 ≈ 5.4:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #A3C4B0 on #2A2420 ≈ 7.6:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #4ADE9B on #123528 ≈ 8:1
- Rows ≥56px; ≥44px targets; no hover-only actions

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Catalogue & Content

### Screen 3 — Catalog Product Table

```
Generate the catalog product table for ruby-veterinary — staff manage price, stock, and the prescription toggle. Desktop-first.

DESKTOP LAYOUT (back-office shell, content max-width 1200px):
- Header: H1 "Products" (36px) + count "142 products" 14px #8A7F77 + right: sage #4A6659 filled "Add product" (44px, plus icon)
- Toolbar: search (320px), category select (All / General Pet Supplies / Therapeutic Diets / Prescription Medication), stock select, "Only Rx items" toggle switch (label visible), export CSV outlined button
- Table (white, 1px #E7DFD8, 12px radius, sticky header on #F6F2EF 44px):
  - Columns: checkbox (44px) · Product (thumb 40px + name + variant count) · SKU (JetBrains Mono 13px #8A7F77) · Category chip · Price (right-aligned, 15px weight 600, editable inline cell affordance) · Stock (number + state chip: "In stock" green, "Low (4)" amber, "Out" grey) · Rx (toggle switch with visible label "Prescription" — on = #4A6659 track) · Status (Live/Draft chips) · Actions (edit icon button, more ⋯, 44px)
  - Rows 56px, 1px #E7DFD8 bottom border, hover #FAF7F5, selected rows #F4F8F5 with checkbox ticked
  - Bulk action bar (appears when rows selected): #F4F8F5 bar, "3 selected" text, outlined "Set price" / "Adjust stock" / "Unpublish", destructive text "Delete" in #B8431F
- Pagination footer: "1–25 of 142" + page buttons (40px, active sage fill white text)
- Empty filter result: sage-50 panel, "No products match these filters" + outlined "Clear filters"

MOBILE LAYOUT (mobile note):
- Toolbar stacks; table becomes stacked cards (thumb + name + SKU line, price + stock chips line, Rx toggle row with label, actions row) — never horizontal-scroll the primary data away; sticky "Add product" FAB or full-width header button

ACCESSIBILITY:
- Semantic table with th scope; sort headers are buttons with aria-sort; select-all checkbox labelled
- Rx toggle is a labelled switch (role="switch" or checkbox) with visible text "Prescription" — state also announced; stock states carry text chips, not colour alone
- Focus ring 3px #E0115F; bulk bar announces "3 products selected" via role="status"
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C ≈ 7.4:1 · #8A7F77 ≈ 3.9:1 (SKU meta — acceptable, non-decisional) · #2F7D5A on #EAF4EF ≈ 4.6:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #5C534C on #EFE9E4 ≈ 5.4:1 · white on #4A6659 ≈ 5.9:1 · #3B5349 on #F4F8F5 ≈ 6.9:1 · #B8431F on #F4F8F5 ≈ 5.3:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE ≈ 9:1 · #94887D ≈ 4.6:1 · #4ADE9B on #123528 ≈ 8:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #C4B8AE on #2A2420 ≈ 8:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #A3C4B0 on #241E1A ≈ 8:1 · #F0754A on #241E1A ≈ 5.4:1
- Rows ≥56px; icon buttons ≥44px; keyboard completes select → bulk action

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — Product Editor (variants + facets)

```
Generate the product editor for ruby-veterinary — name, price, stock, variants, and facet assignments. Desktop-first, no-code for the practice manager.

DESKTOP LAYOUT (back-office shell, two-column editor 8 + 4):
- Header bar: back link "← Products", H2 "Edit — Heartgard Plus" (24px), status chip "Live" green pill, right: outlined "Preview" + sage #4A6659 filled "Save changes" (44px)
- Left column, white cards (12px radius, 1px #E7DFD8, padding 24px, H3 22px card titles):
  - Card "Basics": Product name, Description textarea (rich-text toolbar hint: B I H2 list link image), Category select, "Prescription item" switch with helper "Requires vet authorisation before fulfilment — checkout shows an info notice, not an error."
  - Card "Pricing & stock": Price, Compare-at (optional), Stock quantity, Low-stock threshold, SKU, barcode
  - Card "Variants" (the heart of variable products): variant rows in a mini-table — Variant (e.g. "10–25 kg"), Price, SKU, Stock, actions ⋯; "+ Add variant" outlined button; per-variant helper "Each variant can have its own price, SKU, and stock count."
  - Card "Fulfilment": radio "Ship" / "In-clinic pickup" / "Both"; weight field
- Right rail (sticky): Card "Facets" with checkbox groups — Pet type (Cat, Dog), Life stage (Puppy/Kitten, Adult, Senior), Health condition (Heartworm, Joint care, Weight management, Dental) — checked = #4A6659 fill, white check, 4px radius; Card "Media": image placeholder grid + "Upload" outlined button; Card "Audit": "Last edited by K. Patel · 14 Mar 2026 11:02" 12px #8A7F77
- Unsaved-changes bar: sticky footer #FBF3E4 with warning icon #B7791F "You have unsaved changes" + outlined "Discard" + sage "Save changes"
- Destructive variant: "Delete product" text action #B8431F → confirm modal uses #D65328 filled "Delete product" (never ruby)

MOBILE LAYOUT (mobile note):
- Single column: header actions in sticky footer (Discard / Save full width), cards full width, variants become stacked mini-cards, facet groups full-width checkbox lists (48px rows), media grid 2-up

ACCESSIBILITY:
- Every field has a visible label; switches have accessible names and announce on/off; checkbox groups in fieldsets with legends
- Unsaved-changes bar in role="status"; save confirmation toast announces "Changes saved"
- Focus ring 3px #E0115F; keyboard adds/edits a variant without drag-and-drop only (provide move up/down buttons)
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C ≈ 7.4:1 · #8A7F77 ≈ 3.9:1 (audit meta) · white on #4A6659 ≈ 5.9:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #B8431F on #FFFFFF ≈ 5.5:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE ≈ 9:1 · #94887D ≈ 4.6:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #4ADE9B on #123528 ≈ 8:1 · #F0754A on #171310 ≈ 6.4:1
- Inputs ≥44px; icon buttons ≥44px; keyboard-only edit-save loop

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 5 — Article Editor (markdown, SEO, publish)

```
Generate the article editor for ruby-veterinary — the CMS screen a vet tech uses with no developer in the loop. Desktop-first.

DESKTOP LAYOUT (back-office shell, full-width editor):
- Header bar: back "← Articles", H2 "Edit article" + autosave status 12px #8A7F77 ("Draft saved 14:22" role="status"), right: outlined "Preview" · outlined "Save draft" · sage #4A6659 filled "Publish" (44px, send icon) + dropdown caret (Publish now / Schedule)
- Layout (3 columns): left outline rail 200px · centre writing pane · right settings rail 300px
  - Centre pane (white, max-width 720px): Title input (H1 size, borderless, placeholder "Article title"), slug line "/blog/" + editable slug 13px #8A7F77 mono, then markdown/toolbar row (B · I · H2 · H3 · list · quote · link · image · video) with tooltips
    - Body in Merriweather 18px/30px #1F1A17 showing markdown with rendered feel: a paragraph, an H2 "Signs to watch for", a bullet list, an inline image placeholder with caption field, an embedded video placeholder, a blockquote on #F4F8F5 with 4px left border #4A6659, an external link #9B111E
    - Markdown source hint line (13px JetBrains Mono #8A7F77): "## Signs to watch for"
  - Left rail: document outline (H2/H3 list, clickable), word count "612 words · ~3 min read" 12px #8A7F77, "Last edited by Dr Amara Okoye"
  - Right rail, stacked cards (H4 18px titles):
    - Card "Publishing": status select (Draft/Scheduled/Published), author select (Dr Amara Okoye), publish date
    - Card "SEO": Meta title input with counter "47/60", Meta description textarea with counter "138/160", slug input, search preview snippet (title in #2B6CB0-ish blue like a result link, url #2F7D5A, description #5C534C) — neutral, not ruby
    - Card "Taxonomy": category chips (Dog Care selected), tag input with suggestion chips
    - Card "Social": preview card (image, title 15px, domain 12px)
- Scheduled state: amber pill "Scheduled — 18 Mar 08:00" #B7791F/#FBF3E4 with clock icon; error state if publish fails: #FDF0E8 panel, error icon + text #B8431F "Couldn't publish — check the meta description is under 160 characters." + phone fallback "Or call the practice manager: 010 555 0199"

MOBILE LAYOUT (mobile note):
- Settings rail becomes tabs (Content · SEO · Publishing) under the writing pane; header actions in sticky footer with "Publish" sage full width 56px; toolbar scrolls horizontally with visible edge; autosave status in header

ACCESSIBILITY:
- Textarea is a real editing surface with a labelled title input; toolbar buttons are icon+tooltip and expose pressed state (aria-pressed)
- Autosave in role="status"; publish errors in role="alert" with focus; counters announce "47 of 60 characters"
- Preview snippet uses real text; author/category selects labelled
- Focus ring 3px #E0115F; contrast: #1F1A17 Merriweather on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 ≈ 3.9:1 (hints/meta) · white on #4A6659 ≈ 5.9:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #B8431F on #FDF0E8 ≈ 5.4:1 · #9B111E links on #FFFFFF ≈ 8.4:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE ≈ 9:1 · #94887D ≈ 4.6:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #F0754A on #3B1D10 ≈ 5.6:1 · #F53D6D on #1F1A16 ≈ 5.4:1
- Inputs ≥44px; toolbar buttons ≥44px; keyboard publish without mouse

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 3: Operations Shortcuts

### Screen 6 — Prescription Queue Shortcut Card

```
Generate the prescription queue shortcut card for ruby-veterinary — a dashboard tile that deep-links vets into the review queue.

DESKTOP LAYOUT (a card designed to sit in the back-office dashboard grid, ~380px wide):
- Card: white, 1px #E7DFD8, 12px radius, shadow sm, padding 24px, top accent bar 4px in amber #B7791F (ruby #9B111E ONLY if an emergency-flagged item is pinned — show the amber default)
- Header row: prescription/bottle icon 24px sage #4A6659 + H3 "Prescription queue" 22px + right: info-style badge "For veterinarians" (#EBF2FA/#2B6CB0, 12px)
- Big number 40px weight 600 #1F1A17 "7" + label "awaiting review" 14px #5C534C
- Stats lines (14px): "Oldest waiting: 19h" with amber #B7791F weight 600 · "Approved today: 12" #2F7D5A · "Declined today: 1" #B8431F
- Mini waiting list (3 rows): pet name 14px weight 600 + order ID 12px mono #8A7F77 + status pill ("Pending" amber / "Awaiting records" info) + waiting time right-aligned
- Footer: sage #4A6659 filled "Open review queue" (44px) + text link "View audit trail →" #9B111E
- Empty variant: green check icon, "Queue clear — no prescriptions waiting" 14px #2F7D5A, body "New online orders appear here automatically." + outlined "Refresh"
- Role-restricted variant (receptionist view): card shows count only with note "Visible to veterinarians only — ask a vet to review" and disabled button (#EFE9E4 fill, #8A7F77 text, aria-disabled)

MOBILE LAYOUT (mobile note):
- Card full width; stats stack; mini list rows become two-line; "Open review queue" full width 56px; audit link centred below

ACCESSIBILITY:
- Card is a section with heading; the primary button's accessible name includes context ("Open prescription review queue, 7 awaiting review")
- Counts are text; status pills carry labels with icons — never colour alone; disabled state announces "for veterinarians only"
- Focus ring 3px #E0115F; contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C ≈ 7.4:1 · #8A7F77 ≈ 3.9:1 (mono IDs) · #B7791F on #FFFFFF ≈ 4.6:1 · #2F7D5A on #FFFFFF ≈ 4.9:1 · #B8431F on #FFFFFF ≈ 5.5:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · white on #4A6659 ≈ 5.9:1 · #8A7F77 on #EFE9E4 ≈ 3.4:1 (disabled — exempt from contrast minimums but keep the label readable; prefer #5C534C ≈ 5.4:1); dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #E3B341 on #171310 ≈ 9.9:1 · #4ADE9B on #171310 ≈ 9:1 · #F0754A on #171310 ≈ 6.4:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px (56px mobile primary)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 7 — Shared Inbox Embedded View (note)

```
Generate the shared inbox embedded view for ruby-veterinary — the back office's embedded WhatsApp inbox frame (integration note screen). This is an annotation/mock of a third-party multi-agent inbox embedded in our admin shell.

DESKTOP LAYOUT (back-office shell with "Inbox" active in the sidebar):
- Shell frame: sidebar + top bar ("Shared inbox" H2 24px + channel chip "WhatsApp Business · verified" #EBF2FA/#2B6CB0 + agent chip "K. Patel · Available" + "Open in new tab ↗" outlined button)
- Embedded view area: a bordered container (2px dashed #D4C8BE, 12px radius, background #FAF7F5) representing the third-party iframe, containing a realistic simplified inbox: left conversation list (240px) + centre timeline + right details panel
  - Conversation rows: avatar, name, snippet, claim chip ("Unclaimed" grey / "Claimed · J. Mbeki" green / "Emergency" ruby pill with 4px ruby left border on the pinned row), unread badge (ruby #9B111E fill, white text)
  - Timeline: white/#FFFFFF bot bubbles, owner bubbles, agent bubbles #EBF2FA, system chips ("Claimed", "Bot paused")
  - Composer: input + sage #4A6659 "Send" button
- Annotation panel (outside or below the container, clearly a design note): #EBF2FA with info icon #2B6CB0, title "Integration note", body 14px #1F1A17: "This pane is an embedded multi-agent WhatsApp inbox (ManyChat / Sirena / WhatsApp Business App). Our shell provides navigation, staff identity, and the emergency strip; the vendor provides conversation state, claiming, and delivery receipts. Fallback: if the embed fails to load, show the phone fallback card below."
- Fallback card (shown as a second state): #FFF5F7 with 4px #9B111E left border, alert icon, "Inbox unavailable right now" 16px weight 600, body "Messages still reach the clinic — call 010 555 0199 or email hello@rubyvets.example. Staff: check the WhatsApp Business App directly." + ruby "Call 010 555 0199" button (44px)

MOBILE LAYOUT (mobile note):
- Embed is desktop-primary; on mobile show the fallback-first layout: shell header, annotation collapsed into an accordion "About this view", and the conversation list full width (deep-linking to the vendor's own mobile app if available), with the emergency strip persistent

ACCESSIBILITY:
- Embedded frame has a title (title attribute / aria-label "Shared WhatsApp inbox") so screen readers announce the boundary
- Annotation panel is a complementary note, not mixed into conversation semantics; fallback card in role="alert" when shown
- Internal mock bubbles are static text — mark decorative previews appropriately so they aren't mistaken for live content
- Focus ring 3px #E0115F; contrast: #1F1A17 on #FAF7F5 ≈ 15:1 · #1F1A17 on #EBF2FA ≈ 15:1 · white on #4A6659 ≈ 5.9:1 · white on #9B111E ≈ 8.4:1 · #9B111E on #FFF5F7 ≈ 7.9:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #5C534C on #EFE9E4 ≈ 5.4:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #F7F3F0 on #15273A ≈ 14:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #F53D6D fill with #171310 ≈ 5.4:1 · #F53D6D on #1F1A16 ≈ 5.4:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #C4B8AE on #2A2420 ≈ 8:1
- Tap targets ≥44px; "Open in new tab" announces its behaviour

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Access & Triage | 2 | ~10 |
| Catalogue & Content | 3 | ~15 |
| Operations Shortcuts | 2 | ~10 |
| **Total** | **7** | **~35** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one at a time.
3. Check after each: Save/Publish/Sign in/Open queue are sage #4A6659 (#A3C4B0 dark); delete/unpublish confirms are #D65328; ruby only for call links and alert unread badges.
4. Useful follow-ups: "Add the unsaved-changes bar" · "Show the receptionist role-restricted variant of the Rx card" · "Apply the dark mode mapping to the table."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens
- `06-prescription-pharmacy.md` — the full review queue this card links into
- `07-whatsapp-messaging.md` — inbox conversation surfaces behind the embed
- `03-blog-newsletter.md` — the published output of the article editor
