# UI/UX Documentation

> **ruby-veterinary Documentation**
>
> **Document:** UI/UX Documentation Map
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

The interface standard for ruby-veterinary: design tokens (ruby + white brand with sage secondary), component behaviour, accessibility rules, emergency-first patterns, and Stitch generation prompts. Frontend theming implements these documents in Tailwind CSS 4.

---

# Documents

| Document | Contents |
|----------|----------|
| [design-system.md](design-system.md) | Colours, typography, spacing, radius, shadow, motion, Tailwind tokens, light + dark |
| [ui-components.md](ui-components.md) | Full component catalog — variants, sizes, states, props, a11y, domain-specific vet components |
| [design.md](design.md) | UI/UX system design — navigation, public/owner/staff wireframes, responsive, animation |
| [shadcn-tailwind.md](shadcn-tailwind.md) | Tailwind CSS 4 + shadcn/ui fixed `@theme` config, cva variants, token mapping |
| [shadcn-tailwindcss-custom-sizing.md](shadcn-tailwindcss-custom-sizing.md) | Fluid sizing — `clamp()` type/spacing tokens for public surfaces |
| [accessibility.md](accessibility.md) | WCAG 2.1 AA plan, contrast, keyboard, testing |
| [patterns-emergency-first.md](patterns-emergency-first.md) | 3-second clarity patterns, degradation fallbacks |
| [ui-generation-prompts/](ui-generation-prompts/) | Google Stitch prompt files (~57 screens) for mockup generation |

---

# Brand Snapshot

| Role | Token | Usage |
|------|-------|-------|
| Emergency / brand interactive | ruby `#9B111E` (dark `#F53D6D`) | Call CTAs, links, focus, accents |
| Everyday primary action | sage `#4A6659` (dark `#A3C4B0`) | Book, Submit, Add to cart |
| Surfaces | white `#FFFFFF`, warm `#FAF7F5` (dark warm `#171310`) | Page backgrounds |
| Errors | orange-leaning `#B8431F` / `#D65328` | Validation, destructive — never brand ruby |

Ruby discipline: sparingly. Sage carries the calm work.

---

# Intended Audience

Product designers, frontend engineers, and whoever pastes prompts into Stitch. Cross-team contracts with backend live in `../api-specification.md`.

---

# Change Management

Token or pattern changes update `design-system.md` first, then components/prompts in the same pull request. Stray hex codes in frontend code that contradict this directory are review failures.

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../prd.md` | UX principles and modules |
| `../non-functional-requirements.md` | Performance, accessibility, degradation |
| `../vision.md` | Mobile-first, accessible-to-everyone principle |
| `ui-generation-prompts/` | Mockup generation prompts for this system |
| `../../ruby-veterinary-web-frontend` | Implementation target |

---

# Acceptance Criteria

- Every document in this table exists and is non-empty
- Frontend `globals.css` maps tokens from `design-system.md`
- Prompts directory contains master + flow files listed in its README

---

# Guiding Principle

> **Design is a clinic's bedside manner in pixels. Warm surfaces, honest colours, and the phone number always within reach.**
