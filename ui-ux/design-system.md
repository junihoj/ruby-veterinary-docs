# ruby-veterinary Design System

> **ruby-veterinary Documentation**
>
> **Document:** Design System
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

The canonical design tokens and visual rules for ruby-veterinary: brand colours (ruby + white with a soft sage secondary), typography, spacing, radius, shadows, motion, and theming for light and dark. Frontend theming in `ruby-veterinary-web-frontend` maps these tokens into Tailwind CSS 4 `@theme` variables in `globals.css`.

---

# Design Principles

1. **Emergency before commerce** — the clinic phone, hours, and address win every layout fight
2. **Ruby used sparingly** — brand ruby marks emergency actions, links, and key brand touches; large ruby fills feel clinical, not caring
3. **Warm and trustworthy** — white and warm off-white surfaces; muted sage softens the palette into "vet clinic", not "blood bank"
4. **Errors are not brand buttons** — form errors use a distinct orange-leaning red plus icon and text
5. **Mobile-first, WCAG 2.1 AA** — thumb-friendly targets, high contrast, keyboard paths; the floor, not an audit afterthought
6. **3-second clarity** — any page surfaces emergency instructions, location, and phone within three seconds

---

# Colour System

## Brand Ruby Discipline

| Use | Colour | Never |
|-----|--------|-------|
| Emergency / Call now CTAs | `ruby-700` filled, white text | Ordinary "Add to cart" buttons |
| Links, key brand accents | `ruby-700` | Body text blocks |
| Hover/active on brand elements | `ruby-600` | Full-width hero backgrounds |
| Focus ring | `ruby-500` | — |
| Everyday primary actions (book, submit, continue) | `sage-600` filled, white text | Ruby |
| Form errors, destructive confirms | `error-600` + icon + text | Brand ruby |

## Core Colours (Light Mode)

| Token | Hex | Description |
|-------|-----|-------------|
| `--color-ruby-50` | `#FFF5F7` | Soft ruby wash (badges, selected bg) |
| `--color-ruby-100` | `#FFE4EC` | Light ruby surface |
| `--color-ruby-200` | `#FFC2D5` | Borders on ruby surfaces |
| `--color-ruby-300` | `#FF8FAD` | Decorative |
| `--color-ruby-400` | `#F53D6D` | Accents, icons |
| `--color-ruby-500` | `#E0115F` | Brand bright — accents, focus, borders |
| `--color-ruby-600` | `#C00E52` | Hover for primary interactive |
| `--color-ruby-700` | `#9B111E` | **Primary interactive** — emergency CTAs, links |
| `--color-ruby-800` | `#7B0E1B` | Active/pressed |
| `--color-ruby-900` | `#5A0A14` | Deep brand text |

Contrast (light): white on `ruby-700` ≈ 8.4:1 (AAA); `ruby-700` text on white ≈ 8.4:1. `ruby-500` on white ≈ 4.8:1 (AA normal text) — prefer `ruby-700` for body-size links.

## Soft Secondary — Sage (Everyday Actions)

| Token | Hex | Description |
|-------|-----|-------------|
| `--color-sage-50` | `#F4F8F5` | Soft surface |
| `--color-sage-100` | `#E3EDE6` | Panels |
| `--color-sage-200` | `#C7DCCF` | Borders |
| `--color-sage-300` | `#A3C4B0` | Dark-mode interactive |
| `--color-sage-400` | `#7BA88C` | Accents |
| `--color-sage-500` | `#5C7C6E` | Brand secondary |
| `--color-sage-600` | `#4A6659` | **Primary buttons** (light), hover |
| `--color-sage-700` | `#3B5349` | Strong headings/secondary links |
| `--color-sage-800` | `#2E4239` | Dark accents |
| `--color-sage-900` | `#1F2D27` | Deep sage text |

Contrast: white on `sage-600` ≈ 5.9:1 (AA+); `sage-700` on white ≈ 7.2:1.

## Surfaces & Neutrals (Light)

| Token | Hex | Description |
|-------|-----|-------------|
| `--color-bg-primary` | `#FFFFFF` | Page background |
| `--color-bg-warm` | `#FAF7F5` | Warm off-white alternate sections |
| `--color-bg-secondary` | `#F6F2EF` | Cards/panels |
| `--color-bg-tertiary` | `#EFE9E4` | Subtle fills |
| `--color-bg-elevated` | `#FFFFFF` | Modals, popovers |
| `--color-border-primary` | `#E7DFD8` | Default borders |
| `--color-border-secondary` | `#D4C8BE` | Stronger borders |
| `--color-text-primary` | `#1F1A17` | Body (warm near-black) |
| `--color-text-secondary` | `#5C534C` | Secondary text |
| `--color-text-tertiary` | `#8A7F77` | Muted, captions |
| `--color-text-inverse` | `#FFFFFF` | On coloured fills |
| `--color-text-link` | `#9B111E` | Links |

## Semantic Colours (Light)

| Token | Hex | Description |
|-------|-----|-------------|
| `--color-success` | `#2F7D5A` | Success text/icons |
| `--color-success-bg` | `#EAF4EF` | |
| `--color-success-border` | `#B7DCC7` | |
| `--color-warning` | `#B7791F` | Warnings |
| `--color-warning-bg` | `#FBF3E4` | |
| `--color-warning-border` | `#E8C98A` | |
| `--color-error` | `#B8431F` | **Error text** (orange-leaning, AA on white ≈ 5.5:1) |
| `--color-error-strong` | `#D65328` | Error icons/accents |
| `--color-error-bg` | `#FDF0E8` | Error surfaces |
| `--color-error-border` | `#F5C9B4` | |
| `--color-info` | `#2B6CB0` | Informational |
| `--color-info-bg` | `#EBF2FA` | |
| `--color-info-border` | `#B8D0EA` | |

Error never uses brand ruby. Destructive button fill: `error-strong` with white text (contrast ≈ 4.6:1 — acceptable for large button text; pair with icon + label always).

## Surfaces & Neutrals (Dark)

| Token | Hex | Description |
|-------|-----|-------------|
| `--color-bg-primary` | `#171310` | Warm dark page |
| `--color-bg-secondary` | `#1F1A16` | Cards |
| `--color-bg-tertiary` | `#2A2420` | Subtle |
| `--color-bg-elevated` | `#241E1A` | Modals |
| `--color-border-primary` | `#3A322C` | |
| `--color-border-secondary` | `#4C423A` | |
| `--color-text-primary` | `#F7F3F0` | |
| `--color-text-secondary` | `#C4B8AE` | |
| `--color-text-tertiary` | `#94887D` | |
| `--color-text-inverse` | `#171310` | On light fills if needed |

## Semantic Colours (Dark)

| Token | Hex | Description |
|-------|-----|-------------|
| `--color-ruby-400` | `#F53D6D` | **Primary interactive on dark** (links, emergency CTAs) |
| `--color-ruby-500` | `#E0115F` | Accents |
| `--color-sage-300` | `#A3C4B0` | **Primary buttons on dark** |
| `--color-sage-400` | `#7BA88C` | Hover |
| `--color-success` | `#4ADE9B` | |
| `--color-success-bg` | `#123528` | |
| `--color-warning` | `#E3B341` | |
| `--color-warning-bg` | `#3B2E10` | |
| `--color-error` | `#F0754A` | Orange-leaning on dark |
| `--color-error-strong` | `#E85C30` | Destructive fills |
| `--color-error-bg` | `#3B1D10` | |
| `--color-info` | `#6BA3D6` | |
| `--color-info-bg` | `#15273A` | |

Focus ring on dark: `ruby-400` at 40% alpha, 3px.

---

# Typography

| Token | Value | Usage |
|-------|-------|-------|
| `--font-sans` | Inter, system-ui, sans-serif | UI |
| `--font-serif` | Merriweather, Georgia, serif | Blog article bodies |
| `--font-mono` | JetBrains Mono, ui-monospace, monospace | Code, SKUs |

| Scale | Size / Line | Usage |
|-------|-------------|-------|
| Display | 48px / 56px | Hero (used sparingly; emergency CTAs take priority) |
| H1 | 36px / 44px | Page titles |
| H2 | 28px / 36px | Section headings |
| H3 | 22px / 30px | Subsections |
| H4 | 18px / 26px | Card headings |
| Body large | 18px / 28px | Emphasised body |
| Body | 16px / 24px | Default |
| Body small | 14px / 20px | Helper text |
| Caption | 12px / 16px | Labels, timestamps |

Headings: weight 600, tracking −0.01em. Body: weight 400. Minimum interactive text 16px on mobile forms.

---

# Spacing, Radius, Shadow, Motion

**Spacing** (4px base): 0, 2, 4, 6, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96

**Radius:** sm 4px (badges), md 8px (buttons/inputs), lg 12px (cards), xl 16px (modals), full 9999px (avatars, pills)

**Shadow:** sm `0 1px 3px rgba(31,26,23,.08)` · md `0 4px 12px rgba(31,26,23,.10)` · lg `0 12px 28px rgba(31,26,23,.14)` · dark mode: increase opacity to 0.35+

**Motion:** instant 75ms · fast 150ms · normal 200ms · slow 300ms · easing `cubic-bezier(0.4, 0, 0.2, 1)` · respect `prefers-reduced-motion`

**Z-index:** base 0 · sticky 100 · header 110 · dropdown 200 · overlay 400 · modal 500 · toast 600

**Breakpoints:** sm 640 · md 768 · lg 1024 · xl 1280

---

# Tailwind 4 Token Mapping (`globals.css`)

```css
@import "tailwindcss";

:root {
  --color-bg-primary: #ffffff;
  --color-bg-warm: #faf7f5;
  --color-ruby-700: #9b111e;
  --color-ruby-500: #e0115f;
  --color-sage-600: #4a6659;
  --color-error: #b8431f;
  /* …full tables above map 1:1 to --color-* tokens */
}

@theme inline {
  --color-background: var(--color-bg-primary);
  --color-primary: var(--color-sage-600);      /* everyday actions */
  --color-brand: var(--color-ruby-700);        /* emergency + links */
  --color-surface-warm: var(--color-bg-warm);
  --font-sans: var(--font-geist-sans);
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-primary: #171310;
    --color-ruby-700: #f53d6d; /* interactive ruby on dark */
    --color-sage-600: #a3c4b0;
  }
}
```

Dark mode is first-class: both themes ship in v1. Follow OS preference; staff back office may offer an explicit toggle stored in localStorage.

---

# Iconography

- Stroke icons, 24px grid, 1.75px stroke (Lucide or equivalent)
- Emergency phone icon always paired with visible number text — never icon-only for the main clinic line
- Error icons mandatory beside error text (colour is not the only signal)

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `shadcn-tailwind.md` | Tailwind v4 `@theme` implementation of these tokens |
| `shadcn-tailwindcss-custom-sizing.md` | Fluid type/spacing derived from these tokens |
| `ui-components.md` | Component variants built from these tokens |
| `design.md` | System design assembling components from these tokens |
| `accessibility.md` | Contrast and interaction requirements |
| `patterns-emergency-first.md` | Layout patterns for the 3-second rule |
| `ui-generation-prompts/` | Stitch prompts encoding this system |
| `../non-functional-requirements.md` | WCAG 2.1 AA, mobile-first |
| `../prd.md` | UX principles |

---

# Acceptance Criteria

- Every token above exists in `globals.css` `@theme` before themed components merge
- Ruby appears only on emergency CTAs, links, and documented brand accents
- All normal-text colour pairs meet WCAG AA (4.5:1); large text meets 3:1
- Light and dark both pass the same contrast checks

---

# Guiding Principle

> **Ruby is the emergency colour, not the everything colour. Sage carries the calm work; white and warm off-white carry the trust.**
