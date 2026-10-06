# ruby-veterinary UI Generation Prompts

> Google Stitch prompt files for generating the complete ruby-veterinary UI. Each file contains detailed, per-screen prompts that produce high-fidelity mockups when pasted into [Google Stitch](https://stitch.withgoogle.com).

---

## Purpose

ruby-veterinary is a single veterinary clinic's website: emergency-first public site, services/staff/hours/contact, clinic blog, WhatsApp triage bot with human handover, online store (general supplies, therapeutic diets, prescription items behind vet authorisation), client intake + appointment requests, and a staff back office.

This folder holds the Stitch prompts that encode the brand rules from `../design-system.md` into repeatable, screen-level generation requests — so every mockup, from any file, comes back on-brand in both light and dark mode.

---

## Quick Start

### Step 1: Generate the Design System

Open Google Stitch. Paste the contents of `master-prompt.md` into Stitch. Generate. Stitch will create a **DESIGN.md** file on the canvas containing the full ruby-veterinary design system (colours, typography, spacing, shadows, components, emergency patterns).

### Step 1.5: Understand Dark Mode

Every screen generates in 4 variants: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark. The **Dark Mode Color Mapping** section in each flow file tells Stitch exactly which hex values to swap. You don't need to specify dark colours manually — the mapping does it.

### Step 2: Generate Screens

Pick any flow file. Paste the **Design System Context (Paste First)** block at the top of that file first (so Stitch knows the brand), then paste each screen prompt **one screen at a time**. Stitch generates one screen per paste.

### Step 3: Iterate

After each generation, use follow-up prompts to refine:

- "Make the sticky call bar taller and always visible"
- "Change the Book appointment button to #4A6659"
- "Add a dark mode variant"
- "Show this on a mobile device frame"

**One change per prompt.** Stitch recreates the whole layout when you combine multiple changes.

### Step 4: Connect and Prototype

On Stitch's infinite canvas, connect related screens (homepage → services → booking form → confirmation). Click **Play** to preview the interactive prototype.

### Step 5: Export

- **Paste to Figma** — copies as editable frames for design refinement
- **Export HTML/CSS** — clean starting point for the Next.js + Tailwind build

---

## File Structure

| File | Flow | Screens | Credits |
|------|------|---------|---------|
| `README.md` | This guide | — | — |
| `master-prompt.md` | Design system context (paste first) | 1 | ~5 |
| `01-emergency-homepage.md` | Hero, emergency banner, sticky call bar, hours/location, services teaser | 5 | ~25 |
| `02-services-staff-clinic.md` | Services index/detail, staff grid/profile, contact, care documents | 6 | ~30 |
| `03-blog-newsletter.md` | Blog index, article, category, tag, newsletter subscribe | 5 | ~25 |
| `04-intake-appointments.md` | Appointment request, multi-step intake, uploads, confirmation, status check | 6 | ~30 |
| `05-store-checkout.md` | Catalog, filters, product detail, cart, checkout, payment, order history | 7 | ~35 |
| `06-prescription-pharmacy.md` | Rx status, review queue, detail, decision modal, audit trail | 5 | ~25 |
| `07-whatsapp-messaging.md` | Bot preview, panic escalation, staff inbox, conversation, handover | 5 | ~25 |
| `08-client-account-pets.md` | Dashboard, pets list, pet detail, subscriptions | 4 | ~20 |
| `09-staff-backoffice.md` | Staff login, alerts, product table/editor, article editor, Rx shortcut, inbox embed | 7 | ~35 |
| `10-shared-states-mobile.md` | Skeletons, empty states, degraded banner, 500/404, validation errors, mobile nav | 7 | ~35 |

**Totals: 57 screens, ~285 credits (~290 including the master prompt)**

---

## Recommended Generation Order (~2–3 days)

Stitch has daily credit limits (~350 Flash / ~50 Pro per day). Follow this schedule:

### Day 1 — Public site and content (~85 credits)
1. `master-prompt.md` (~5)
2. `01-emergency-homepage.md` (~25)
3. `02-services-staff-clinic.md` (~30)
4. `03-blog-newsletter.md` (~25)

### Day 2 — Intake and commerce (~90 credits)
5. `04-intake-appointments.md` (~30)
6. `05-store-checkout.md` (~35)
7. `06-prescription-pharmacy.md` (~25)

### Day 3 — Messaging, account, back office, polish (~115 credits)
8. `07-whatsapp-messaging.md` (~25)
9. `08-client-account-pets.md` (~20)
10. `09-staff-backoffice.md` (~35)
11. `10-shared-states-mobile.md` (~35)

---

## Each Flow File Contains

1. **Header** — Title, Purpose / Coverage / Tip blockquote
2. **Design System Context (Paste First)** — condensed brand token block, identical in every file, so any file works standalone
3. **Dark Mode Color Mapping** — hex-level token swaps for the dark variant
4. **Grouped Screens** — screens organised by sub-flow, each in its own fenced code block with DESKTOP LAYOUT, MOBILE LAYOUT, and an Accessibility block
5. **Credits Estimate** — per-group and total credit breakdown (~5 credits per screen on Flash)
6. **Usage Instructions** — step-by-step workflow

---

## Stitch Tips

- **Paste the Design System Context first** for every flow file. Stitch needs it to keep screens consistent.
- **Generate one screen at a time.** Stitch performs better with focused requests.
- **Save screenshots** after every successful generation. Stitch can reset unexpectedly.
- **One change per follow-up prompt.** Combining changes causes layout breakage.
- **Use UI/UX terminology**: "sticky bar", "filter rail", "card grid", "bottom sheet", "modal".
- **Use hex colours** (#4A6659, #9B111E) not colour names ("sage", "ruby") inside prompt blocks.
- **Both modes, always.** Every screen prompt ends with the instruction to generate light and dark, desktop and mobile (4 variants).
- **If dark mode looks wrong**, reference the mapping explicitly: "Apply the Dark Mode Color Mapping from the reference section. Use #A3C4B0 for the Book appointment button."
- **Ruby discipline check** after each generation: only emergency/Call-now CTAs and links are ruby #9B111E / #F53D6D. If every button came back ruby, correct it: "Make all non-emergency buttons #4A6659."
- **Errors are never ruby.** Validation and destructive states must use #B8431F text / #D65328 fill plus an icon.

---

## Related Documents

| Document | Location | Purpose |
|----------|----------|---------|
| Design System | `../design-system.md` | Authoritative brand palette and tokens — source of truth |
| UI Components | `../ui-components.md` | Component variants built from these tokens |
| Emergency Patterns | `../patterns-emergency-first.md` | Layout patterns for the 3-second rule |
| Product Requirements | `../../prd.md` | Scope, modules, and sequencing |
| Functional Requirements | `../../functional-requirements.md` | Feature checklist each screen satisfies |
| User Personas | `../../user-personas.md` | Who each screen is for |
