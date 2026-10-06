# ruby-veterinary Component Library

> **ruby-veterinary Documentation**
>
> **Document:** UI Component Library
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

Specifies the reusable components built from `design-system.md`: variants, sizes, states, props, and accessibility behaviour. Implementation lives in `ruby-veterinary-web-frontend`; this document is the contract reviewers check against.

---

# Global Rules

- Touch targets ≥ 44×44px on mobile for all interactive elements
- Focus ring always visible (`ruby-500`/`ruby-400`), never `outline: none` without replacement
- Every interactive component works keyboard-only
- Colour is never the only signal for state (icon + text accompany colour)
- Disabled controls remain readable; they are not removed from layout
- Ruby (`ruby-700` light / `ruby-400` dark) is reserved for emergency CTAs, links, and brand accents; everyday primary actions use sage

---

# Primitives

## Button

| Variant | Fill | Text | Usage |
|---------|------|------|-------|
| `primary` | `sage-600` | white | Everyday actions: Book, Submit, Continue, Add to cart |
| `brand-emergency` | `ruby-700` | white | Emergency / Call now only |
| `secondary` | transparent | `sage-700` + `sage-200` border | Supporting actions |
| `ghost` | transparent | `sage-700` | Low-emphasis |
| `danger` | `error-strong` | white | Destructive (reject Rx, cancel order) — not brand ruby |
| `link` | none | `ruby-700` (dark: `ruby-400`) | Inline navigation |

Sizes: sm 36px · md 44px (default) · lg 52px · emergency lg+ emphasised padding.

States: default, hover (darken fill 8%), active (scale 0.98), focus (ring), disabled (opacity 0.5), loading (spinner replaces label, `aria-busy`).

Accessibility: Enter/Space activates; loading announced; emergency buttons include visible phone number text, not icon-only.

## IconButton

Icon-only actions (close, chevron). Required `aria-label`. Same variant colours as Button.

## Input / Textarea / Select

- Height 44px, radius `md`, border `border-secondary`, focus ring
- Labels always visible above fields (no placeholder-only labels)
- Error state: `error-strong` border + error icon + message below (`error` text colour)
- Helper text in `text-tertiary`
- Autocomplete attributes on forms (`email`, `tel`, `name`, `street-address`)

## Checkbox / Radio / Switch

- 20px control, 44px hit area, clear focus ring
- Prescription-related toggles include explanatory microcopy

## Badge / Chip

| Variant | Usage |
|---------|-------|
| neutral | default metadata |
| sage | positive operational state (fulfilled, active) |
| ruby | emergency flag only |
| warning | awaiting prescription, paused |
| error | rejected, failed payment |

Always text + optional icon; colour alone insufficient.

## Card

`bg-secondary` or white on warm sections, radius `lg`, shadow `sm`; hover shadow `md` only on clickable cards. Optional warm background variant (`bg-warm`) for section alternation.

## Modal / Dialog

Radius `xl`, shadow `lg`, backdrop `rgba(23,19,16,0.45)`. Focus trapped; Escape closes; returns focus to trigger. Prescription decision modals show pet + order context before destructive actions.

## Toast / Alert

Success `success` tokens · info `info` · error `error` tokens + icon · critical (emergency detection) `ruby` accent bar + alert icon + text.

## Table (staff)

Sticky header, zebra optional (`bg-warm`), row actions as ghost buttons, pagination cursor-based. Mobile: card-list fallback — never horizontal scroll without affordance.

## Skeleton

`bg-tertiary` shimmer bars matching content shape; respects reduced motion (static blocks).

## Empty State

Illustration optional, title, plain-language explanation, primary action (sage), secondary link, and where relevant the clinic phone fallback.

---

# Composite Components

## EmergencyCallBar

Sticky bottom bar (mobile) / header strip (desktop):

- Left: phone icon + clinic emergency number (tap-to-call `tel:`)
- Right: optional "Call now" `brand-emergency` button
- Appears on all owner-facing routes; never dismissible on mobile
- Dark mode: elevated surface + inverse text

## StickyContactHeader

Compact header: logo, Call (ruby, tel link), Hours snippet, Menu. On scroll: emergency number remains visible.

## AppointmentForm

Step-aware form: contact details, pet details, reason (required textarea), preferred window (date + time slots), urgency select (routine/soon/urgent — labelled as a hint, not clinical triage). Submit = sage primary. Inline errors per field + error summary at top on submit failure.

## IntakeStepper

Progress indicator (3–4 steps), current step highlighted sage, completed steps check icon. Upload step shows accepted types (`PDF`, `JPEG`), 10 MB cap, per-file progress, success per file.

## ProductCard / VariantPicker

Card: image, name, category chip, "From" price, Rx badge when `requires_prescription`. VariantPicker: pill buttons for size/pack; disabled variants visibly marked; pet-claim selector appears when Rx item.

## CartLine / CheckoutSummary

Line: variant, qty stepper, line total, pet association chip for Rx lines. Summary: subtotal, tax, shipping or pickup (0), grand total, prescription-hold notice block (warning tokens + icon + explanation) when Rx items present.

## RxStatusBadge

`pending` (warning) · `approved` (sage) · `rejected` (error + reason tooltip) · `queried` (info). Owner-facing copy is plain language, never internal codes alone.

## ReviewQueueRow

Queue table row: pet name, owner, requested time, emergency pin (ruby badge), status, claim/decision actions. Claimed-by indicator for concurrent agents.

## DecisionPanel (vet)

Pet + prescriber context, order lines, history upload links, then actions: Approve (sage), Query (info), Reject (danger + required reason fields). Every action writes audit entry (silent to user; visible in audit view).

## InboxConversation

Message timeline (owner left, clinic right), claim button when `awaiting_handover`, composer (enabled only when claimed by current user or supervisor), bot-paused badge during handover.

## NewsletterBlock

Email input + sage submit, consent microcopy, success state with plain next-steps (confirm email).

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `design-system.md` | Tokens behind every variant |
| `accessibility.md` | Interaction requirements |
| `patterns-emergency-first.md` | Layout patterns using these components |
| `ui-generation-prompts/` | Visual generation of these components |
| `../api-specification.md` | Data contracts feeding forms and tables |

---

# Acceptance Criteria

- Every variant above is implemented or explicitly deferred with a ticket
- Emergency CTAs use `brand-emergency`; no routine button uses ruby
- Form errors always show icon + text + field association
- Staff tables degrade to card lists on mobile

---

# Guiding Principle

> **Components exist so staff stop inventing buttons. If a flow needs a new variant, the design system needs a rule, not a one-off.**
