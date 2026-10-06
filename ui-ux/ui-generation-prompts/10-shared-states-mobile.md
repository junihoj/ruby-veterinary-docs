# Google Stitch Prompt — ruby-veterinary Shared States & Mobile Chrome

> **Purpose:** Paste this prompt into Google Stitch to generate the cross-cutting states every ruby-veterinary flow depends on — loading, empty, degraded, error, validation, and mobile navigation.
>
> **Coverage:** 7 screens — loading skeletons, empty states, offline/degraded banner, 500 error page, 404 page, form validation errors, mobile sticky bottom nav.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Every failure state keeps the clinic phone visible — graceful degradation never a dead end.

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
- Skeletons (loading blocks): base `#EFE9E4` → `#2A2420`, shimmer highlight `#F6F2EF` → `#3A322C`
- Cards/panels: `#F6F2EF` → `#1F1A16` · subtle fills `#EFE9E4` → `#2A2420` · elevated `#FFFFFF` → `#241E1A`
- Borders: `#E7DFD8` → `#3A322C` · stronger `#D4C8BE` → `#4C423A`
- Text: primary `#1F1A17` → `#F7F3F0` · secondary `#5C534C` → `#C4B8AE` · tertiary `#8A7F77` → `#94887D` · inverse `#FFFFFF` → `#171310`
- Links: `#9B111E` → `#F53D6D`
- Emergency ruby CTAs ("Call now", Call nav item): fill `#9B111E` → `#F53D6D`, white text in both modes · hover `#C00E52` → `#E0115F`
- Everyday primary buttons (Book, Submit, Continue, Retry): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310` · hover `#3B5349` → `#7BA88C`
- Input fills: `#FFFFFF` → `#1F1A16`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning (degraded banners): `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces
- Mobile bottom nav: bar `#FFFFFF` → `#1F1A16`, top border `#E7DFD8` → `#3A322C`, active item text `#4A6659` → `#A3C4B0`, inactive `#8A7F77` → `#94887D`, Call item stays ruby in both modes

---

## Group 1: Loading & Empty

### Screen 1 — Loading Skeletons (cards + forms)

```
Generate the loading skeleton states for ruby-veterinary — shimmer placeholders for product/service cards and for forms.

DESKTOP LAYOUT:
- Section A "Card grid skeleton" on white #FFFFFF: H2 "Loading cards" 22px (annotation label, 12px uppercase #8A7F77) above a 3-column row of skeleton cards:
  - Each card matches the real card geometry: white, 1px #E7DFD8, 12px radius, shadow sm
  - Skeleton blocks on #EFE9E4 with a soft shimmer sweep: image block (1:1, 12px radius), two text lines (16px tall, 60% and 85% width, 8px radius), one short line (40%, price), a button block (120×40, 8px radius)
  - Skeleton block colour #EFE9E4 with highlight #F6F2EF (dark: #2A2420 / #3A322C), subtle 1.2s shimmer animation (disabled under prefers-reduced-motion — static blocks instead)
- Section B "Form skeleton": a white card (12px radius, 1px #E7DFD8, padding 32px, max-width 640px) with skeleton label lines (100×14), input blocks (48px tall, 8px radius, full width), a select block with chevron, and a button block (180×48 in sage-tinted #E3EDE6)
- Live-region note (annotation): "Announce 'Loading…' politely once; skeletons are aria-hidden so screen readers aren't spammed."

MOBILE LAYOUT (primary):
- Card skeleton becomes single column: image block 16:9, then text lines, full-width button block (48px)
- Form skeleton full width, 16px padding, input blocks 48px, button block full width 56px
- Keep layout stable — no content jump when real data arrives (reserve final heights)

ACCESSIBILITY:
- Skeleton containers aria-hidden="true" with a single polite status message ("Loading products…"); no shimmer motion when prefers-reduced-motion is set (static #EFE9E4 blocks)
- Contrast is not required for decorative blocks, but surrounding real text must stay ≥4.5:1: #1F1A17 on #FFFFFF ≈ 16:1 · #8A7F77 annotation labels on #FFFFFF ≈ 3.9:1 (use only for mockup annotations; in-product annotations should be #5C534C ≈ 7.4:1); dark: #F7F3F0 on #171310 ≈ 17:1 · #94887D on #1F1A16 ≈ 4.6:1
- Focus must remain usable: while loading, the region is not focus-trapping; once loaded, focus stays where the user left it
- Stable heights prevent layout shift (CLS); tap targets on revealed content ≥44px

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 2 — Empty States (cart, inbox, orders)

```
Generate the empty states for ruby-veterinary — three variants: empty cart, empty staff inbox, and empty order history. Show all three on one canvas.

DESKTOP LAYOUT (three cards side by side, or three stacked panels):
Shared anatomy per empty state: centred line-art icon 48px in sage #4A6659 on a sage-50 #F4F8F5 circle 96px, H2 22px #1F1A17, body 16px/24px #5C534C, primary action, secondary link, and — where owner-facing — a phone fallback line.

1) Empty cart (owner):
- Icon: shopping cart outline
- H2 "Your cart is empty"
- Body: "Add supplies, therapeutic diets, or prescription items to get started."
- Actions: sage #4A6659 filled "Browse the store" (48px) + text link "See what's popular →" #9B111E
- Phone line 14px #5C534C: "Prefer to order by phone? Call 010 555 0199." with ruby link

2) Empty staff inbox:
- Icon: message-circle outline
- H2 "No conversations waiting"
- Body: "New WhatsApp messages and handovers appear here the moment they arrive. Unclaimed conversations sort above claimed ones."
- Actions: outlined "Refresh" (44px) + text link "Open admin alerts →" #9B111E
- Note 13px #8A7F77: "Out-of-hours away messages are active 18:00–08:00."

3) Empty order history (owner):
- Icon: package outline
- H2 "No orders yet"
- Body: "When you order supplies or prescription refills, they'll show up here with their approval status."
- Actions: sage #4A6659 filled "Browse the store" (48px) + outlined "Book an appointment instead" (44px)
- Phone line: "Or call the clinic: 010 555 0199."

MOBILE LAYOUT (primary):
- Each state full width, 32px vertical padding, icon centred, heading and body centred (max ~34ch), actions stack full width — sage button first (56px), then outlined/secondary (52px), phone row full width 48px
- Three states shown stacked with mock labels above each

ACCESSIBILITY:
- Each state is a labelled section (H2) — not an unlabelled image; icon aria-hidden
- Actions are real buttons/links with distinct accessible names ("Browse the store", "Open admin alerts")
- Phone fallbacks are tel: links with the number in the accessible name
- Focus ring 3px #E0115F (dark rgba(245,61,109,0.4)); contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #3B5349 on #F4F8F5 ≈ 6.9:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (helper meta only) · white on #4A6659 ≈ 5.9:1 · #9B111E on #FFFFFF ≈ 8.4:1; dark: #F7F3F0 on #171310 ≈ 17:1 · #C4B8AE on #1F1A16 ≈ 9:1 · #A3C4B0 on #241E1A ≈ 8:1 · #94887D on #171310 ≈ 4.6:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #F53D6D on #171310 ≈ 6.2:1
- Empty states must never strand a user: every one includes an action AND (owner-facing) a phone path
- Tap targets ≥44px (56px primary mobile)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Degraded & Error Pages

### Screen 3 — Offline / Degraded Banner (payment or WhatsApp down)

```
Generate the degraded-service banner for ruby-veterinary — shown when the payment gateway or WhatsApp integration is unavailable. Graceful degradation with a phone fallback.

DESKTOP LAYOUT (two banner variants shown stacked, both full-width strips under the site header, z-index sticky 100):

VARIANT A — Payment gateway down:
- Strip: background #FBF3E4 (dark #3B2E10), 4px top border warning #B7791F (dark #E3B341), padding 12px 24px
- Row: warning triangle icon 20px #B7791F + text 15px #1F1A17: "Online payments aren't working right now — nothing has been charged. You can still order and we'll call you to take payment, or collect and pay in clinic."
- Actions inline: outlined "Try again" (40px) + ruby #9B111E text link with phone icon "Call 010 555 0199"
- Optional dismiss × on the gateway banner only (44px), with note "service status re-checks every 30s"

VARIANT B — WhatsApp / bot unavailable:
- Strip: background #EBF2FA (dark #15273A), 4px top border info #2B6CB0 (dark #6BA3D6)
- Row: info icon 20px #2B6CB0 + text 15px #1F1A17: "WhatsApp messaging is temporarily unavailable. Your messages will queue — or call us now and we'll pick up: 010 555 0199."
- Actions: outlined "Check again" (40px) + ruby #9B111E link "Call the clinic"
- Sub-line 13px #5C534C: "Hours today 08:00–18:00 · 14 Maple Street, Rosebank"

Both variants: banner is non-dismissible while the service is down (variant A's dismiss only hides for the session, status chip "Payments: down" persists in the footer/status area).

MOBILE LAYOUT (primary):
- Full-width stacked strip: icon + heading text on line 1 (15px weight 600), body wraps below at 14px/20px, actions stack — outlined button full width 48px, then ruby call row full width 48px (tappable, phone icon + number)
- Hours + address line visible without scrolling past the banner
- Banner must not push the tap-to-call bar off screen — it sits directly under the header

ACCESSIBILITY:
- Banner is a labelled alert region: role="status" for the info variant, role="alert" for the payment outage (announce once, not repeatedly)
- Icon + text carry meaning; service state also stated in words ("down", "unavailable") — never colour alone
- "Try again / Check again" buttons announce state while retrying ("Checking…")
- Focus ring 3px #E0115F; contrast: #1F1A17 on #FBF3E4 ≈ 14:1 · #B7791F on #FBF3E4 ≈ 4.7:1 (icon/decorative; text is #1F1A17) · #1F1A17 on #EBF2FA ≈ 15:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #5C534C on #EBF2FA ≈ 7.1:1 · #9B111E on #FBF3E4 ≈ 7.4:1; dark: #F7F3F0 on #3B2E10 ≈ 13:1 · #E3B341 icon on #3B2E10 ≈ 7.4:1 · #F7F3F0 on #15273A ≈ 14:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #C4B8AE on #15273A ≈ 9:1 · #F53D6D on #3B2E10 ≈ 5.1:1
- Tap targets ≥44px; phone fallback present on every degraded state — never a dead end

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — 500 Error Page

```
Generate the 500 server error page for ruby-veterinary — a failed page load that still gets an owner to help.

DESKTOP LAYOUT (full-page, centred column max-width 640px on warm #FAF7F5):
- Small label 14px weight 600 uppercase #8A7F77 letter-spaced: "SERVER ERROR · 500"
- Illustration: simple line-art 96px in sage #4A6659 (a calm "we're on it" mark — e.g. a stethoscope or heartbeat line), NO red alarm imagery, no ruby flood
- H1 "Something went wrong on our end" (36px, #1F1A17)
- Body 18px/28px #5C534C: "This isn't your fault, and your details are safe. Please try again in a moment — or call us and we'll help straight away."
- Error reference row: "Reference " + mono "ERR-5841-773" (JetBrains Mono 14px #8A7F77) with copy button — for staff to trace
- Actions row: sage #4A6659 filled "Try again" (48px) + outlined "Back to home" (48px)
- Phone card directly below: #FFF5F7 background, 4px left border #9B111E, phone icon #9B111E + "Call the clinic: 010 555 0199" (number as a tel: link, 20px weight 600 #9B111E), helper 14px #5C534C "Open 08:00–18:00 · 14 Maple Street, Rosebank. Out of hours, call and follow the emergency instruction."
- Status link 13px #5C534C: "Check service status →"

MOBILE LAYOUT (primary):
- Full width, 24px padding: label, illustration 72px centred, H1 28px/36px centred, body 17px/26px, reference row wraps with copy button 44px
- Actions stack: sage "Try again" full width 56px, "Back to home" outlined full width 52px
- Phone card full width with the number as a full-width 56px tap row (icon + number), hours + address below in 14px
- Sticky tap-to-call bar from the design system also present

ACCESSIBILITY:
- Page has a single H1; the 500 code is text (screen readers get "Server error, code 500")
- Focus moves to the H1 or "Try again" button on load; focus ring 3px #E0115F
- Error reference copy button announces "Copy reference"
- No auto-refresh loops; "Try again" is a real button
- Contrast: #1F1A17 on #FAF7F5 ≈ 15:1 · #5C534C on #FAF7F5 ≈ 7.1:1 · #8A7F77 on #FAF7F5 ≈ 3.9:1 (reference meta — pair with real text) · #9B111E on #FFF5F7 ≈ 7.9:1 · white on #4A6659 ≈ 5.9:1; dark: #F7F3F0 on #171310 ≈ 17:1 · #C4B8AE on #1F1A16 ≈ 9:1 · #94887D on #171310 ≈ 4.6:1 · #F53D6D on #1F1A16 ≈ 5.4:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px; calm, non-alarming tone — no flashing, no ruby backgrounds

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 5 — 404 Page

```
Generate the 404 not-found page for ruby-veterinary — helpful wayfinding rather than a dead end.

DESKTOP LAYOUT (full-page, centred column max-width 640px on white #FFFFFF):
- Big display number "404" 96px weight 600, colour #E3EDE6 (light sage, decorative — aria-hidden) with "Page not found" as the real heading above it: H1 "We couldn't find that page" (36px, #1F1A17)
- Body 18px/28px #5C534C: "The link may be old, or we may have moved something. Everything below still works."
- Quick links list (white cards or plain list with 1px #E7DFD8 dividers, each row ≥48px with arrow icon):
  - "Home" → 
  - "Services & pricing"
  - "Book an appointment" (sage #4A6659 filled button variant, 48px)
  - "Clinic store"
  - "Blog"
  - "Contact & hours"
- Search field: "Search the site" label + input (44px, 1px #E7DFD8) + sage outlined "Search" button
- Phone card: #FFF5F7 bg, 4px left border #9B111E, phone icon + "Looking for something specific? Call 010 555 0199" (#9B111E link, 18px weight 600) + "08:00–18:00 · 14 Maple Street, Rosebank"

MOBILE LAYOUT (primary):
- Full width, 24px padding: "Page not found" H1 28px, "404" 72px in #E3EDE6 (aria-hidden), body 17px/26px
- Quick links as full-width rows ≥52px with arrows; "Book an appointment" full width 56px sage
- Search input full width 48px + "Search" full width 48px outlined
- Phone card full width with 56px tap row; hours + address below
- Sticky tap-to-call bar present

ACCESSIBILITY:
- One H1 ("We couldn't find that page"); decorative 404 numerals aria-hidden so the heading carries meaning
- Links list is a nav landmark ("Helpful links"); search input labelled with a visible label
- Focus ring 3px #E0115F; contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #9B111E on #FFF5F7 ≈ 7.9:1 · white on #4A6659 ≈ 5.9:1 · #E3EDE6 decorative only (non-text contrast exempt as decorative); dark: #F7F3F0 on #171310 ≈ 17:1 · #C4B8AE ≈ 9:1 · #F53D6D on #1F1A16 ≈ 5.4:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · decorative numeral should use #2A2420 on #171310
- Tap targets ≥44px (52–56px on mobile); every path offered — navigate, search, or call

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 3: Forms & Mobile Navigation

### Screen 6 — Form Validation Error Summary + Field Errors

```
Generate the form validation error states for ruby-veterinary — an error summary block plus per-field errors, using the orange-leaning error tokens and mandatory icons.

DESKTOP LAYOUT (one form card demonstrating the whole pattern, white card 12px radius, 1px #E7DFD8, padding 32px, max-width 640px):

ERROR SUMMARY (top of form, shown after failed submit):
- Panel: background #FDF0E8, border 1px #F5C9B4, 8px radius, 16px padding, left border 4px #D65328
- Header row: alert-circle icon 20px #D65328 + heading 16px weight 600 #B8431F "There are 3 problems with this form"
- List of links 15px #B8431F underlined: "Enter your email address" · "Choose a preferred time window" · "Tell us the reason for the visit"
- The summary receives focus after submit (tabindex="-1", focused in the mock with a visible focus ring)

FIELD ERRORS (three fields shown in error):
- Field 1 — Email: label "Email address (required)", input with border 2px #B8431F, value placeholder greyed, below: row with error icon 16px #B8431F + message 14px #B8431F on #FDF0E8 padding 8px radius 4px: "Enter a valid email address, e.g. name@example.com"
- Field 2 — Preferred window (radiogroup): legend "Preferred time window (required)", fieldset border 2px #B8431F with legend on #FDF0E8, three radio cards, group-level message with icon: "Choose a time window so we know when to call you."
- Field 3 — Reason (textarea): border 2px #B8431F, char counter "0 / 500", message with icon: "Tell us briefly what the visit is for (at least 10 characters)."
- Healthy field shown for contrast: normal border 1px #E7DFD8, helper text #5C534C

SUBMIT BUTTON: sage #4A6659 filled, 48px — stays enabled after errors (never disables submission), helper 13px #8A7F77 "Fix the highlighted fields and submit again."
Success summary variant: panel #EAF4EF, border #B7DCC7, check icon #2F7D5A, heading #2F7D5A-ish dark text "Everything looks good — sending…"

MOBILE LAYOUT (primary):
- Card full width, 16px padding; summary panel full width (icon above heading if narrow), list links full-width rows 48px
- Fields full width, inputs 48px, messages full width with icon above text if needed
- Radiogroup cards stack full width ≥56px; fieldset border 2px #B8431F
- Submit full width 56px sage; helper below
- After failed submit, focus lands on the summary which sits directly under the page title

ACCESSIBILITY:
- Summary: role="alert" container focused on submit; each item is a link that moves focus to its field
- Per-field messages associated via aria-describedby and aria-invalid="true"; required fields marked in text, not colour alone
- Error = icon + text + 2px border — never border colour alone
- Focus ring 3px #E0115F on inputs/links/buttons (on error fields the ring is visible outside the red border)
- Contrast: #B8431F on #FDF0E8 ≈ 5.4:1 · #B8431F on #FFFFFF ≈ 5.5:1 · #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (helper meta) · white on #4A6659 ≈ 5.9:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #1F1A17 on #FDF0E8 ≈ 15:1; dark: #F0754A on #3B1D10 ≈ 5.6:1 · #F0754A on #1F1A16 ≈ 5.6:1 · #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE ≈ 9:1 · #94887D ≈ 4.6:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #4ADE9B on #123528 ≈ 8:1 · #F7F3F0 on #3B1D10 ≈ 13:1
- Tap targets ≥44px (48–56px mobile); keyboard-only fix-and-resubmit loop works

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 7 — Mobile Sticky Bottom Navigation (Call / Book / Store / Account)

```
Generate the mobile sticky bottom navigation for ruby-veterinary — the persistent 4-item thumb bar for owner-facing pages. Call is the ruby item.

MOBILE LAYOUT (primary, shown at 390×844 with page content behind it):
- Bar: fixed bottom, height 64px + safe-area inset, background #FFFFFF (dark #1F1A16), top border 1px #E7DFD8 (dark #3A322C), shadow md upward (0 -4px 12px rgba(31,26,23,.10)), z-index sticky 100
- Four equal items, each ≥44×44px tap area, icon 24px + label 11px:
  - Item 1 "Call" — RUBY: phone icon and label in #9B111E (dark #F53D6D), weight 600, sitting on a subtle ruby-soft pill background #FFF5F7 (dark: #1F1A16 with #F53D6D) OR simply ruby-coloured glyphs; entire item is a tel: link "Call clinic — 010 555 0199" in the accessible name; this is the emergency affordance and the ONLY ruby item
  - Item 2 "Book" — calendar icon, sage #4A6659 when active (dark #A3C4B0), inactive #8A7F77 (dark #94887D)
  - Item 3 "Store" — shopping-bag icon, same active/inactive rules
  - Item 4 "Account" — user icon, same active/inactive rules
- Active state: icon + label in the item's active colour plus a 2px top indicator or weight change — never colour alone; current page also announced with aria-current="page"
- Optional above-bar status: when open, a slim pill row above the nav showing green dot + "Open · closes 18:00" (12px) — collapses when closed ("Closed · emergency line open")
- Content behaviour: page content gets bottom padding ≥ 64px + safe area so nothing is hidden; the nav hides or stacks with the tap-to-call bar from Screen 3 of `01-emergency-homepage.md` (design note: when both are present, merge them — Call item replaces the standalone bar)

DESKTOP LAYOUT (annotation only):
- No bottom nav on desktop: navigation lives in the header (logo, Services, Our Team, Blog, Store, Contact, theme toggle) with the phone number in the utility strip. Show a small annotated note: "Desktop: bottom nav is hidden ≥768px; header + utility strip take over."

ACCESSIBILITY:
- Nav is a landmark: nav aria-label="Primary" ; items are links/buttons ≥44×44px; Call is a real tel: link
- Active item: aria-current="page" + visible weight/indicator change — not colour alone
- Focus ring 3px #E0115F (dark rgba(245,61,109,0.4)) visible on every item and NOT clipped by the bar (outline-offset inside)
- Contrast: #9B111E on #FFFFFF ≈ 8.4:1 (Call, light) · #8A7F77 on #FFFFFF ≈ 3.9:1 (inactive labels — raise inactive label colour to #5C534C ≈ 7.4:1 or keep icons ≥3:1 as non-text with text labels at 11px meeting 4.5:1 — prefer #5C534C) · #4A6659 on #FFFFFF ≈ 5.9:1 (active Book) · #FFF5F7 pill behind #9B111E ≈ 7.9:1; dark: #F53D6D on #1F1A16 ≈ 5.4:1 · #94887D inactive → prefer #C4B8AE ≈ 9:1 · #A3C4B0 active ≈ 8.5:1 · #1F1A16 bar with #F7F3F0 labels ≈ 16:1
- Labels are always visible text (no icon-only items); safe-area inset respected; works with 200% text scaling without clipping labels (allow labels to shrink to 10px then wrap/hide icons)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Loading & Empty | 2 | ~10 |
| Degraded & Error Pages | 3 | ~15 |
| Forms & Mobile Navigation | 2 | ~10 |
| **Total** | **7** | **~35** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one at a time.
3. Verify after each: every error/degraded/empty state includes a phone fallback; every error uses #B8431F/#D65328 + icon, never ruby #9B111E.
4. Useful follow-ups: "Merge the Call nav item with the sticky tap-to-call bar" · "Make the skeletons static under reduced motion" · "Apply the dark mode mapping to the error summary."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens (error vs ruby rule)
- `../patterns-emergency-first.md` — persistent contact chrome patterns
- `01-emergency-homepage.md` — sticky tap-to-call bar this nav merges with
- `05-store-checkout.md` — degraded payment banner in checkout context
- `07-whatsapp-messaging.md` — degraded WhatsApp banner in chat context
