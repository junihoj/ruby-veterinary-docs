# Google Stitch Prompt — ruby-veterinary Client Account & Pets

> **Purpose:** Paste this prompt into Google Stitch to generate the logged-in client account area for ruby-veterinary — dashboard, pets, and subscriptions.
>
> **Coverage:** 4 screens — account dashboard, pets list, pet detail with history uploads, subscriptions management.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Rx-linked subscription changes carry a warning (icon + text), never ruby.

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
- Everyday primary buttons (Book, Add pet, Save changes, Continue): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310` · hover `#3B5349` → `#7BA88C`
- Destructive (Cancel subscription confirm): fill `#D65328` → `#E85C30` with white text — never ruby
- Inputs/selects: fill `#FFFFFF` → `#1F1A16`, border `#E7DFD8` → `#3A322C`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning (Rx-linked pause notices): `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces

---

## Group 1: Account Overview

### Screen 1 — Account Dashboard

```
Generate the account dashboard for ruby-veterinary — the client's home after signing in.

DESKTOP LAYOUT:
- App shell: left sidebar 240px (Dashboard · My pets · Orders · Subscriptions · Appointment requests · Documents · Settings) — active item sage-100 #E3EDE6 background with #3B5349 text and 4px left border #4A6659; inactive #5C534C on white; footer of sidebar: "Need help?" ruby #9B111E phone link "010 555 0199"
- Main content on warm #FAF7F5, max-width 1080px:
  - Greeting header: H1 "Hello, Naledi" (36px) + subline 16px #5C534C "Here's what's happening with your pets and orders." + right-aligned sage #4A6659 filled "Book appointment" (44px)
  - Summary card row (4 cards, white, 1px #E7DFD8, 12px radius, padding 20px):
    - "Orders": big number 28px weight 600, subline "1 awaiting vet approval" amber #B7791F with clock icon, link "View orders →" #9B111E
    - "Pets": big number, subline "Biscuit · Momo", link "Manage pets →"
    - "Subscriptions": big number, subline "Next charge 1 Apr · $46.80", link "Manage →"
    - "Appointment requests": big number, subline "1 in review" info #2B6CB0, link "View requests →"
  - Two-column lower area:
    - Card "Recent activity" (white, list): rows with icon + text 14px — "Rx order RV-ORD-77123 pending review" (amber chip) · "Appointment request sent Tue" (info chip) · "History file uploaded — Biscuit" (green chip); footer link "See all activity →"
    - Card "Next appointment" (sage-50 #F4F8F5): "Thu 19 Mar · 10:30 · Biscuit — dental check" 16px weight 600, with "Add to calendar" outlined and "Reschedule" text link; empty variant: "No appointments booked" + sage "Book appointment"
  - Emergency card (small, always present): #FFF5F7 bg, 4px left border #9B111E, "Emergency outside hours? Call 010 555 0199" ruby link + "nearest ER" link

MOBILE LAYOUT (primary):
- Sidebar becomes a hamburger drawer; top bar: menu icon, "My account" H2, avatar
- Greeting stacks; sage "Book appointment" full width 56px
- Summary cards: 2×2 grid, each ≥140px tall, number + subline + link
- Recent activity full width; next appointment card full width; emergency card full width with 48px tap row
- Bottom sticky nav from the design system applies (Call / Book / Store / Account)

ACCESSIBILITY:
- Sidebar is a nav landmark with aria-current="page" on the active item; drawer traps focus when open on mobile
- Summary numbers are text; sublines carry status as words ("pending", "in review") plus icons — never colour alone
- Focus ring 3px #E0115F (dark rgba(245,61,109,0.4)); logical order: skip link → greeting → book → cards → activity → emergency
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FAF7F5 ≈ 7.1:1 · #3B5349 on #E3EDE6 ≈ 6.4:1 · white on #4A6659 ≈ 5.9:1 · #B7791F on #FFFFFF ≈ 4.6:1 · #2B6CB0 on #FFFFFF ≈ 5.9:1 · #9B111E on #FFF5F7 ≈ 7.9:1 · #3B5349 on #F4F8F5 ≈ 6.9:1; dark: #F7F3F0 on #171310 ≈ 17:1 · #C4B8AE on #1F1A16 ≈ 9:1 · #A3C4B0 on #1F1A16 ≈ 8.5:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #E3B341 on #171310 ≈ 9.9:1 · #6BA3D6 on #171310 ≈ 6.2:1 · #F53D6D on #1F1A16 ≈ 5.4:1
- Tap targets ≥44px (56px primary on mobile); cards are links or contain one clear link each

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Pets

### Screen 2 — Pets List

```
Generate the pets list for ruby-veterinary — every pet on the client's file.

DESKTOP LAYOUT (content area, same sidebar shell):
- Header row: H1 "My pets" (36px) + subline "Each pet has its own record with our vets." + right: sage #4A6659 filled "Add a pet" (44px, plus icon)
- Pet card grid 3 columns, white cards, 1px #E7DFD8, 12px radius, shadow sm, padding 20px:
  - Avatar 72px circle photo placeholder (alt text noted) + species icon badge 24px on #F6F2EF
  - Name H3 22px #1F1A17 ("Biscuit"), species/breed 14px #5C534C ("Dog · Beagle · 6y"), "Primary vet: Dr Amara Okoye" 13px #8A7F77 with small user icon
  - Status chips row: "Vaccinations up to date" green pill (#EAF4EF/#2F7D5A, check icon) or "Due in 12 days" amber pill (#FBF3E4/#B7791F, calendar icon); "Rx items on file" info pill (#EBF2FA/#2B6CB0) where relevant
  - Footer: outlined "View record" button (40px) + text link "Book for Biscuit →" #9B111E
- Second card is the "Add a pet" dashed placeholder: 2px dashed #D4C8BE, 12px radius, plus icon #4A6659, "Add another pet" 16px #5C534C
- Empty variant: sage-50 #F4F8F5 panel, paw icon, H2 "No pets added yet", body "Add your pets so bookings, prescriptions, and orders can be matched to them.", sage "Add your first pet" button + ruby "Or call us: 010 555 0199"

MOBILE LAYOUT (primary):
- Header stacks; "Add a pet" full width 56px sage
- Cards single column; layout per card: avatar left 64px + name/breed right, chips wrap below, buttons full width (outlined "View record" 48px, then link row 48px)
- Placeholder card full width 96px tall
- Empty state full width with stacked buttons (sage first, ruby call row 48px second)

ACCESSIBILITY:
- Each card has a single clear primary link; if multiple links, all are individually focusable with distinct names ("View record for Biscuit", "Book for Biscuit")
- Species conveyed by text + optional icon (icon aria-hidden); status chips contain text labels
- Alt text: "Photo of Biscuit, a beagle" (placeholder noted as such for mockups)
- Focus ring 3px #E0115F; contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (metadata only) · #2F7D5A on #EAF4EF ≈ 4.6:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #3B5349 on #F4F8F5 ≈ 6.9:1 · white on #4A6659 ≈ 5.9:1; dark: equivalents #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE ≈ 9:1 · #94887D ≈ 4.6:1 · #4ADE9B on #123528 ≈ 8:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #A3C4B0 on #241E1A ≈ 8:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px (48–56px on mobile)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 3 — Pet Detail with History Uploads

```
Generate the pet detail page for ruby-veterinary — one pet's record summary and uploaded medical history.

DESKTOP LAYOUT (sidebar shell, content max-width 1000px):
- Breadcrumb: My pets / Biscuit
- Header card (white, 12px radius, padding 24px): avatar 96px circle, name H1 "Biscuit" (36px), meta line "Dog · Beagle · Male neutered · 6y 2m · 14.2 kg" 16px #5C534C, chips: "Primary vet: Dr Amara Okoye" (sage-50/#3B5349), "Vaccinations up to date" green pill, "On prescription diet" info pill; right actions: sage "Book for Biscuit" filled (44px) + outlined "Edit details"
- Two-column (7 + 5):
  - Left: Card "Medical history uploads":
    - Upload row: outlined "Upload a file" button + helper "PDF or JPEG · up to 10 MB"
    - File list rows (1px #E7DFD8 dividers): file icon 20px sage, name 15px weight 600 #1F1A17 ("Biscuit_vaccination_record.pdf"), meta 12px #8A7F77 "PDF · 1.2 MB · Uploaded 12 Mar 2026 by you", right: outlined "View" (40px) + text "Download" link #9B111E; one row in uploading state: 4px progress bar (#4A6659 fill on #EFE9E4) "68%" + cancel; one row in error state: error icon + text #B8431F on #FDF0E8 "Upload failed — check your connection and try again." with "Retry" text link
    - Card "Care team": rows with avatar, name, role, "Message" text link; footer "Questions about Biscuit? Call 010 555 0199" ruby link
  - Right rail: Card "Details" definition list — Species, Breed, Sex, DOB, Weight, Microchip, allergies row highlighted #FBF3E4 with warning icon "Chicken protein allergy" #B7791F; Card "Active prescriptions" — pill "Rx pending" amber + product line + link "See status →"

MOBILE LAYOUT (primary):
- Header stacks: avatar 72px, name, meta wraps, chips wrap, buttons stack (sage full width 56px, outlined full width)
- History card full width: upload button full width 48px, file rows stack (name, meta, actions row with View/Download ≥44px each), error row full width with icon above text
- Details dl full width, rows split label/value; allergy row full width with icon + text
- Care team + prescriptions cards below; ruby phone row ≥48px

ACCESSIBILITY:
- dl/dt/dd for details; file list is a list with per-file accessible names including filename
- Uploading progress in role="status" ("Uploading, 68 percent"); upload failure in role="alert" with icon + text — not red alone
- Allergy warning uses icon + text (#B7791F on #FBF3E4), never colour alone
- Focus ring 3px #E0115F; keyboard completes upload → view → download
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C ≈ 7.4:1 · #8A7F77 ≈ 3.9:1 (metadata) · white on #4A6659 ≈ 5.9:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #B8431F on #FDF0E8 ≈ 5.4:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #9B111E on #FFFFFF ≈ 8.4:1; dark: #A3C4B0 fill with #171310 ≈ 9.4:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #F0754A on #3B1D10 ≈ 5.6:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #4ADE9B on #123528 ≈ 8:1 · #F53D6D on #171310 ≈ 6.2:1
- Tap targets ≥44px; images alt text ("Photo of Biscuit")

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 3: Subscriptions

### Screen 4 — Subscriptions Management

```
Generate the subscriptions management screen for ruby-veterinary — recurring preventatives and diet deliveries with pause/cancel and an Rx-linked warning.

DESKTOP LAYOUT (sidebar shell, content max-width 960px):
- Header: H1 "Subscriptions" (36px) + subline "Monthly preventatives and diet deliveries. Change or stop any time." + sage outlined "Start a subscription" (44px)
- Subscription cards (white, 1px #E7DFD8, 12px radius, padding 24px, margin-bottom 16px), example card:
  - Top row: product thumb 56px on #F6F2EF, name H3 22px "Heartgard Plus — Biscuit", variant 14px #5C534C "10–25 kg · 6 months supply", status chip "Active" green pill (#EAF4EF/#2F7D5A, check icon)
  - Facts row (4 mini stats, caption label 12px #8A7F77 + value 15px weight 600 #1F1A17): Frequency "Every month" · Next billing "1 Apr 2026 · $46.80" · Payment method "Visa •••• 4242" · Deliver to "Free in-clinic pickup"
  - Rx link note (if prescription-linked): #FBF3E4 panel, warning icon #B7791F, text 14px #1F1A17: "This subscription includes a prescription item. Pausing or cancelling stops future vet approvals too — your pet's current supply won't ship after the next cycle."
  - Actions row: outlined "Pause" (44px), outlined "Change frequency" (44px), text link "Change delivery" #9B111E, and destructive text action "Cancel subscription" in #B8431F with trash icon (NOT ruby, NOT a big red button — it's a labelled text action that opens a confirm dialog)
  - Collapsed state for a paused card: grey pill "Paused" (#EFE9E4/#5C534C) + "Resume" sage outlined button + line "Next charge on hold"
- Frequency editor (inline, shown expanded on one card): radio group "Every 2 weeks / Every month / Every 3 months" — selected 2px #4A6659 + #F4F8F5; sage "Save changes" (44px) + outlined "Cancel"
- Cancel confirm modal variant: white, 16px radius, H2 "Cancel Heartgard Plus subscription?", body "Biscuit won't receive the next cycle on 1 Apr 2026. You can restart any time — prescription items will need vet approval again." with warning icon panel; actions: outlined "Keep subscription" + destructive filled "Cancel subscription" (#D65328 fill, white text, trash icon) — error tokens, never ruby

MOBILE LAYOUT (primary):
- Header stacks; "Start a subscription" full width 56px
- Card stacks: thumb+name row, chips, 2×2 stat grid, Rx warning panel full width (icon above text), actions stack — Pause and Change frequency full width 48px outlined, "Cancel subscription" full width text row 48px with icon
- Frequency editor radios full width rows; sage Save full width 52px
- Cancel modal becomes a bottom sheet: stacked actions (destructive "Cancel subscription" full width 56px #D65328, then outlined "Keep subscription")

ACCESSIBILITY:
- Status conveyed by pill text ("Active"/"Paused") plus icon — never colour alone
- Pause/Resume/Cancel are real buttons with clear names including the product ("Cancel Heartgard Plus subscription")
- Rx warning is a complementary region with a heading or strong first sentence; icon aria-hidden, text mandatory
- Modal: role="dialog", aria-modal, focus trapped, Esc keeps subscription (safe default focus on "Keep subscription")
- Focus ring 3px #E0115F; contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (captions) · #2F7D5A on #EAF4EF ≈ 4.6:1 · #5C534C on #EFE9E4 ≈ 5.4:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #1F1A17 on #FBF3E4 ≈ 14:1 · #B8431F on #FFFFFF ≈ 5.5:1 · white on #4A6659 ≈ 5.9:1 · white on #D65328 ≈ 4.6:1 (large button text, icon + label); dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE ≈ 9:1 · #4ADE9B on #123528 ≈ 8:1 · #C4B8AE on #2A2420 ≈ 8:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #F7F3F0 on #3B2E10 ≈ 13:1 · #F0754A on #171310 ≈ 6.4:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · white on #E85C30 ≈ 4.5:1 (large)
- Tap targets ≥44px (48–56px on mobile); keyboard completes pause → save without a mouse

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Account Overview | 1 | ~5 |
| Pets | 2 | ~10 |
| Subscriptions | 1 | ~5 |
| **Total** | **4** | **~20** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one at a time.
3. Verify after each generation: only links and the emergency sidebar/strip are ruby; Book/Add pet/Save are sage #4A6659 (#A3C4B0 dark); Cancel uses #D65328 or #B8431F text.
4. Useful follow-ups: "Show the Rx-linked pause warning on the subscription card" · "Add the empty pets state with the phone fallback" · "Apply the dark mode mapping."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens
- `05-store-checkout.md` — order history and Rx badges linked from the dashboard
- `04-intake-appointments.md` — intake submissions visible in recent activity
