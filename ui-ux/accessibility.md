# Accessibility Requirements

> **ruby-veterinary Documentation**
>
> **Document:** Accessibility Requirements
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** UI Standard

---

# Purpose

The WCAG 2.1 AA conformance plan for ruby-veterinary: colour contrast, keyboard paths, motion, mobile ergonomics, and the emergency-surface rules that make the product usable under stress. This is a binding floor (`../non-functional-requirements.md` §3), not an audit afterthought.

---

# Target

**WCAG 2.1 Level AA** for all public surfaces and the staff back office. Critical owner flows (emergency info, call, booking, checkout) are additionally usability-tested with keyboard-only and slow-network profiles.

---

# Colour and Contrast

| Requirement | Rule |
|-------------|------|
| Normal text | ≥ 4.5:1 against background (body, labels, helper text) |
| Large text (≥24px or 19px bold) | ≥ 3:1 |
| UI components & focus indicators | ≥ 3:1 |
| Brand pairing | `ruby-700`/white ≈ 8.4:1; sage-600/white ≈ 5.9:1 (both pass) |
| Error communication | Orange-leaning error tokens + icon + text — never ruby-only |
| Emergency CTA | Ruby fill + white text + visible phone number label |
| Dark mode | Same ratios against dark surfaces; interactive ruby lightens to `#F53D6D` |

Automated checks (axe/Lighthouse) run in CI for critical routes; manual contrast review required for any new token pair.

---

# Keyboard and Focus

- Every interactive element reachable by Tab in logical order
- Visible focus ring on all themes; skip-to-content link on every page
- Modals trap focus and restore it to the trigger on close
- Emergency CallBar reachable by keyboard and announced by screen readers
- No keyboard traps in checkout, intake, or inbox composers

---

# Screen Reader Semantics

- Landmarks: `header`, `nav`, `main`, `footer`; single `h1` per page
- Form fields associated with labels; errors linked via `aria-describedby`
- Status changes (cart updates, Rx status, claim state) announced via live regions where non-blocking
- Decorative icons `aria-hidden`; meaningful icons have accessible names
- Blog images require alt text (enforced at CMS publish time — `data-model/publishing.md`)

---

# Motion and Vision

- All animations respect `prefers-reduced-motion: reduce`
- No content flashes more than 3 times per second
- Text remains selectable and zoomable; reflow at 320px width without two-dimensional scrolling (AA 1.4.10)

---

# Mobile Ergonomics (Emergency Context)

- Tap targets ≥ 44×44px; spacing between targets ≥ 8px
- Emergency number tappable (`tel:`) and visible without opening a menu
- Forms: correct input types (`tel`, `email`, `inputmode`), no tiny date widgets where native is better
- Sticky CallBar must not obscure form submit controls (padding-bottom on main)

---

# Emergency-Surface Specifics

| Rule | Rationale |
|------|-----------|
| Phone, hours, address render without JavaScript-dependent data fetch where possible | Slow networks |
| Works with no session; never redirects to login | Emergency seeker persona |
| Fallback text + phone shown when WhatsApp/bot/payment unavailable | Graceful degradation NFR |
| Urgency labels in intake are owner hints only — no clinical colour coding that implies diagnosis | Safety |
| Out-of-hours flow points exclusively to emergency care, not to a dead end | Vision principles |

---

# Staff Back Office

- Same contrast/focus rules as public site
- Inbox and queue tables operable at 200% zoom
- Decision modals: destructive reject path requires keyboard-accessible reason entry; focus starts on non-destructive control
- Colour-blind safe status badges (icon + text always)

---

# Testing Plan

| Layer | What | When |
|-------|------|------|
| CI automated | axe on critical routes (home, services, product, cart, checkout, intake) | Every PR |
| Keyboard manual | Tab through all owner flows | Before each release |
| Screen reader smoke | VoiceOver or NVDA on emergency bar + intake form | Before launch + quarterly |
| Contrast tokens | Check new token pairs in both themes | Design review |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `design-system.md` | Token contrast values |
| `ui-components.md` | Per-component a11y behaviour |
| `patterns-emergency-first.md` | Layout rules for the 3-second rule |
| `../non-functional-requirements.md` | WCAG 2.1 AA commitment |
| `../vision.md` | Accessible-by-default principle |

---

# Acceptance Criteria

- Lighthouse/axe: zero critical violations on critical routes
- Emergency number reachable by keyboard within first tab stops on every page
- All error states pass colour-blind simulation (icon + text present)
- `prefers-reduced-motion` disables non-essential animation

---

# Guiding Principle

> **Accessibility is how the clinic treats people on the worst day of their week. Ship it as the default, not the retrofit.**
