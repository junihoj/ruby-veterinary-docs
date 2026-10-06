# ruby-veterinary — Design System Context for Google Stitch

> **Purpose:** Paste this prompt into Google Stitch FIRST to generate a DESIGN.md file. All screen prompts in this directory assume this DESIGN.md exists on the canvas.
>
> **Coverage:** 1 paste-first prompt covering product context, full colour system (light + dark), typography, spacing, radius, shadows, z-index, motion, breakpoints, ruby discipline, WCAG 2.1 AA, and emergency CTA guidance.
>
> **Tip:** Copy the code block below, paste into Google Stitch, generate once. Then move to any flow file.

---

## Prompt (Paste into Stitch)

```
I'm designing ruby-veterinary — the complete digital front door for a single veterinary clinic. It is an emergency-first public website: the clinic phone number, hours, and address must be visible within 3 seconds on every page. Around that core it carries service pages with baseline pricing, staff profiles with credentials and accreditations, a clinic blog with categories, tags, and author bylines, a WhatsApp triage bot with numbered menus and human handover, an online store (general supplies, therapeutic diets, and prescription items that require veterinarian authorisation), new-client intake with PDF/JPEG history uploads, appointment requests, a client account area (orders, pets, subscriptions), and a role-scoped staff back office (alerts, catalogue, content, prescription review).

TARGET PLATFORM: Responsive web app — mobile-first for pet owners, desktop-first for staff back-office tools. Built with Next.js 16 + React 19 + Tailwind CSS 4. Light mode AND dark mode are both first-class and ship together.

BRAND PERSONALITY: Calm, caring, trustworthy, urgent only when needed. Plain-spoken, clinically credible, never salesy in an emergency context. A vet clinic, not a blood bank.

DESIGN PHILOSOPHY: Warm and trustworthy. White and warm off-white surfaces carry the trust; muted sage #4A6659 carries the calm everyday work (book, submit, add to cart); brand ruby #9B111E is reserved for emergencies and links. Ruby discipline: ruby appears ONLY on emergency CTAs, "Call now" buttons, links, focus rings, and documented brand accents — never on every button, never as a full-width hero background, never for errors. Errors and destructive confirms use a distinct orange-leaning red (#B8431F text, #D65328 fill) with an icon and text, because colour is never the only signal. "Emergency before commerce": the clinic phone, hours, and address win every layout fight. 3-second clarity on any page. Graceful degradation never a dead end — every error, empty, or offline state keeps a phone fallback line.

TARGET AUDIENCE:
- Emergency Seeker: panicked owner on a phone, scans, taps the first phone number visible
- New Client: registering, uploading medical history, booking a first appointment
- Routine Care Owner: booking, checking prices, reading the blog, downloading care sheets
- Repeat Buyer: reordering preventatives, therapeutic diets, prescription refills, subscriptions
- Clinic staff: receptionist, veterinarian, vet technician, practice manager — role-scoped back office

COLOR SYSTEM (light / dark):
- Page background: #FFFFFF / #171310
- Warm alternate sections: #FAF7F5 / #1F1A16
- Cards and panels: #F6F2EF / #1F1A16
- Subtle fills (wells, table stripes): #EFE9E4 / #2A2420
- Elevated surfaces (modals, popovers): #FFFFFF / #241E1A
- Border primary: #E7DFD8 / #3A322C
- Border secondary (stronger): #D4C8BE / #4C423A
- Text primary: #1F1A17 / #F7F3F0
- Text secondary: #5C534C / #C4B8AE
- Text tertiary (muted, captions): #8A7F77 / #94887D
- Text inverse (on coloured fills): #FFFFFF / #171310
- Links: #9B111E / #F53D6D

Ruby ramp (light):
- ruby-50 #FFF5F7 · ruby-100 #FFE4EC · ruby-200 #FFC2D5 · ruby-300 #FF8FAD · ruby-400 #F53D6D
- ruby-500 #E0115F (brand bright — accents, focus ring, decorative) · ruby-600 #C00E52 (hover) · ruby-700 #9B111E (primary interactive: emergency CTAs, links) · ruby-800 #7B0E1B (active) · ruby-900 #5A0A14 (deep brand text)
- On dark: interactive ruby is #F53D6D, accent ruby is #E0115F, hover #E0115F
- Contrast: white on #9B111E ≈ 8.4:1 (AAA); #9B111E text on #FFFFFF ≈ 8.4:1; prefer #9B111E over #E0115F for body-size links

Sage ramp (everyday actions):
- sage-50 #F4F8F5 · sage-100 #E3EDE6 · sage-200 #C7DCCF · sage-300 #A3C4B0 · sage-400 #7BA88C
- sage-500 #5C7C6E · sage-600 #4A6659 (primary buttons in light) · sage-700 #3B5349 (strong headings, secondary links) · sage-800 #2E4239 · sage-900 #1F2D27
- On dark: interactive sage is #A3C4B0 with #171310 text, hover #7BA88C
- Contrast: white on #4A6659 ≈ 5.9:1 (AA+); #3B5349 on #FFFFFF ≈ 7.2:1

Semantic colours (light / dark):
- Success text: #2F7D5A / #4ADE9B · success bg: #EAF4EF / #123528 · success border: #B7DCC7 / #123528
- Warning text: #B7791F / #E3B341 · warning bg: #FBF3E4 / #3B2E10 · warning border: #E8C98A / #3B2E10
- Error text: #B8431F / #F0754A · error fill (destructive buttons): #D65328 / #E85C30 with white text · error bg: #FDF0E8 / #3B1D10 · error border: #F5C9B4 / #3B1D10
- Info text: #2B6CB0 / #6BA3D6 · info bg: #EBF2FA / #15273A · info border: #B8D0EA / #15273A
- Disabled fill: #EFE9E4 / #2A2420 · disabled text: #8A7F77 / #94887D
- Focus ring: 3px #E0115F light / 3px rgba(245,61,109,0.4) dark
- Backdrop scrim: rgba(0,0,0,0.45) both modes

BUTTON SEMANTICS (critical):
- Emergency / "Call now" / "Call the clinic": ruby fill #9B111E light / #F53D6D dark, white text, phone icon plus visible number
- Everyday primary (Book appointment, Add to cart, Submit, Continue, Save, Publish): sage fill #4A6659 light (white text) / #A3C4B0 dark (#171310 text)
- Secondary/outline: transparent fill, 1px border #D4C8BE light / #4C423A dark, text #3B5349 light / #A3C4B0 dark
- Destructive / reject / delete confirm: error fill #D65328 light / #E85C30 dark, white text, warning icon — NEVER brand ruby
- All buttons: 48px minimum height on mobile, 40px on desktop tables; focus ring visible

TYPOGRAPHY:
- Primary font: Inter (system-ui, -apple-system, sans-serif) — all UI
- Serif font: Merriweather (Georgia, serif) — blog article bodies and article card titles only
- Mono font: JetBrains Mono (ui-monospace, monospace) — SKUs, order IDs, tokens
- Display: 48px / 56px / -0.02em (used sparingly; emergency CTAs take priority)
- H1: 36px / 44px / -0.01em
- H2: 28px / 36px
- H3: 22px / 30px
- H4: 18px / 26px
- Body large: 18px / 28px (article lead paragraphs)
- Body: 16px / 24px (default)
- Body small: 14px / 20px (helper text)
- Caption: 12px / 16px (labels, timestamps)
- Headings weight 600, tracking -0.01em; body weight 400; minimum 16px interactive text in mobile forms

SPACING (4px base unit): 0, 2, 4, 6, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96

BORDER RADIUS: 4px badges/tags · 8px buttons and inputs (md) · 12px cards (lg) · 16px modals (xl) · 9999px avatars and pills

SHADOWS:
- sm: 0 1px 3px rgba(31,26,23,.08)
- md: 0 4px 12px rgba(31,26,23,.10)
- lg: 0 12px 28px rgba(31,26,23,.14)
- Dark mode: raise opacity to 0.35+ so cards separate from #171310
- Focus ring: 0 0 0 3px rgba(224,17,95,0.35) light / rgba(245,61,109,0.4) dark

Z-INDEX SCALE: base 0 · sticky 100 · header 110 · dropdown 200 · overlay 400 · modal 500 · toast 600

MOTION: instant 75ms · fast 150ms · normal 200ms · slow 300ms · easing cubic-bezier(0.4, 0, 0.2, 1) · respect prefers-reduced-motion — no non-essential animation, no parallax, no attention-grabbing pulse on emergency UI

BREAKPOINTS: sm 640 · md 768 · lg 1024 · xl 1280 · 12-column grid · max width 1280px centred · gutters 16px mobile / 24px tablet / 32px desktop

EMERGENCY CTA GUIDANCE:
- A ruby "Call now" button with the literal phone number sits in the first viewport of every owner-facing page
- On mobile, a sticky bottom tap-to-call bar (z-index 100) keeps the clinic phone one tap away while scrolling
- An always-visible emergency strip (never dismissible) carries the phone plus the line "If this is life-threatening, go to the nearest emergency veterinary hospital"
- Hours and street address appear without opening a menu or accordion
- The phone icon is never shown alone: icon plus visible number text, always
- Out-of-hours states swap to an away message pointing exclusively to emergency care, still with the number

ACCESSIBILITY (WCAG 2.1 AA — the floor, not an audit afterthought):
- Full keyboard operability with a logical tab order and a visible focus ring on every interactive element
- Minimum 4.5:1 contrast for normal text, 3:1 for large text, in BOTH light and dark mode
- Tap targets ≥44×44px on all owner-facing mobile screens
- Meaningful alt text on photos; decorative icons aria-hidden
- Never colour alone: errors carry an icon and text; status badges carry a label
- Form errors announced with an error summary at the top and per-field messages
- Motion honours prefers-reduced-motion

Generate a DESIGN.md file from this complete design system specification. This DESIGN.md is the source of truth for all UI generation across ruby-veterinary — public site, blog, store, intake, WhatsApp surfaces, client account, and staff back office.
```

---

## Usage Note

1. Paste the block above into Google Stitch **first** and generate once. Stitch writes a **DESIGN.md** onto the canvas.
2. Then open any flow file (`01-emergency-homepage.md` … `10-shared-states-mobile.md`), paste its **Design System Context (Paste First)** block, and generate screens one at a time.
3. The condensed Design System Context in every flow file repeats the same hex tokens as this master prompt — if you start from a flow file alone, Stitch still gets a consistent brand.
4. Every screen prompt already requests both light and dark, desktop and mobile (4 variants). If a generation returns only one mode, follow up with: "Generate the dark mode variant using the Dark Mode Color Mapping."
5. Estimated cost: ~5 credits (Flash) for this design system generation.
