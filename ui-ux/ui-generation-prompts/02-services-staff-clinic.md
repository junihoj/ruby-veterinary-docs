# Google Stitch Prompt — ruby-veterinary Services, Staff & Clinic Info

> **Purpose:** Paste this prompt into Google Stitch to generate the public services catalog, staff profiles, and clinic information screens for ruby-veterinary.
>
> **Coverage:** 6 screens — services index, service detail, staff grid, staff profile, contact page, downloadable care documents.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Baseline prices are always shown as "from $X" — transparency is a product requirement.

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

## Group 1: Services Catalog

### Screen 1 — Services Index Grid

```
Generate the services index page for ruby-veterinary — the public grid of everything the clinic does, with transparent baseline pricing.

DESKTOP LAYOUT:
- Breadcrumb: Home / Services (14px, #5C534C, current item #1F1A17)
- Page header on warm background #FAF7F5: H1 "Our services" (36px), subline "Clear baseline pricing, honest guidance, and no surprises." (18px, #5C534C)
- Category chips row (filter chips, pill radius 9999px): All · Wellness · Surgery · Dental · Urgent care · Nutrition — selected chip sage #4A6659 fill with white text, unselected white fill with 1px #E7DFD8 border and #5C534C text
- 3-column card grid (12-col grid, 8 cards), each card white #FFFFFF, 1px #E7DFD8 border, 12px radius, shadow sm, padding 24px:
  - 24px stroke icon in sage #4A6659 (Urgent care card uses ruby #9B111E)
  - Service title H3 22px #1F1A17 (e.g. Wellness Exams · Vaccinations · Soft-tissue Surgery · Dental Cleaning · Urgent Care · Therapeutic Diet Planning · Microchipping & Travel Certificates · Senior Pet Care)
  - Two-line description 14px #5C534C
  - Baseline price "From $45" 14px weight 600 #3B5349, with helper "(baseline — final quote at consult)" 12px #8A7F77
  - Row of actions: sage outlined "Details" link + ruby text link "Call about this" only on the Urgent care card
- Footer CTA band on sage-50 #F4F8F5: "Not sure which you need? Call us" + ruby "Call 010 555 0199" link + sage filled "Book appointment" button

MOBILE LAYOUT (primary):
- Breadcrumb collapses to "Home / Services" single line
- H1 + subline stacked; chips become a horizontally scrollable row (visible partial next chip)
- Single-column cards, full width, 16px padding; Urgent care card promoted to first position with 4px ruby left border
- Each card: icon+title row, description, price block, full-width "Details" link row (48px)
- Footer CTA stacks: ruby call link first, sage Book button full-width 56px

ACCESSIBILITY:
- Chips are real buttons in a group with aria-pressed; selected state has text/shape change, not colour alone
- Cards keyboard reachable in reading order; focus ring 3px #E0115F visible; no hover-only content
- Contrast: #3B5349 on #FFFFFF ≈ 7.2:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 only used for 12px non-essential helper text — raise to #5C534C for anything load-bearing; dark equivalents all ≥4.5:1
- Price "From $X" is real text, selectable; icons aria-hidden
- Tap targets ≥44px; horizontal chip scroller must also work with keyboard (arrow/tab, no scroll trap)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 2 — Service Detail Page

```
Generate the service detail page for ruby-veterinary — one service (Dental Care) with baseline pricing and a booking call to action.

DESKTOP LAYOUT:
- Breadcrumb: Home / Services / Dental Care
- Two-column layout (8 + 4) on white #FFFFFF:
  - Left column: H1 "Dental Care" (36px, #1F1A17); price block card on sage-50 #F4F8F5: "From $280" (28px, weight 600, #3B5349) with note "Includes exam, scale and polish under anaesthesia. Final quote confirmed at consult." (14px, #5C534C)
  - Intro paragraph 16px/24px #1F1A17
  - "What's included" list with sage #4A6659 check icons: pre-anaesthetic bloodwork discussion, scale and polish, dental charting, take-home care plan
  - "When to book" section: bullets for bad breath, tartar, loose teeth, reluctance to eat
  - FAQ accordion (3 items), 1px #E7DFD8 dividers, chevron icons, question text 16px weight 600
  - Related staff row: two small vet cards (photo placeholder, name, "Dentistry") with "View profile" ruby links
  - Right column (sticky, top 110px): booking card — white, 1px #E7DFD8, 12px radius, shadow md
    - "Book this service" H3 22px
    - Sage #4A6659 filled "Book appointment" (48px, white text) — primary
    - Outlined secondary "Add to my pets" 
    - Divider, then ruby #9B111E link line with phone icon: "Questions? Call 010 555 0199"
    - Small hours snippet "Today 08:00–18:00" with green #2F7D5A dot
- Emergency strip retained above header (persistent chrome)

MOBILE LAYOUT (primary):
- Stacked: breadcrumb, H1, price card, intro, included list, when-to-book, staff row, FAQ
- Booking card becomes a sticky bottom panel above the tap-to-call bar: sage "Book appointment" full width (56px) with a small ruby "Call" icon button beside it (48px, phone icon + "Call")
- FAQ accordions full-width rows, ≥48px headers

ACCESSIBILITY:
- Accordion buttons expose aria-expanded and are keyboard operable with Enter/Space; focus ring 3px #E0111E-equivalent #E0115F visible (dark: rgba(245,61,109,0.4))
- Price and "from" qualifier are text — never baked into an image
- Alt text for the service photo placeholder ("Dog receiving a dental check at the clinic")
- Contrast: #3B5349 on #F4F8F5 ≈ 6.9:1 · white on #4A6659 ≈ 5.9:1 · #9B111E on #FFFFFF ≈ 8.4:1; dark mode #A3C4B0 fill with #171310 text ≈ 9.4:1
- Tap targets ≥44px; sticky booking panel must not cover the last FAQ row (bottom padding reserved)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: People & Contact

### Screen 3 — Staff Grid

```
Generate the staff grid page for ruby-veterinary — the public team listing that builds trust through credentials.

DESKTOP LAYOUT:
- Breadcrumb: Home / Our Team
- Page header on warm #FAF7F5: H1 "The people who care for your pet" (36px), subline 18px #5C534C "Licensed, accredited, and happy to answer your questions."
- 4-column card grid (or 3 wide cards), each card white, 1px #E7DFD8 border, 12px radius, shadow sm, padding 0 with image top:
  - Photo placeholder area 4:3 (alt text noted), warm background #F6F2EF
  - Body padding 20px: name H3 22px #1F1A17; role 14px #5C534C ("Veterinarian", "Vet Technician", "Practice Manager")
  - Credential chips row: pill badges, sage-50 #F4F8F5 background, #3B5349 text, 12px: "DVM", "AAHA", "Fear Free", "BVSc"
  - Two-line bio teaser 14px #5C534C
  - Ruby #9B111E text link "View profile →"
- Bottom band: sage #F4F8F5 panel "Book an appointment with our team" + sage filled button

MOBILE LAYOUT (primary):
- Single column, full-width cards, 16px padding
- Card: photo left (80px square, full radius 9999px avatar) + name/role/chips right, bio below, "View profile →" full-width link row 48px
- Book band: sage button full-width 56px

ACCESSIBILITY:
- Staff photos have descriptive alt text ("Dr Amara Okoye, veterinarian at ruby-veterinary")
- Cards are single link targets with clear accessible names containing the person's name
- Credential chips are text; the chip background alone never carries meaning
- Contrast: #3B5349 on #F4F8F5 ≈ 6.9:1 · #5C534C on #FFFFFF ≈ 7.4:1; dark mode #A3C4B0 on #241E1A ≈ 8:1
- Focus ring visible on every card; tab order follows visual order; ≥44px targets

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — Staff Profile Page

```
Generate the staff profile page for ruby-veterinary — a single veterinarian's public profile with credentials and accreditations.

DESKTOP LAYOUT:
- Breadcrumb: Home / Our Team / Dr Amara Okoye
- Two-column header (4 + 8) on warm #FAF7F5:
  - Left: large portrait placeholder (square, 12px radius, alt text noted) + outlined secondary button "Book with Dr Okoye"
  - Right: name H1 "Dr Amara Okoye" (36px), role "Veterinarian · BVSc, MSc" 18px #5C534C, accreditation badges row (pill, white fill, 1px #E7DFD8 border, 14px #3B5349 with small check icon): "AAHA accredited clinic", "Fear Free certified", "SAVC registered"
  - Contact row: phone icon + ruby #9B111E link "Ask for Dr Okoye: 010 555 0199", languages spoken chips
- Body on white #FFFFFF: H2 "About" with 16px/24px paragraph; H2 "Clinical interests" as sage #F4F8F5 chips (Dentistry, Feline medicine, Senior care); H2 "Articles by Dr Okoye" as 3 compact article rows with Merriweather 16px titles and dates
- Right sidebar card: "Book an appointment" — sage filled "Book appointment" button, hours snippet, ruby call link

MOBILE LAYOUT (primary):
- Stacked: portrait (full width, max 240px), name, role, badges wrap, book button full-width sage 56px, contact ruby link row, About, interests chips, articles list, booking card
- 48px minimum on all controls

ACCESSIBILITY:
- Portrait alt text "Dr Amara Okoye smiling in scrubs at ruby-veterinary"
- Accreditation badges include text labels; icons aria-hidden
- Heading hierarchy strictly H1 name → H2 sections → H3 article titles
- Contrast: #3B5349 on #FFFFFF ≈ 7.2:1 · #5C534C on #FAF7F5 ≈ 7.1:1; dark mode equivalents #A3C4B0 / #C4B8AE on #171310 ≥4.5:1
- Focus visible on book/contact/article links; tap targets ≥44px

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 5 — Contact Page

```
Generate the contact page for ruby-veterinary — where the emergency number is deliberately dominant.

DESKTOP LAYOUT:
- Breadcrumb: Home / Contact
- Top emergency panel (full width, background #FFF5F7, 4px left border #9B111E, dark: #1F1A16 + #F53D6D border):
  - H1 "Contact the clinic" (36px, #1F1A17) sits inside the panel
  - Emergency line: phone icon (32px, #9B111E) + "Emergency: 010 555 0199" at 30px weight 600, tel: link in #9B111E — the single largest element on the page
  - Supporting 14px #5C534C: "Open 08:00–18:00 daily. Outside hours, call and follow the emergency instruction, or go to the nearest emergency hospital."
- Two-column body (6 + 6) on white:
  - Left "Ways to reach us" list: General line 010 555 0199 (ruby link) · WhatsApp triage 010 555 0198 (info icon + note "Bot first, human on request") · Email hello@rubyvets.example · Address 14 Maple Street, Rosebank with map-pin icon and "Get directions" sage outlined button · Pharmacy/refill line
  - Right: contact form card — Name, Email, Phone, "How can we help?" select (Appointment question / Prescription refill / Records transfer / Something else), Message textarea, consent checkbox, sage #4A6659 filled "Send message" (48px) + helper "For emergencies, please call — this form is checked once a day." (14px, #5C534C)
- Hours table compact under the columns with today's row highlighted sage-50 #F4F8F5

MOBILE LAYOUT (primary):
- Emergency panel first, number at 28px, full-width tap-to-call row (56px tall)
- Ways to reach us as stacked rows, each ≥48px tappable
- Contact form full width, inputs 48px tall, submit full-width 56px sage
- Hours table full width; directions button full-width

ACCESSIBILITY:
- Emergency phone is the first focusable element after skip link; accessible name includes the number
- Form fields have persistent visible labels (not placeholder-only), required fields marked with text "(required)"
- Error state example shown: field border #B8431F, error icon + text message 14px #B8431F on #FDF0E8
- Contrast: #9B111E on #FFF5F7 ≈ 7.9:1 · #1F1A17 on #FFF5F7 ≈ 15:1 · #B8431F on #FDF0E8 ≈ 5.4:1; dark mode #F53D6D on #1F1A16 ≈ 5.4:1, #F0754A on #3B1D10 ≈ 5.6:1
- Keyboard: logical order through the form; focus ring 3px #E0115F; error summary receives focus on failed submit
- Tap targets ≥44px; no colour-only status

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 6 — Downloadable Care Documents List

```
Generate the downloadable care documents page for ruby-veterinary — public PDFs: post-op care sheets, travel certificates, liability waivers.

DESKTOP LAYOUT:
- Breadcrumb: Home / Care documents
- H1 "Care documents & forms" (36px) + intro 16px #1F1A17: "Everything you may need before or after a visit. All files are PDFs."
- Category group headings H2 (22px): "After your pet's surgery", "Travel & certificates", "Forms to complete before your visit"
- Each document row (list, not cards): white background, 1px #E7DFD8 bottom border, 12px radius on hover, padding 16px 20px:
  - File-type icon 24px sage #4A6659 with "PDF" label
  - Title 16px weight 600 #1F1A17 (e.g. "Post-operative dental care sheet", "International travel certificate checklist", "New client registration form", "Liability waiver")
  - Meta 13px #8A7F77: "PDF · 240 KB · Updated Mar 2026"
  - Right: outlined secondary "Download" button with download icon (40px desktop)
- Sidebar note on sage-50 #F4F8F5: "Need this form in another format? Call 010 555 0199 and we'll help." with ruby phone link

MOBILE LAYOUT (primary):
- Single-column rows, full width, 16px padding
- Row stacks: icon + title, meta line, full-width outlined "Download" button (48px) with download icon
- Category headings sticky-free, plain
- Help note full-width with ruby call link as a 48px tap row

ACCESSIBILITY:
- Each download link names the file and format: accessible name "Download Post-operative dental care sheet (PDF, 240 KB)"
- File-type icon aria-hidden; the "PDF" text label carries meaning
- Row is one focus target; focus ring 3px #E0115F visible
- Contrast: #8A7F77 meta text is ≥3:1 on white — acceptable for 13px secondary meta only if paired with the 16px title; prefer #5C534C for anything needed to make a decision; dark mode #94887D on #171310 ≈ 4.6:1
- Tap targets ≥44px; download buttons full-width on mobile; keyboard operable without hover

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Services Catalog | 2 | ~10 |
| People & Contact | 4 | ~20 |
| **Total** | **6** | **~30** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one group at a time.
3. Save a screenshot after each generation; one change per follow-up prompt.
4. Useful follow-ups: "Show the price block on every service card" · "Make the emergency number larger than everything else on the contact page" · "Apply the dark mode mapping."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens
- `01-emergency-homepage.md` — persistent header/emergency strip patterns reused here
- `04-intake-appointments.md` — booking form the service CTAs link to
