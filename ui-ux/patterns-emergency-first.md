# Emergency-First UI Patterns

> **ruby-veterinary Documentation**
>
> **Document:** Emergency-First UI Patterns
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

The layout patterns that enforce the 3-second emergency clarity requirement: what every page must surface, how ruby is used to signal urgency, and how each degraded dependency shows a plain-language fallback. These patterns bind product and frontend work; component variants live in `ui-components.md`.

---

# The 3-Second Rule

Within three seconds of landing on **any** owner-facing page, a visitor must be able to identify:

1. The emergency phone number (tap-to-call)
2. Whether the clinic is open now (or the next opening)
3. The physical location (city/address minimum; full address on contact/hours sections)

If a layout cannot satisfy all three, the layout is wrong — not the rule.

---

# Pattern 1 — EmergencyCallBar (Sticky)

| Attribute | Spec |
|-----------|------|
| Placement | Fixed bottom on mobile; persistent strip in header on desktop |
| Content | Phone icon + emergency number label + optional "Call now" `brand-emergency` button |
| Colour | Ruby accent / white surface; never dismissible on mobile |
| Behaviour | `tel:` link; appears on all public routes including blog and store |
| A11y | Landmark-adjacent announcement; first tab stop after skip-link |

Owner-facing pages reserve `padding-bottom` so forms' submit buttons are never covered.

---

# Pattern 2 — Emergency Hero (Homepage and Contact)

- Homepage: warm off-white or white hero — **no full-width ruby fill**
- Left: clinic name, one-line caring value prop, two CTAs — **Call now** (`brand-emergency`) and **Book appointment** (`primary` sage)
- Right or below: hours snippet with open/closed state + emergency number repeated as text
- Contact page: emergency number is the largest interactive element on the page

---

# Pattern 3 — Degradation Fallbacks

| Dependency down | UI pattern |
|-----------------|------------|
| Payment gateway | Checkout shows `error`/warning banner: "Online payment is temporarily unavailable. Call us to order or pay in clinic — [phone]" + sage Call CTA |
| WhatsApp / bot | Site shows click-to-chat unavailable state: "Message us on WhatsApp when it's working, or call [phone] now" — sticky bar unchanged |
| Email / newsletter | Subscribe form still accepts input; success copy says "we'll confirm by email; if you don't hear from us, call [phone]" |
| Full origin down | CDN-cached emergency strip + hours remain where cached; DNS/emergency page strategy in `../../deployment-architecture.md` |
| Intake forms | "Prefer not to wait online? Call [phone] during opening hours" beneath every form |

Rules for every fallback: plain language, clinic phone present, no dead ends, never a raw error code.

---

# Pattern 4 — Urgency Without Alarm

- Intake urgency select: `routine` / `soon` / `urgent` with helper text "This helps us prioritise. For a life-threatening emergency, call [phone] now"
- Do not colour-code clinical urgency in ruby; ruby stays reserved for the emergency CTA itself
- Bot panic detection raises internal priority; owner-facing copy stays calm: "We've alerted the clinic team. Please call [phone] if this is urgent right now"

---

# Pattern 5 — Hours and Open State

- "Open now · closes 18:00" / "Closed · opens 09:00 tomorrow" chips near contact CTAs
- Holiday hours override standard hours everywhere they appear
- Out-of-hours pages replace generic contact CTAs with a single emergency-forward CTA block

---

# Pattern 6 — Mobile Navigation

- Sticky bottom nav (owner app shell): **Call** (ruby) · **Book** (sage) · **Store** · **Account**
- Menu drawer includes emergency number at top, above navigation links
- Back office uses desktop-first nav; mobile inbox keeps claim/reply actions reachable

---

# Pattern 7 — Confirmation and Reassurance States

After appointment, intake, or order submission:

- Show plain-language "what happens next" (who calls, expected timing)
- Repeat clinic phone as the alternative path
- Prescription orders: explain hold in owner language + how to reach the clinic; never "error" framing for a normal guardrail

---

# Accessibility Tie-In

See `accessibility.md`. Pattern-specific requirements: call bar keyboard reach, fallback banners as `role="status"`, no reliance on colour for open/closed (text labels mandatory).

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `ui-components.md` | EmergencyCallBar, buttons, banners |
| `design-system.md` | Ruby discipline and tokens |
| `../prd.md` | 3-second UX clarity requirement |
| `../non-functional-requirements.md` | Graceful degradation |
| `ui-generation-prompts/01-emergency-homepage.md` | Visual generation of these patterns |

---

# Acceptance Criteria

- Every public route includes the emergency call affordance without scrolling on mobile
- Each degraded dependency in `05-communication-matrix.md` has a mapped UI fallback here
- No layout uses a full-viewport ruby background
- Confirmations always offer the phone path

---

# Guiding Principle

> **Calm surfaces, ruby only where urgency belongs. When anything fails, the phone number is already on screen.**
