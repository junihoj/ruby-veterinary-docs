# Google Stitch Prompt — ruby-veterinary Emergency Homepage

> **Purpose:** Paste this prompt into Google Stitch to generate the emergency-first public homepage surfaces for the ruby-veterinary clinic website.
>
> **Coverage:** 5 screens — homepage hero, emergency banner strip, sticky mobile tap-to-call bar, hours + location section, services teaser cards.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Mobile is primary here: the Emergency Seeker arrives on a phone.

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
- Everyday primary buttons (Book appointment, Submit, Continue): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310` · hover `#3B5349` → `#7BA88C`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning: `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces

---

## Group 1: Homepage Core

### Screen 1 — Homepage Hero with Emergency CTAs

```
Generate the homepage hero for ruby-veterinary — a single veterinary clinic's emergency-first landing view. This is the 3-second clarity screen: an owner in distress must see the clinic phone, hours, and address without scrolling.

DESKTOP LAYOUT:
- Top utility strip (full width, warm background #FAF7F5, border-bottom #E7DFD8): phone icon + "Call us: 010 555 0199" as a ruby #9B111E link, "Open today 08:00–18:00", and street address "14 Maple Street, Rosebank" — all in the first strip, 14px text
- Main header (white #FFFFFF, sticky, z-index 110): text logo "ruby-veterinary" left (ruby #9B111E dot/mark accent), nav links Services · Our Team · Blog · Store · Contact in #1F1A17, light/dark theme toggle icon right, sage outlined "Book appointment" button
- Hero band on warm background #FAF7F5 (NOT a ruby background, no full-width ruby fills), two columns:
  - Left column: H1 "Expert care for the animals you love" (36–48px, #1F1A17), supporting line "A calm, experienced team — and a real person on the phone when it matters." (18px, #5C534C)
  - Left column CTA row: ruby #9B111E filled "Call now — 010 555 0199" button with phone icon (48px height, white text) PLUS sage #4A6659 filled "Book appointment" button (48px, white text) — two buttons side by side, ruby first
  - Right column: clinic photo placeholder (rounded 12px, alt text noted), with a small white card overlapping it showing "Open now · Closes 18:00" with a green #2F7D5A dot and the address
- Hours snippet row below hero (white background): "Today 08:00–18:00 · Sat 09:00–13:00 · Sun closed (emergency line available)" plus "14 Maple Street, Rosebank" with map-pin icon
- Trust row: three small sage #F4F8F5 chips — "AAHA accredited", "Fear Free certified", "Same-week appointments"

MOBILE LAYOUT (primary):
- Compact header: logo left, phone icon button right (ruby #9B111E icon), hamburger menu
- Emergency strip directly under header: ruby phone icon + tappable number text (16px minimum)
- Hero stacks: H1, supporting line, then full-width buttons stacked — ruby "Call now" first (56px height), sage "Book appointment" second (56px), 48px+ tap targets
- Clinic photo below buttons (full width, 12px radius)
- Hours + address card visible in the first scroll — no accordion, no menu tap
- Sticky bottom tap-to-call bar reserved (space it out so Screen 3's bar doesn't cover content)

ACCESSIBILITY:
- Phone numbers are tel: links, keyboard focusable, with visible 3px #E0115F focus ring (rgba(245,61,109,0.4) on dark)
- Ruby "Call now" and sage "Book appointment" both have visible text labels — never icon-only
- Alt text on the clinic photo ("Veterinarian examining a dog at ruby-veterinary clinic")
- Contrast: white on #9B111E ≈ 8.4:1, white on #4A6659 ≈ 5.9:1, #1F1A17 on #FFFFFF ≈ 16:1 — all pass AA
- Tap targets ≥44px; logical tab order: utility strip → header nav → CTAs → hours snippet

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 2 — Emergency Banner Strip (always visible)

```
Generate the always-visible emergency banner strip for ruby-veterinary — persistent chrome that sits on every public page, above or below the header, never dismissible.

DESKTOP LAYOUT:
- Full-width horizontal strip, height ~44px, background ruby-soft #FFF5F7 with a 4px left border in ruby #9B111E (dark mode: #1F1A16 background with #F53D6D left border)
- Left: alert/phone icon (24px, #9B111E) + bold text "Emergency? Call now: 010 555 0199" — the number is a tel: link in #9B111E, 16px, weight 600
- Centre/right secondary line (14px, #5C534C): "If this is life-threatening, go to the nearest emergency veterinary hospital. We're open 08:00–18:00 daily."
- Right: small sage outlined " directions" link to the address, and an "Out of hours" text link
- Strip is sticky with the header (z-index 110), no close/dismiss button anywhere

MOBILE LAYOUT (primary):
- Two-row strip, full width, no horizontal scrolling
- Row 1: phone icon + "Emergency? Call now: 010 555 0199" (tap-to-call, full-width tappable, 48px tall row)
- Row 2: wrapped 14px line "Life-threatening? Go to the nearest emergency vet." plus "Open until 18:00"
- Nothing truncated; the number is never hidden behind an icon

ACCESSIBILITY:
- Strip is landmark content (role="region", accessible name "Emergency contact"), reachable by keyboard in the first few tab stops
- Contrast: #9B111E on #FFF5F7 ≈ 7.9:1 (AA); #5C534C on #FFF5F7 ≈ 6.5:1 (AA); dark mode #F53D6D on #1F1A16 ≈ 5.4:1
- Alert icon is decorative (aria-hidden) — the text carries the meaning, never colour or icon alone
- Tap target for the phone link spans the full text width, ≥44px tall
- No animation that flashes; respect prefers-reduced-motion

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Persistent Contact + Discovery

### Screen 3 — Sticky Mobile Tap-to-Call Bar

```
Generate the sticky mobile tap-to-call bar for ruby-veterinary — the persistent bottom bar that keeps the clinic phone one thumb-tap away while an owner scrolls any public page.

MOBILE LAYOUT (primary):
- Fixed bottom bar, height 64px + safe-area inset padding, background #FFFFFF with top border #E7DFD8 and shadow md (0 4px 12px rgba(31,26,23,.10)), z-index sticky 100
- Left ~65% of bar: ruby #9B111E filled button, full height, phone icon + two lines of text: "Call clinic" (bold, white) and "010 555 0199" (14px, white, always visible number)
- Right ~35%: sage #4A6659 filled button "Book" with calendar icon, white text
- Optional thin status pill above the buttons inside the bar: green dot + "Open · closes 18:00" (12px, #2F7D5A on #EAF4EF background, pill radius 9999px)
- Bar appears after ~200px of scroll (shown in its visible state in the mockup); page content gets 80px bottom padding so nothing is covered

DESKTOP LAYOUT:
- No bottom bar on desktop. Instead the header utility strip keeps phone + hours visible (reference the header from Screen 1). Show a small annotation: "Desktop: phone stays in the sticky header utility strip"
- Optionally show the bar as a compact floating pill bottom-right: ruby "Call 010 555 0199" only

ACCESSIBILITY:
- Both buttons keyboard focusable with visible 3px focus ring (#E0115F light / rgba(245,61,109,0.4) dark); focus must not be hidden behind the bar
- Each button ≥44px tall; entire left button is one tel: link with an accessible name containing the number
- White on #9B111E ≈ 8.4:1; white on #4A6659 ≈ 5.9:1 — AA in both modes
- Bar must not trap focus or cover form submit buttons; respects safe-area-inset-bottom
- "Open/closes" status uses a dot PLUS text — never colour alone

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — Hours + Location + Map Section

```
Generate the hours and location section for the ruby-veterinary homepage — the block that proves "we're here and you can find us" within the 3-second rule.

DESKTOP LAYOUT:
- Two-column section on warm background #FAF7F5, rounded 12px inner card, max width 1280px:
  - Left column: H2 "Hours & location" (28px, #1F1A17)
    - Weekly hours table: Monday–Friday 08:00–18:00, Saturday 09:00–13:00, Sunday Closed — with a "Today" row highlighted using sage-50 #F4F8F5 background and a green #2F7D5A "Open now" pill (or grey "Closed now" pill using #8A7F77 on #EFE9E4)
    - Address block: "14 Maple Street, Rosebank, 2196" with map-pin icon, plus "Free parking on Maple Street" helper line (14px, #5C534C)
    - Phone block: phone icon + "010 555 0199" as a ruby #9B111E link, 22px, weight 600
    - Buttons: sage #4A6659 filled "Get directions" (8px radius) and outlined secondary "Call the clinic"
  - Right column: map placeholder rectangle (12px radius, label "Interactive map — 14 Maple Street"), caption below: "Emergency after hours: 24/7 partner line 010 555 0100" in 14px with ruby link
- Out-of-hours note: small info panel on #EBF2FA with info icon #2B6CB0: "Outside these hours, call and listen for the emergency instruction, or go straight to the nearest emergency hospital."

MOBILE LAYOUT (primary):
- Single column, stacked: H2, "Open now" pill + today's hours, full hours list (no accordion), address (tappable to maps), big ruby phone link, full-width sage "Get directions" button, map placeholder (4:3 ratio)
- Emergency partner line visible without scrolling past the map
- All rows ≥44px tall

ACCESSIBILITY:
- Hours table uses real table markup semantics with header cells; "Today" is marked with text, not only background colour
- Focus ring visible on phone, directions, and map links; address is selectable text (not an image)
- Contrast: #1F1A17 on #FAF7F5 ≈ 15:1; #2F7D5A on #EAF4EF ≈ 4.6:1; dark mode #4ADE9B on #123528 ≈ 8:1
- Tap targets ≥44px; no horizontal scroll at 360px width

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 5 — Services Teaser Cards

```
Generate the services teaser card section for the ruby-veterinary homepage — a preview grid that links into the full service catalog.

DESKTOP LAYOUT:
- Section on white #FFFFFF with H2 "How we care for your pet" (28px) and a right-aligned ruby #9B111E text link "View all services →"
- 3-column card grid (2 rows), each card: background #FFFFFF, border 1px #E7DFD8, radius 12px, shadow sm, padding 24px
- Six cards: Wellness & Vaccinations · Surgery · Dental Care · Emergency & Urgent Care · Therapeutic Diets · Pet Supplies & Nutrition
- Each card: 24px stroke icon in sage #4A6659 (the Emergency card icon instead in ruby #9B111E), H3 card title (22px, #1F1A17), one-line description (14px, #5C534C), baseline price hint "From $45" (14px, weight 600, #3B5349), and a link "View service →" (#9B111E)
- Only the Emergency & Urgent Care card may use a ruby accent border-left (4px #9B111E) and a ruby "Call now" mini button — all other cards stay neutral/sage
- Below grid: sage #F4F8F5 panel with "Book an appointment" heading, plain-language line, and a sage #4A6659 filled "Book appointment" button (48px)

MOBILE LAYOUT (primary):
- Single-column stacked cards, full width, 16px page padding
- Card layout: icon + title row, description, price, full-width link row (48px tap target)
- Emergency card pinned FIRST in the stack on mobile, with ruby border-left and ruby "Call now" button
- Book panel full-width; sage button full-width, 56px height
- "View all services →" link sits directly under the H2, not hidden at the bottom

ACCESSIBILITY:
- Each card is a focusable link target with one accessible name (avoid nested links; use a single stretched link or separate clearly labelled controls)
- Focus ring 3px #E0115F visible on every card link
- Icons decorative (aria-hidden); price and status conveyed as text
- Contrast: #3B5349 on #FFFFFF ≈ 7.2:1; #5C534C on #FFFFFF ≈ 7.4:1; #9B111E on #FFFFFF ≈ 8.4:1; dark equivalents #A3C4B0 / #C4B8AE / #F53D6D on #1F1A16 all ≥4.5:1
- Tap targets ≥44px; cards do not rely on hover to reveal information

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Homepage Core | 2 | ~10 |
| Persistent Contact + Discovery | 3 | ~15 |
| **Total** | **5** | **~25** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate Screen 1.
3. Generate screens one at a time in the order listed; save a screenshot after each.
4. Iterate with one change per prompt, e.g. "Make the ruby Call now button 56px tall on mobile" or "Apply the dark mode mapping to this screen."
5. Connect Screens 1 → 4 → 5 on the canvas, then Screens 2 → 3 as persistent chrome variants.

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens
- `../patterns-emergency-first.md` — 3-second rule layout patterns
- `02-services-staff-clinic.md` — next flow in generation order
