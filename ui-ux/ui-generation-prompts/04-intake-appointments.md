# Google Stitch Prompt — ruby-veterinary Intake & Appointments

> **Purpose:** Paste this prompt into Google Stitch to generate the owner-facing booking and new-client intake flows for ruby-veterinary.
>
> **Coverage:** 6 screens — appointment request form, multi-step intake, history upload step, upload success, confirmation page, status check.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Urgency wording must never push an emergency owner into a form — always offer the phone.

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
- Form inputs: fill `#FFFFFF` → `#1F1A16`, border `#E7DFD8` → `#3A322C`, text `#1F1A17` → `#F7F3F0`, placeholder `#8A7F77` → `#94887D`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning: `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces
- Progress/stepper: active step fill `#4A6659` → `#A3C4B0`, completed step check `#2F7D5A` → `#4ADE9B`, upcoming step `#8A7F77` → `#94887D`

---

## Group 1: Appointment Request

### Screen 1 — Appointment Request Form

```
Generate the appointment request form for ruby-veterinary — the primary booking surface for pet owners.

DESKTOP LAYOUT:
- Breadcrumb: Home / Book an appointment
- Two-column layout (8 + 4) on white #FFFFFF:
  - Left: H1 "Request an appointment" (36px), subline 16px #5C534C "Tell us about your pet and when suits you — we'll confirm by phone or email within one business day."
    - Emergency notice above the form on #FFF5F7 with 4px left border #9B111E and alert icon: "Is this an emergency? Don't wait on a form — call 010 555 0199 now." (ruby #9B111E link)
    - Fieldset "Your details": Full name (text), Email (email), Phone (tel) — labels above fields, 14px weight 600 #1F1A17, inputs 44px, 1px #E7DFD8, 8px radius, white fill
    - Fieldset "Your pet": Pet name, Species select (Dog/Cat/Other), Breed (optional), "Which of your pets is this?" select if multiple
    - Fieldset "The visit": Reason for visit textarea (rows 4, helper "A sentence or two is plenty"), Preferred date input, Preferred window radio group as 3 large cards: Morning 08:00–11:00 · Midday 11:00–14:00 · Afternoon 14:00–18:00 (selected card: 2px #4A6659 border, #F4F8F5 background, check icon)
    - Urgency select: "Routine (check-up, vaccination)" · "Soon — within a few days" · "Urgent today" — helper text under Urgent: "If your pet is bleeding, struggling to breathe, or has eaten something toxic, call us instead." with ruby phone link
    - Consent checkbox: "I agree to the clinic contacting me about this request (see privacy policy)"
    - Sage #4A6659 filled "Send appointment request" button (48px, white text) + helper 13px #8A7F77 "This is a request, not a confirmed booking."
  - Right sidebar card (sticky): "What happens next" 1-2-3 list with sage numbered circles; hours snippet with green dot; ruby "Call 010 555 0199" link
- Error summary state shown: panel on #FDF0E8 with error icon #D65328, heading "There are 2 problems with this form" 16px weight 600 #B8431F, list of links to fields

MOBILE LAYOUT (primary):
- Emergency notice FIRST (full width, 48px tap-to-call row), then H1, subline
- All fields full width, stacked, 48px tall inputs, 16px font (no zoom-on-focus)
- Window cards stack vertically, each ≥56px tall
- Urgency select full width; helper + ruby call link full-width 48px row
- Submit button full width, 56px, sage #4A6659, sticky above the tap-to-call bar while focused
- "What happens next" card below the form; sidebar phone/hours card moves above the form fields (not hidden)

ACCESSIBILITY:
- Fieldsets with legends; every input has a persistent visible label; required fields marked with text "(required)"
- Radiogroup for window and urgency keyboard operable with arrow keys; visible focus ring 3px #E0115F (dark rgba(245,61,109,0.4))
- Error summary is focusable (role="alert", tabindex="-1"), each message links to its field; per-field messages pair icon + text in #B8431F on #FDF0E8
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C ≈ 7.4:1 · white on #4A6659 ≈ 5.9:1 · #B8431F on #FDF0E8 ≈ 5.4:1 · #9B111E on #FFF5F7 ≈ 7.9:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #F0754A on #3B1D10 ≈ 5.6:1
- Tap targets ≥44px; no placeholder-only labels; error state never signalled by border colour alone

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: New-Client Intake

### Screen 2 — Multi-Step Intake (Step 1 of 4)

```
Generate the new-client intake screen for ruby-veterinary — a multi-step form; show it at step 1 of 4.

FULL LAYOUT (responsive, same structure both breakpoints):
- Page on warm background #FAF7F5 with white form card (12px radius, shadow md, max-width 760px centred, padding 32px)
- Emergency line above card: small alert row #FFF5F7 + ruby #9B111E phone link "Emergency? Call 010 555 0199"
- Step indicator: horizontal 4-step progress across the top of the card —
  - Steps: 1 Your details · 2 Your pets · 3 Medical history · 4 Review
  - Active step: sage #4A6659 filled circle with white number, label 14px weight 600 #1F1A17
  - Completed steps: circle #EAF4EF fill with #2F7D5A check icon, label #5C534C, clickable
  - Upcoming steps: circle #EFE9E4 with #8A7F77 number, label #8A7F77
  - Connector lines 2px: completed #B7DCC7, upcoming #E7DFD8
  - Step text also reads "Step 1 of 4" for clarity
- Card body: H2 "Your details" (28px), helper 14px #5C534C
  - Fields: Full name, Email, Phone, Address (street, suburb, city, postal code), Preferred contact method (radio: Phone / Email / WhatsApp), "How did you hear about us?" optional select
  - Consent checkbox with privacy policy link
- Card footer: left "Save and continue later" outlined secondary button (helper: "We'll email you a resume link"), right sage #4A6659 filled "Continue to your pets" (48px)
- Sidebar note: "Takes about 4 minutes. You can save and come back."

MOBILE LAYOUT (primary):
- Card full width, 16px page padding, 20px card padding
- Step indicator becomes compact: "Step 1 of 4" text + 4-segment progress bar (sage fill 25%, track #EFE9E4) with step labels as a scrollable text row underneath
- Fields stack full width, 48px tall; address fields 2×2 grid collapses to single column
- Footer buttons stack: sage "Continue" full width 56px on top, "Save and continue later" full width outlined below

ACCESSIBILITY:
- Stepper is an ordered list with aria-current="step" on the active step; completed steps announced as "completed"
- All fields labelled and grouped in fieldsets; autosave status announced politely (role="status": "Draft saved 12:04")
- Focus ring 3px #E0115F on inputs, buttons, stepper links; tab order follows visual order
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #8A7F77 upcoming labels on #FFFFFF ≈ 3.9:1 (secondary only — step name also conveyed by position and "Step 1 of 4" text) · white on #4A6659 ≈ 5.9:1 · #2F7D5A on #EAF4EF ≈ 4.6:1; dark: #94887D on #241E1A ≈ 4.6:1, #A3C4B0 on #171310 ≈ 9.4:1, #4ADE9B on #123528 ≈ 8:1
- Tap targets ≥44px; progress segments are decorative, real text states progress

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 3 — History Upload Step (Step 3 of 4)

```
Generate the medical-history upload step for ruby-veterinary intake — step 3 of 4, where the owner uploads past vet records.

FULL LAYOUT (responsive):
- Same card shell as Screen 2; step indicator at "3 Medical history" (sage active, steps 1–2 completed green checks)
- H2 "Upload medical history" (28px), helper 14px #5C534C: "Vaccination records, past lab work, or discharge papers help us prepare. Optional — skip if you don't have them."
- Dropzone: 2px dashed border #D4C8BE, 12px radius, background #FAF7F5, padding 40px, centred:
  - Upload/cloud icon 32px in sage #4A6659
  - Text 16px weight 600 #1F1A17 "Drag files here, or browse"
  - Constraints 14px #5C534C: "PDF or JPEG · up to 10 MB per file · up to 8 files"
  - Outlined secondary "Choose files" button (40px)
- Uploaded file rows (list): white background, 1px #E7DFD8, 8px radius, padding 12px 16px, margin-top 8px:
  - File icon 20px sage (JPEG icon alt tone), filename 14px weight 600 #1F1A17 truncated, meta 12px #8A7F77 "PDF · 2.4 MB"
  - Right: progress bar (4px track #EFE9E4, fill #4A6659, width 68%) with "68%" 12px #5C534C, plus cancel icon button (44px hit area)
  - Completed row variant: progress replaced by green check #2F7D5A + "Uploaded" 12px on #EAF4EF pill
- Oversized-file error row: error icon + text #B8431F on #FDF0E8 "vet-records.pdf is 14 MB — files must be 10 MB or smaller. Try splitting or compressing it." with "Remove file" text link
- Footer: "Back" outlined secondary · sage "Continue to review" (48px)

MOBILE LAYOUT (primary):
- Step indicator compact bar (as Screen 2)
- Dropzone becomes a stacked tap target: icon, text, full-width outlined "Choose files" button (48px); supports tap-to-open file picker
- File rows full width; progress bar full width under filename; cancel button ≥44px
- Error message full width with icon + text, never truncated
- Footer buttons stack: sage "Continue to review" full width 56px on top, "Back" outlined below

ACCESSIBILITY:
- Dropzone works by keyboard: it is a focusable control with visible focus ring 3px #E0115F; browse button is a real button
- Progress announced with role="status" text ("Uploading vet-records.pdf, 68 percent"); completion announced once
- Errors in role="alert" with icon + text; contrast #B8431F on #FDF0E8 ≈ 5.4:1 (light), #F0754A on #3B1D10 ≈ 5.6:1 (dark)
- Contrast elsewhere: white on #4A6659 ≈ 5.9:1 · #1F1A17 on #FAF7F5 ≈ 15:1 · #5C534C on #FAF7F5 ≈ 7.1:1; dark #A3C4B0 fill with #171310 ≈ 9.4:1, #F7F3F0 on #1F1A16 ≈ 16:1
- File type and size limits are real text, not icon-only; tap targets ≥44px

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — Upload Success State

```
Generate the upload success state for ruby-veterinary intake — the confirmation moment after files finish uploading.

FULL LAYOUT (responsive, inside the same intake card):
- Large success icon: circle 64px, fill #EAF4EF, check stroke #2F7D5A (dark: #123528 / #4ADE9B)
- H2 "3 files uploaded" (28px, #1F1A17), helper 16px #5C534C: "Your pet's history is with us. Our team will review it before your first visit."
- Uploaded files grid/list: compact rows — filename 14px weight 600 #1F1A17, meta 12px #8A7F77 "PDF · 2.4 MB · Uploaded 14:22", green "Uploaded" pill (#EAF4EF / #2F7D5A), outlined "View" and text "Remove" links
- Info panel on #EBF2FA with info icon #2B6CB0: "You can add more files later from your account. Need to send something else now? Reply to your confirmation email."
- Actions: outlined secondary "Upload another file" + sage #4A6659 filled "Continue to review" (48px)
- Small ruby #9B111E link with phone icon: "Having trouble? Call 010 555 0199"

MOBILE LAYOUT (primary):
- Success icon centred, heading and helper centred then left-aligned list
- File rows full width with pill and links on their own line
- Info panel full width with icon above text if needed
- Buttons stack: sage "Continue to review" full width 56px, "Upload another file" outlined full width
- Ruby call row full width 48px

ACCESSIBILITY:
- Success message container role="status" (polite) announced once; icon aria-hidden
- "Uploaded" state conveyed by text pill, not only the green colour; removal controls named per file ("Remove vet-records.pdf")
- Focus ring 3px #E0115F on View/Remove/Continue; keyboard order logical
- Contrast: #2F7D5A on #EAF4EF ≈ 4.6:1 · #1F1A17 on #FFFFFF ≈ 16:1 · #2B6CB0 on #EBF2FA ≈ 5.6:1 · white on #4A6659 ≈ 5.9:1; dark: #4ADE9B on #123528 ≈ 8:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px; reduced-motion users get no celebratory animation (static check icon)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 3: Confirmation & Status

### Screen 5 — Intake / Appointment Confirmation Page

```
Generate the confirmation page for ruby-veterinary — shown after an intake submission or appointment request. Plain language, no jargon, tells the owner exactly what happens next.

FULL LAYOUT (responsive, single centred column max-width 720px on white #FFFFFF):
- Success icon circle 64px (#EAF4EF fill, #2F7D5A check)
- H1 "We've got your request" (36px, #1F1A17)
- Plain-language paragraph 18px/28px #1F1A17: "Thanks, Naledi. We've sent your appointment request for Biscuit (dog) to the clinic. Nothing is confirmed yet — a receptionist will call you on 072 555 0134 to agree a time."
- Reference card on sage-50 #F4F8F5, 12px radius: "Your reference" caption 12px #8A7F77 + code "RV-2026-0418" in JetBrains Mono 18px weight 600 #1F1A17 + copy icon button; helper "Quote this if you call."
- "What happens next" ordered list with sage #4A6659 numbered circles:
  1. We review your request — usually within a few hours during opening times
  2. We call or email you to confirm the exact time
  3. You get a confirmation with directions and what to bring
- What to bring panel on #EBF2FA with info icon: "Bring any medication your pet is on, and previous records if you have them."
- Hours + phone row: green dot + "Open now · closes 18:00" and ruby #9B111E link "Need to change something? Call 010 555 0199"
- Buttons: sage "Go to my account" filled (48px) + outlined "Back to home"
- Emergency note strip (small, #FFF5F7 with 4px #9B111E left border): "If your pet's condition changes, don't wait for us to call — call 010 555 0199 now."

MOBILE LAYOUT (primary):
- Single column, 16px padding; icon + H1 centred
- Paragraph 17px/26px; reference code centred with copy button
- Numbered list stacked, numbers left of text, full width
- Buttons stack: sage "Go to my account" full width 56px, "Back to home" outlined full width
- Emergency note full width with ruby call link as a 48px tap row

ACCESSIBILITY:
- H1 present; success region role="status"; reference code selectable and copy button announces "Copy reference"
- List uses semantic ol; step numbers are text
- Focus ring 3px #E0115F on all buttons/links; first focusable element after skip link is "Go to my account"
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #3B5349 on #F4F8F5 ≈ 6.9:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #9B111E on #FFF5F7 ≈ 7.9:1 · #2B6CB0 on #EBF2FA ≈ 5.6:1; dark equivalents ≥4.5:1 (#4ADE9B on #123528 ≈ 8:1, #F53D6D on #1F1A16 ≈ 5.4:1)
- Tap targets ≥44px; status never conveyed by the check icon alone (heading carries meaning)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 6 — Request Status Check Page

```
Generate the request status check page for ruby-veterinary — where an owner can check an appointment request or intake without an account, using a token/ID.

FULL LAYOUT (responsive, single centred column max-width 640px on warm #FAF7F5 with white card):
- Breadcrumb: Home / Check your request
- H1 "Check your request" (36px), helper 16px #5C534C "Enter the reference from your confirmation email — no account needed."
- White card (12px radius, shadow md, padding 32px):
  - Label "Reference number" 14px weight 600, input 44px, placeholder "RV-2026-0418", helper 13px #8A7F77 "Found in your confirmation email, format RV-YYYY-NNNN"
  - Label "Email address used" 14px weight 600, input 44px
  - Sage #4A6659 filled "Check status" button (48px, full width on card)
  - Divider "or" + ruby #9B111E phone link with icon: "Lost your reference? Call 010 555 0199"
- Result state panel (shown below the card): status timeline —
  - Header row: "Appointment request · Biscuit (dog)" 16px weight 600 + status pill
  - Status pills (icon + text always): Received (info #2B6CB0 on #EBF2FA) · In review (warning #B7791F on #FBF3E4) · Confirmed (success #2F7D5A on #EAF4EF) · Needs your call (warning with phone icon)
  - Vertical timeline with dots: "Received — Tue 14:22", "In review — Tue 15:05 by K. Patel", "Awaiting your callback — Wed 09:10"
  - Plain-language line 16px: "We still need to agree a time. Reception will call you today before 18:00."
- Not-found error state: error panel #FDF0E8 with error icon + text #B8431F "We couldn't find a request with that reference and email. Check both and try again, or call 010 555 0199."

MOBILE LAYOUT (primary):
- Card full width, 16px page padding; inputs and button full width, 48–52px tall
- Result timeline full width, dates under each event, pills wrap to their own line
- Status pills ≥44px tall chips with icon + text
- Error state full width, ruby call link as a full-width 48px row

ACCESSIBILITY:
- Both inputs labelled; error message in role="alert" and receives focus after a failed lookup
- Status pills use icon + text label — never colour alone (e.g. "Confirmed" always shows the word)
- Timeline is a semantic ordered list; dates are text
- Focus ring 3px #E0115F; contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #B8431F on #FDF0E8 ≈ 5.4:1 · #B7791F on #FBF3E4 ≈ 4.7:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · white on #4A6659 ≈ 5.9:1; dark: #F0754A on #3B1D10 ≈ 5.6:1 · #E3B341 on #3B2E10 ≈ 7.4:1 · #4ADE9B on #123528 ≈ 8:1 · #A3C4B0 fill with #171310 ≈ 9.4:1
- Tap targets ≥44px; keyboard-only path completes the whole lookup

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Appointment Request | 1 | ~5 |
| New-Client Intake | 3 | ~15 |
| Confirmation & Status | 2 | ~10 |
| **Total** | **6** | **~30** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate one screen at a time in order (form → steps → confirmation → status).
3. Useful follow-ups: "Make the emergency notice the first thing on mobile" · "Change the Continue button to sage #4A6659" · "Show the error summary state."
4. Connect Screens 2 → 3 → 4 → 5 into a prototype flow on the canvas.

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens
- `../patterns-emergency-first.md` — emergency-before-form layout rules
- `08-client-account-pets.md` — where owners later see their submissions
