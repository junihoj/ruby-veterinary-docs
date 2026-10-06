# ruby-veterinary UI/UX System Design

> **ruby-veterinary Documentation**
>
> **Document:** UI/UX System Design
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** UI Standard
>
> **References:** `design-system.md`, `ui-components.md`, `shadcn-tailwind.md`, `patterns-emergency-first.md`, `accessibility.md`

---

# 1. Purpose & Philosophy

ruby-veterinary is the digital front door for a single veterinary clinic: emergency-first public website, services with baseline pricing, staff profiles, blog, WhatsApp triage bot, online store (including prescription items requiring vet authorisation), new-client intake, appointment requests, owner account, and a role-scoped staff back office.

### Design Goals

- Surface emergency phone, hours, and address within **3 seconds** on every owner-facing page
- Feel calm, caring, and trustworthy — a vet clinic, not a blood bank
- Keep commerce and routine care easy without ever burying the emergency path
- Stay accessible to everyone on mobile under stress (WCAG 2.1 AA floor)
- Degrade gracefully — every error state keeps a phone fallback

### Brand Personality

| Trait | Expression |
|-------|------------|
| Caring | Warm off-white surfaces, plain language, soft sage |
| Calm | No aggressive animation, no full-viewport ruby fills |
| Trustworthy | Consistent patterns, transparent pricing, visible hours |
| Urgent only when needed | Ruby reserved for emergency CTAs and links |
| Plain-spoken | Clinical credibility without jargon walls |
| Inclusive | Mobile-first, keyboard paths, high contrast both themes |

### Design Principles

1. **Emergency before commerce** — clinic phone, hours, address win every layout fight
2. **Ruby used sparingly** — emergency CTAs, links, focus, brand accents only
3. **Warm and trustworthy** — white + warm off-white carry trust; sage softens everyday actions
4. **Errors are not brand buttons** — orange-leaning error tokens + icon + text
5. **Mobile-first, WCAG 2.1 AA** — the floor, not an audit afterthought
6. **3-second clarity** — emergency instructions, location, phone on every page
7. **Graceful degradation** — fallbacks are plain language + phone, never dead ends

---

# 2. Navigation Architecture

## 2.1 Public Header (all owner-facing pages)

```
+-------------------------------------------------------------------------+
| [Logo]  Services  Store  Blog  Staff        Hours  [Call 01632 960245] |
+-------------------------------------------------------------------------+
| Emergency strip (sticky, never dismissible):                           |
| If this is life-threatening, go to the nearest emergency hospital —    |
| [01632 960245]                                                          |
+-------------------------------------------------------------------------+
```

- Logo links home
- Call is a `brand-emergency` (ruby) tel: link with visible number
- Hours snippet uses `ClinicHoursChip` (open/closed text, not colour alone)
- Emergency strip never dismissible on mobile

## 2.2 Owner Mobile Bottom Navigation

```
+-------------------------------------+
|           [Content Area]            |
+-------------------------------------+
| [Call☎ruby] [Book sage] [Store] [Account] |
+-------------------------------------+
```

| Tab | Style | Action |
|-----|-------|--------|
| Call | brand-emergency (ruby) | tel: clinic line |
| Book | primary (sage) | Appointment request |
| Store | default | Catalogue |
| Account | default | Orders, pets, messages |

## 2.3 Staff Sidebar (desktop, role-scoped)

```
+------------------+
| [Dashboard]      |  today's appointments, alerts
| [Rx Review]      |  prescription queue
| [Catalogue]      |  products, diets, Rx items
| [Content]        |  blog, pages, staff profiles
| [Inbox]          |  WhatsApp + messages (claim/handover)
| [Clients]        |  owners, pets, records
| [Reports]        |  operations metrics
| [Settings]       |
+------------------+
```

States: expanded 256px · collapsed 72px · mobile hidden (menu).

## 2.4 Breadcrumbs

```
Home > Store > Heartgard Plus
Home > Blog > Dental care at home
```

Current page bold, not clickable; overflow dropdown for deep paths.

## 2.5 Quick Actions

| Role | Primary | Secondary |
|------|---------|-----------|
| Owner | Book appointment | Call clinic, Reorder |
| Reception | New appointment | Intake, Call owner |
| Vet | Rx review queue | Decision panel |
| Manager | Catalogue | Content, Reports |

---

# 3. Public Experience

## 3.1 Homepage (Emergency-First)

```
+-------------------------------------------------------------------------+
| Header: Logo  Services Store Blog Staff          Hours  [Call 01632…]  |
| Emergency strip: life-threatening → nearest emergency hospital + phone  |
+-------------------------------------------------------------------------+
| HERO (warm #FAF7F5 or white — NO full ruby fill)                       |
| [H1: Calm care for every life stage]                                   |
| [Sub: Same-day sick visits · Wellness plans · 24h emergency line]     |
|                                                                         |
| [Call now — 01632 960245  ruby]   [Book appointment — sage]           |
|                                                                         |
| Hours: Open now · closes 18:00     14 Oak Street, Springfield          |
+-------------------------------------------------------------------------+
| Services                                                               |
| [Wellness] [Surgery] [Dental] [Emergency] [Pharmacy]                   |
| Each card: baseline "From $X" + Book (sage) + Call for urgent (ruby)   |
+-------------------------------------------------------------------------+
| Why this clinic (3 cards) · Staff teaser · Latest blog · Newsletter    |
+-------------------------------------------------------------------------+
| Footer: address, hours, emergency number repeated, legal              |
+-------------------------------------------------------------------------+
```

**Psychology:** one clear next action (Book) beside the emergency path (Call); hours and address visible without menu; calm surfaces.

## 3.2 Services

Category sections with baseline pricing ("From $X"), each with Book (sage) + Call for urgent (ruby). No clinical overclaim language.

## 3.3 Staff Profiles

Photo, name, credentials/accreditations, bio, languages. Calm cards; contact via clinic line, not personal numbers.

## 3.4 Blog Index / Article

Index: category chips, article cards (serif titles), author byline.  
Article: Merriweather body, clear H1–H3, related services CTAs, emergency strip still present.

## 3.5 Store

```
+-------------------------------------------------------------------------+
| Store                                    [Search]  [Filters]           |
| Categories: [All] [Preventatives] [Therapeutic diets] [Rx items]       |
+-------------------------------------------------------------------------+
| +--------------+ +--------------+ +--------------+                      |
| | [Product]    | | [Product]    | | [Product]    |                      |
| | Name         | | Name         | | Name         |                      |
| | From $X      | | From $X      | | From $X      |                      |
| | [Rx badge]   | |              | | [Rx badge]   |                      |
| | [Add cart]   | | [Add cart]   | | [Add cart]   |                      |
| +--------------+ +--------------+ +--------------+                      |
+-------------------------------------------------------------------------+
```

Rx badge = warning + text "Prescription required". Add to cart = sage primary.

## 3.6 Cart / Checkout

```
+-------------------------------------------------------------------------+
| Cart                                                                     |
| [Line: variant, qty stepper, total, pet chip if Rx] [Remove error+icon]|
|                                                                         |
| Subtotal / Tax / Pickup $0 / Total                                      |
| [Prescription hold notice — warning tokens + icon + phone fallback]     |
|                                                                         |
| [Continue — sage]     Payment hand-off to tokenised gateway            |
| If payment is unavailable: call 01632 960245                            |
+-------------------------------------------------------------------------+
```

Never collect card fields on our page. Payment gateway down → error banner + phone, not a dead checkout.

## 3.7 Intake / Appointment Request

```
+-------------------------------------------------------------------------+
| New client intake                    Step 2 of 4                       |
| (1) Contact → (2) Pet → (3) Reason → (4) Prefer when → Submit          |
+-------------------------------------------------------------------------+
| [Form fields — labels always visible, 16px min]                        |
| Urgency: [routine | soon | urgent]  (hint: for life-threatening,       |
| call 01632 960245 now — not a clinical diagnosis)                      |
|                                                                         |
| Upload history: PDF/JPEG, 10 MB max, per-file progress                 |
|                                                                         |
| [Submit request — sage]                                                |
| Prefer not to wait online? Call 01632 960245 during opening hours     |
+-------------------------------------------------------------------------+
```

## 3.8 Status Check (no account)

Token/ID input → status card (appointment request or intake) in plain language + clinic phone.

---

# 4. Owner Account

## 4.1 Orders

List + detail: statuses, Rx approval state (`RxStatusBadge`), receipt, reorder.

## 4.2 Pets

Species, breed, weight, medication flags. Pet profiles support Rx claim selection.

## 4.3 Subscriptions

Preventatives and diets. Cancel confirm modal: warning context + destructive cancel in **error-strong** (never ruby); restart notes in plain language.

## 4.4 Messages

Conversation with clinic; claim/handover badges where relevant; phone fallback always available.

---

# 5. Staff Back Office

## 5.1 Dashboard

```
+-------------------------------------------------------------------------+
| Staff Dashboard                                                          |
| [Today: 12 appts] [Rx awaiting: 3] [Intake uploads: 2] [Handover: 1]  |
|                                                                         |
| Alerts                                                                  |
| - Rx review queue — 3 items pending                                     |
| - Intake: history PDF uploaded, not yet triaged                         |
| - Bot handover: Biscuit (heartworm concern)                            |
+-------------------------------------------------------------------------+
```

## 5.2 Rx Review Queue

```
+-------------------------------------------------------------------------+
| Rx Review                                    [Filter] [Sort]            |
| +-------------------------------------------------------------------+ |
| | Pet | Owner | Requested | Status | Emergency pin | [Claim] [Decide] |
| | Biscuit | Jane D. | 10:15 | pending | 🔴 pin | [Claim]              |
| | Max | Sam K. | 11:00 | pending | | [Claim]                        |
| +-------------------------------------------------------------------+ |
| Claimed by: Dr. Patel · awaiting supervisor handover…                  |
+-------------------------------------------------------------------------+
```

**Decision modal:** pet + prescriber context · order lines · history links  
Actions: **Approve** (sage primary) · **Query** (info) · **Reject** (danger/error-strong + required reason) — never ruby. Audit entry silent to user.

## 5.3 Catalogue

Product table; Rx toggle; variant editor (size/pack); pricing.

## 5.4 Content

Blog post editor: title, body, category, tags; **Publish** = sage primary.

## 5.5 Inbox

```
+-------------------------------------------------------------------------+
| Inbox   [Unclaimed] [Mine] [All]                                        |
| +----------------+ +--------------------------------------------------+ |
| | Biscuit — Jane | | Messages timeline (owner left / clinic right)    |
| | Max — Sam K.   | | [Claim conversation] when awaiting_handover     |
| |                | | Bot-paused badge during handover                |
| |                | | [Composer — enabled when claimed]               |
| +----------------+ +--------------------------------------------------+ |
+-------------------------------------------------------------------------+
```

---

# 6. Shared Experiences

## 6.1 Authentication

Staff sign-in; owner account sign-in/up. Clean card on warm surface; emergency strip still visible on public chrome.

## 6.2 Notifications Panel

Unread count; types (Rx decision, appointment, message, system); "Mark all read".

## 6.3 Theme Switcher

System / light / dark. 300ms crossfade. OS preference default; staff may toggle explicit dark (localStorage). Same tokens both themes (see `shadcn-tailwind.md`).

## 6.4 Empty / Error / Loading

| State | Pattern |
|-------|---------|
| Empty | Illustration optional, sage primary action, phone fallback line |
| Error | Icon + plain language + Retry + clinic phone |
| Loading | Skeletons matching layout; reduced-motion → static |
| Offline/degraded | Banner + phone; never a dead end |

## 6.5 Degraded Dependencies

| Dependency down | UI |
|-----------------|-----|
| Payment gateway | Checkout banner: call to order/pay in clinic + phone |
| WhatsApp/bot | "Message us when it's working, or call [phone] now" |
| Email/newsletter | Accept input; success copy includes phone if no confirmation |
| Origin down | CDN-cached emergency strip + hours where possible |

---

# 7. Responsive Behavior

| Breakpoint | Layout | Navigation | Sidebar | EmergencyCallBar |
|------------|--------|------------|---------|------------------|
| < 640px mobile | Single column, stacked | Bottom tabs + header Call | Hidden | Sticky bottom, never dismissible |
| 640–1024px tablet | 2-column grids | Top navbar | Collapsible | Header strip |
| > 1024px desktop | 3–4 column grids | Top navbar | Fixed/expandable | Header strip |

**Mobile adaptations:** full-width cards · bottom-sheet filters/dropdowns · tables → card lists · modals full-screen · search full-screen · tap targets ≥ 44px · forms reserve space above EmergencyCallBar.

---

# 8. Data Visualization (Staff)

| Type | Use case |
|------|----------|
| Line | Appointments/day, intake volume |
| Bar | Rx turnaround, catalogue sales |
| Donut | Order status mix |

Series colour: sage primary; ruby only for emergency-related highlights (e.g. emergency call volume). Pattern fills as colour alternative; screen reader labels; keyboard navigation between points.

---

# 9. Animations & Transitions

| Transition | Duration | Easing | Usage |
|------------|----------|--------|-------|
| Fade In | 200ms | ease-out | Page content |
| Slide Left/Right | 250ms | ease-in-out | Navigation |
| Crossfade | 300ms | ease-in-out | Theme switch |
| Scale In | 150ms | ease-out | Modal |
| Slide Up/Down | 200ms | ease-out | Dropdown |
| Toast In/Out | 300/250ms | ease-out/in | Notifications |
| Skeleton pulse | 1.5s | ease-in-out | Loading |
| Button press | 100ms | ease-out | Micro-interaction |

**Rules:** honour `prefers-reduced-motion`; no parallax; no attention-grabbing pulse on emergency UI; use transform/opacity only.

---

# 10. Accessibility

**Target:** WCAG 2.1 AA (AAA where feasible on brand pairs).

- Contrast: 4.5:1 normal text, 3:1 large text — both light and dark (see `design-system.md` hexes)
- Keyboard: full operability; visible focus (`ruby-500` / `ruby-400` @40%); skip links; modal focus traps
- Screen readers: landmarks, heading hierarchy, labels, `aria-live` for toasts/status, errors via `aria-describedby`
- Forms: visible labels; error summary + per-field messages; focus first error
- Emergency surfaces: phone reachable without login; works with degraded JS; hours/address without menu
- Motion: reduced-motion respected
- Never colour alone: errors + icons + text; status badges labelled

See `accessibility.md` for the full plan and testing matrix.

---

# 11. Error Handling & Edge Cases

**Network:** offline detection; manual retry; cached emergency info where possible.

**Validation:** inline + summary; suggestions; prevent where possible (input types, masks).

**Server errors:** friendly 404/500/503; navigation preserved; clinic phone; error ID for staff.

**Loading:** skeletons matching content; preserve scroll; no layout shift.

**Empty:** clear message; next steps; help + phone.

**Edge cases:** long content truncate + "read more"; large lists paginate/virtualise; rapid actions debounce + disable during submit; concurrent staff edits via claim indicators.

**Recovery:** retry with backoff; undo where safe (toast 5–10s); version history for content.

---

# 12. Performance Considerations

- Core Web Vitals: LCP prioritises emergency phone/hours markup (not late JS)
- Critical CSS inlined; route-level code splitting; lazy below-fold media
- Images: responsive `srcset`, modern formats, lazy load non-hero
- Caching: CDN (Cloudflare) in front of single-VPS origin; static assets long-cache
- Runtime: debounce inputs; virtualise long staff lists; cleanup listeners
- See `../../deployment-architecture.md` for topology

---

# 13. Internationalization

- String externalisation ready; layouts accommodate 30–50% text expansion
- Phone/address formats per locale; single locale v1 acceptable
- Avoid idioms in emergency copy — plain language is load-bearing

---

# 14. Testing & Quality Assurance

| Layer | Scope |
|-------|-------|
| Unit | Components, form validation, token mapping |
| Integration | Booking, checkout, Rx decision, inbox claim |
| E2E | Emergency call path (tap-to-call present), booking happy path, intake upload, staff Rx approve/reject |
| A11y automated | axe/Lighthouse on critical routes |
| A11y manual | Keyboard-only + screen reader smoke before release |
| Visual | Optional screenshot regression on design-system tokens |

---

# 15. Implementation Guidelines

- **Stack:** Next.js 16 + React 19 + Tailwind CSS 4 + shadcn/ui (`shadcn-tailwind.md`)
- **Tokens:** `design-system.md` is source of truth; stray hex codes are review failures
- **Components:** implement against `ui-components.md` contract
- **Patterns:** emergency layouts follow `patterns-emergency-first.md`
- **Folders:** feature-based in `ruby-veterinary-web-frontend` (`src/app`, `src/components`)
- **Type safety:** TypeScript strict; prop types from this catalog

---

# 16. References to Related Documents

| Document | Relationship |
|----------|-------------|
| `design-system.md` | Design token reference |
| `ui-components.md` | Component library specifications |
| `shadcn-tailwind.md` | Tailwind v4 + shadcn implementation |
| `shadcn-tailwindcss-custom-sizing.md` | Fluid sizing for public surfaces |
| `accessibility.md` | WCAG 2.1 AA plan |
| `patterns-emergency-first.md` | 3-second clarity patterns |
| `ui-generation-prompts/` | Stitch mockup prompts |
| `../prd.md` | Product modules and UX principles |
| `../non-functional-requirements.md` | Accessibility, performance, degradation |
| `../openapi/` | API contracts |
| `../../ruby-veterinary-web-frontend` | Implementation target |

---

# 17. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Initial UI/UX system design |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `design-system.md` | Tokens and brand discipline |
| `ui-components.md` | Component contract |
| `patterns-emergency-first.md` | Emergency layout patterns |
| `accessibility.md` | AA compliance plan |

---

# Acceptance Criteria

- Every screen family in §3–§6 has a corresponding Stitch prompt or implementation ticket
- EmergencyCallBar and header Call present on all owner-facing routes
- No layout uses a full-viewport ruby background
- Destructive staff actions use error tokens, never brand ruby
- Degraded dependencies map to the fallbacks in §6.5

---

# Guiding Principle

> **Design is the clinic's bedside manner in pixels. The phone number always wins the layout fight.**
