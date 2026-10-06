# Google Stitch Prompt — ruby-veterinary Prescription & Pharmacy Workflow

> **Purpose:** Paste this prompt into Google Stitch to generate the prescription authorisation surfaces — the owner-facing status view and the veterinarian's review tooling for ruby-veterinary.
>
> **Coverage:** 5 screens — owner Rx status view, vet review queue, prescription detail, decision modal, audit trail.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Approve = sage, Reject = error-strong #D65328 with a reason. Brand ruby never appears on pharmacy decisions.

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
- Everyday primary buttons (Approve, Save, Continue): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310` · hover `#3B5349` → `#7BA88C`
- Reject / destructive fill: `#D65328` → `#E85C30` with white text — never brand ruby
- Inputs/tables: fill `#FFFFFF` → `#1F1A16`, border `#E7DFD8` → `#3A322C`, row hover `#FAF7F5` → `#241E1A`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Rx status chips: Pending `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10` · Approved `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528` · Rejected `#B8431F` on `#FDF0E8` → `#F0754A` on `#3B1D10` · Info/queued `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning: `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces

---

## Group 1: Owner View

### Screen 1 — Owner Prescription Status View

```
Generate the prescription status view for ruby-veterinary — the owner-facing "where is my Rx order?" screen with plain-language explanation.

DESKTOP LAYOUT (max-width 860px centred column, white card on warm #FAF7F5):
- Breadcrumb: Account / Orders / RV-ORD-77123 / Prescription status
- H1 "Prescription status" (36px) + order ID "RV-ORD-77123" in JetBrains Mono 16px #5C534C
- Status chip row (large, icon + text): "Pending vet review" amber #B7791F on #FBF3E4, clock icon, 14px weight 600, pill radius 9999px
- Plain-language panel on #EBF2FA with info icon #2B6CB0:
  - Heading 18px weight 600 #1F1A17: "This is normal — nothing is wrong"
  - Body 16px/24px #1F1A17: "One item in your order is a prescription product. Dr Okoye checks it against Biscuit's file before we can send it. Most reviews finish within one business day. You haven't been charged for this item yet."
- Timeline (ordered list): Submitted Tue 14:22 → Vet review Wed 09:15 (current, amber dot with ring) → Decision expected by Thu 18:00 → Ship/pickup after approval (grey)
- Order lines mini-table: product thumb + name + variant + price; the Rx line carries a small info badge "Prescription item"; non-Rx lines badge "Shipping now"
- Actions: sage #4A6659 filled "View full order" (48px) · outlined "Need it sooner? Call the clinic" (secondary, keeps phone visible)
- Footer note with phone icon: "Questions about your prescription? Call 010 555 0199 — we're open 08:00–18:00."

Alternate chip states shown as a small legend row: "Approved" green #2F7D5A/#EAF4EF check · "Declined — we'll call you" orange #B8431F/#FDF0E8 alert icon · "Awaiting your pet's records" info #2B6CB0/#EBF2FA

MOBILE LAYOUT (primary):
- Full-width stack: H1 + order ID, status chip full-width style, info panel (icon above heading), timeline vertical with dates under labels, order lines as compact rows, stacked buttons (sage full width 56px, outlined full width), ruby phone row 48px
- Legend chips wrap 2 per line, each ≥44px with icon + text

ACCESSIBILITY:
- Status region role="status" (polite) so screen readers announce the current state once
- Every chip contains a text label plus icon — status never conveyed by colour alone
- Timeline is a semantic ol; the current step text says "In progress"
- Focus ring 3px #E0115F (dark rgba(245,61,109,0.4)) on chips-as-links, buttons, phone link
- Contrast: #B7791F on #FBF3E4 ≈ 4.7:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #B8431F on #FDF0E8 ≈ 5.4:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #1F1A17 on #FFFFFF ≈ 16:1 · white on #4A6659 ≈ 5.9:1; dark: #E3B341 on #3B2E10 ≈ 7.4:1 · #4ADE9B on #123528 ≈ 8:1 · #F0754A on #3B1D10 ≈ 5.6:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #F7F3F0 on #1F1A16 ≈ 16:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px; keyboard-only path reads status → timeline → order lines → actions

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Vet Review Tools (desktop-first)

### Screen 2 — Prescription Review Queue (sortable table)

```
Generate the prescription review queue for ruby-veterinary — the veterinarian's sortable worklist. Desktop-first back-office screen.

DESKTOP LAYOUT:
- Page header: H1 "Prescription reviews" (36px) + subline 14px #8A7F77 "Authorise or decline online orders. Median turnaround: 4 hours."
- Stat cards row (4 cards, white, 1px #E7DFD8, 12px radius): "Awaiting review 7" (amber accent bar) · "Oldest waiting 19h" (amber) · "Approved today 12" (green #2F7D5A) · "Declined today 1" (orange #B8431F)
- Toolbar: search input "Search pet, owner, or order" (360px, 40px), status select (All statuses / Pending / Awaiting records / Approved / Declined), date select, refresh icon button
- Table (white, 1px #E7DFD8 container, 12px radius, sticky header on #F6F2EF):
  - Columns: PIN (star/pin icon button, 44px) · Order (mono ID + submitted time) · Pet (name + species icon) · Owner · Items (count + first Rx line) · Assigned vet · Waiting (e.g. "19h" amber text weight 600 if >24h → use #B8431F text) · Status chip · Action ("Review" sage #4A6659 outlined button 40px)
  - Sortable column headers: chevron icon + header button, aria-sort indicated; "Waiting" currently sorted descending with visible arrow
  - Emergency-flag pin: a pinned row sits at the very top, outside the sort, with a 4px left border in ruby #9B111E, a small ruby pill "Pinned — urgent patient" (#FFF5F7 bg, #9B111E text, 12px weight 600), and a ruby "Call owner" link — the ONLY ruby on this screen
  - Row hover background #FAF7F5; zebra optional (rows white, alternate #FAF7F5)
- Pagination footer: "1–20 of 47" + prev/next buttons (40px)
- Empty queue variant: sage-50 #F4F8F5 panel, check icon, H2 "Queue clear", body "No prescriptions are waiting. New orders arrive here automatically."

MOBILE LAYOUT (mobile note, not primary):
- Back-office is desktop-first; provide a compact responsive note: table collapses to stacked cards (order ID + pet on line 1, status + waiting on line 2, "Review" full-width button), pinned emergency item stays first, search full width, stat cards 2×2 grid

ACCESSIBILITY:
- Table uses semantic thead/tbody/th with scope; sortable headers are buttons with aria-sort and a text direction cue (not chevron alone)
- Pinned emergency row is marked with an accessible label ("Pinned urgent prescription") — ruby border is decorative reinforcement
- Status chips: icon + text always; contrast #B7791F on #FBF3E4 ≈ 4.7:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #B8431F on #FFFFFF ≈ 5.5:1 · #9B111E on #FFF5F7 ≈ 7.9:1 (pin pill only) · #1F1A17 on #FFFFFF ≈ 16:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (metadata only, not for statuses); dark: #E3B341 on #3B2E10 ≈ 7.4:1 · #4ADE9B on #123528 ≈ 8:1 · #F0754A on #171310 ≈ 6.4:1 · #F53D6D on #1F1A16 ≈ 5.4:1 · #C4B8AE on #1F1A16 ≈ 9:1
- Full keyboard: tab through toolbar → rows → actions; focus ring 3px #E0115F on every control; no click-only rows (whole-row click must also be reachable by a focusable Review button)
- Tap/click targets ≥40px desktop, ≥44px on the collapsed mobile cards

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 3 — Prescription Detail (review view)

```
Generate the prescription detail view for ruby-veterinary — everything the vet needs in one screen before deciding.

DESKTOP LAYOUT (three-column feel: main 8 + context rail 4, on warm #FAF7F5):
- Header bar: back link "← Queue" 14px #5C534C, H1 "RV-ORD-77123" (28px) + mono, status chip "Pending review" amber pill, waiting time "Submitted 19h ago" 14px #B7791F weight 600, right-aligned action group: sage "Approve" filled (48px) and error-strong "Decline" filled #D65328 (48px, warning icon) — never ruby
- Main column, white cards (12px radius, 1px #E7DFD8, padding 24px):
  - Card "Patient": pet name "Biscuit", species/breed "Dog · Beagle · 6y", weight, microchip; primary vet line "Primary vet: Dr Amara Okoye"; a sage outlined "Open pet record" button
  - Card "Owner": name, phone (ruby #9B111E tel link), email; "Preferred contact: Phone"
  - Card "Order lines": table with thumb, product, variant, qty, price — Rx line badge info "Prescription item" (#EBF2FA/#2B6CB0), non-Rx lines "General" grey pill; a repeat-line note "Auto-refill: monthly"
  - Card "Uploaded history": rows with file icon, name ("Biscuit_vaccination_record.pdf · 1.2 MB · uploaded with order"), "View" outlined button (40px); empty variant: grey line "No records attached — check the pet file instead" + "Open pet file" link
  - Card "Clinical notes" (textarea, label visible, helper "Visible in the audit trail with your name")
- Right context rail (sticky): "Recent history" card — previous prescriptions for this pet with status chips and dates (e.g. "Nov 2025 — Apoquel 16mg — Approved"); alert panel #FBF3E4 with warning icon #B7791F: "Allergy noted: chicken protein" if present; clinic hours + phone footer

MOBILE LAYOUT (mobile note, not primary):
- Stack: header actions become a sticky bottom bar (sage "Approve" 50% / #D65328 "Decline" 50%, both 56px), cards full width, context rail moves to the bottom, history rows full width with chips on their own line

ACCESSIBILITY:
- Approve and Decline are visually distinct AND labelled with text; decline uses the error tokens with an icon — decision state never signalled by colour alone
- All cards are sections with headings (H2 within page H1); tables have scoped headers
- Focus ring 3px #E0115F; action group reachable early in tab order after the page heading (skip link to actions)
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FAF7F5 ≈ 7.1:1 · white on #4A6659 ≈ 5.9:1 · white on #D65328 ≈ 4.6:1 (large button text, acceptable; pair with icon + label) · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #B7791F on #FFFFFF ≈ 4.6:1; dark: #A3C4B0 fill with #171310 ≈ 9.4:1 · white on #E85C30 ≈ 4.5:1 (large text) · #6BA3D6 on #15273A ≈ 5.5:1 · #E3B341 on #3B2E10 ≈ 7.4:1
- Files viewable by keyboard; textarea labelled; ≥40px targets (44px mobile)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — Decision Modal (approve vs reject)

```
Generate the prescription decision modal for ruby-veterinary — the confirm step for approving or declining an order. Show both variants side by side on the canvas.

VARIANT A — APPROVE (calm, sage confirm):
- Modal centred, max-width 480px, white #FFFFFF (dark #241E1A), 16px radius, shadow lg, backdrop rgba(0,0,0,0.45), z-index modal 500
- Header: check-circle icon 24px in #2F7D5A, H2 "Approve this prescription?" 22px #1F1A17, close × (44px)
- Body 16px/24px #1F1A17: "Biscuit (Dog, 6y) — Hill's Prescription Diet k/d 2 kg, ×1. Approving releases the order for fulfilment and notifies Naledi Dlamini by email."
- Optional field: "Note to the owner (optional)" textarea, helper "Plain language — the owner will read this."
- Actions: right-aligned — outlined "Cancel" (44px) + sage #4A6659 filled "Approve order" (48px, white text, check icon)

VARIANT B — DECLINE (destructive uses error tokens, NEVER ruby):
- Same shell; header: alert-triangle icon 24px in #D65328, H2 "Decline this prescription?" 22px #1F1A17
- Body 16px/24px #1F1A17: "The order line won't ship and the owner isn't charged for it. Naledi will see your reason and we'll call if they need an alternative."
- Required reason fields (labelled "(required)"):
  - Radio group "Reason": Not clinically appropriate · Dose/strength needs a consult · Records insufficient · Duplicate of recent script · Other
  - Conditional textarea "Tell the owner why (required)" helper "Plain language. This is shown to the owner verbatim."
  - Checkbox "Offer a callback" (default checked)
- Error preview state: if submitted empty — field border #B8431F, error icon + text "Choose a reason and tell the owner why." on #FDF0E8, error summary at top of modal
- Actions: outlined "Cancel" + destructive filled "Decline order" — fill #D65328 (dark #E85C30), white text, alert icon (48px)
- Small footnote 13px #8A7F77: "Every decision is recorded in the audit trail with your name and the time."

MOBILE LAYOUT:
- Both variants become bottom sheets (radius 16px top, full width, sticky footer actions)
- Actions stack full width: primary action on top (56px), Cancel below outlined (52px)
- Radios full-width rows ≥48px; textarea full width

ACCESSIBILITY:
- Modal: role="dialog" aria-modal="true", labelled by its H2; focus moves in on open, is trapped, Esc cancels, focus returns to the trigger button
- Decline button text + icon make the destructive action explicit; ruby is never used
- Error state in role="alert", receives focus after failed submit; per-field messages link back
- Focus ring 3px #E0115F on all controls; contrast: white on #4A6659 ≈ 5.9:1 · white on #D65328 ≈ 4.6:1 (large text; always paired with icon + label) · #B8431F on #FDF0E8 ≈ 5.4:1 · #1F1A17 on #FFFFFF ≈ 16:1 · #2F7D5A on #FFFFFF ≈ 4.9:1; dark: #A3C4B0 fill with #171310 ≈ 9.4:1 · white on #E85C30 ≈ 4.5:1 (large) · #F0754A on #3B1D10 ≈ 5.6:1 · #F7F3F0 on #241E1A ≈ 15:1 · #4ADE9B on #241E1A ≈ 9:1
- Tap targets ≥44px (56px primary on mobile); keyboard-only completion of both variants

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 3: Accountability

### Screen 5 — Audit Trail View

```
Generate the audit trail view for ruby-veterinary — the append-only history of every action taken on a prescription order.

DESKTOP LAYOUT (max-width 900px content column, white background):
- Breadcrumb: Prescriptions / RV-ORD-77123 / Audit trail
- H1 "Audit trail" (36px) + subline 14px #8A7F77 "Append-only. Entries can't be edited or deleted — including by administrators."
- Mono order header strip on #F6F2EF: "RV-ORD-77123 · Biscuit (Dog) · Owner: Naledi Dlamini · Created 14 Mar 2026 14:22" 13px JetBrains Mono #5C534C
- Timeline list (semantic ol, vertical 2px line #E7DFD8, dots 12px):
  - Each entry row: dot colour by type — neutral #8A7F77 (system), amber #B7791F (pending/waiting), green #2F7D5A (approved), orange #B8431F (declined), blue #2B6CB0 (owner action); ALWAYS paired with a type text label
  - Entry content: action line 15px weight 600 #1F1A17 ("Order submitted by owner" · "System flagged 1 prescription item" · "Assigned to Dr Amara Okoye" · "History file uploaded: Biscuit_vaccination_record.pdf" · "Approved by Dr Amara Okoye" · "Owner notified by email")
  - Sub-line 13px #8A7F77: "14 Mar 2026, 14:22 SAST · web · IP 197.88.x.x" and for decisions a quoted note "Note: Continue current dose, recheck in 6 months."
  - The most recent entry sits at top with a subtle current marker (2px ring in its type colour)
- Filter row: select "All events" / "Decisions only" / "Owner actions" + "Download CSV" outlined button
- Footer note: "Questions about an entry? Call 010 555 0199" (ruby #9B111E link)

MOBILE LAYOUT (mobile note, not primary):
- Stack: header strip wraps to two lines, timeline full width with dot on the left and content indented, entries' metadata wraps under the action, filter select full width, download button full width, phone row ≥48px

ACCESSIBILITY:
- Ordered list semantics; entries readable in linear order; timestamps are real text
- Event type conveyed by text label first, colour second — dots are decorative (aria-hidden)
- Focus ring 3px #E0115F on filters, download, and phone link; keyboard-scrollable timeline without traps
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #F6F2EF ≈ 6.8:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (timestamps — acceptable only as supplementary text; raise to #5C534C if load-bearing) · #2F7D5A dot/text on white ≈ 4.9:1 · #B7791F on white ≈ 4.6:1 · #B8431F on white ≈ 5.5:1 · #2B6CB0 on white ≈ 5.9:1; dark: #F7F3F0 on #171310 ≈ 17:1 · #C4B8AE ≈ 10:1 · #94887D ≈ 4.6:1 · #4ADE9B ≈ 9:1 · #E3B341 ≈ 9.9:1 · #F0754A ≈ 6.4:1 · #6BA3D6 ≈ 6.5:1
- Tap targets ≥44px (desktop ≥40px controls); no hover-only expansion of entries

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Owner View | 1 | ~5 |
| Vet Review Tools | 3 | ~15 |
| Accountability | 1 | ~5 |
| **Total** | **5** | **~25** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one at a time.
3. After the decision modal, verify: Approve = sage #4A6659, Decline = #D65328. If Stitch made decline ruby, correct it: "Decline must use #D65328 with white text and an alert icon — never ruby #9B111E."
4. Useful follow-ups: "Pin an emergency-flagged prescription at the top of the queue" · "Add the decline validation error state" · "Apply the dark mode mapping to the table."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens (error vs ruby rule)
- `05-store-checkout.md` — owner-facing counterpart of these Rx states
- `09-staff-backoffice.md` — prescription queue shortcut card linking into this flow
