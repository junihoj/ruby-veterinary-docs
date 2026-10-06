# ruby-veterinary shadcn/ui + Tailwind CSS v4 — Custom Fluid Sizing

> **ruby-veterinary Documentation**
>
> **Document:** shadcn/ui + Tailwind CSS v4 — Custom Fluid Sizing
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** UI Standard
>
> **References:** `design-system.md`, `design.md`, `ui-components.md`, `shadcn-tailwind.md`

---

# Purpose

Fluid (`clamp()`-based) typography and semantic spacing layered on the **same** brand tokens as `shadcn-tailwind.md`. Prefer this config for public marketing surfaces (homepage hero, service pages, blog). Staff back office may use the fixed scale from `shadcn-tailwind.md`. Colour hexes are identical in both docs — `design-system.md` remains the source of truth.

**Ruby discipline:** unchanged — `brand` (ruby) for emergency CTAs and links only; `primary` (sage) for everyday actions; `destructive` = error-strong, never ruby.

---

# 1. Installation & Setup

Same as `shadcn-tailwind.md` §1 (pnpm, `shadcn@latest init` New York/Slate/CSS variables, component add list, lucide-react + cva + clsx + tailwind-merge). See that file for commands.

---

# 2. Tailwind CSS v4 Theme — Fluid Configuration

## 2.1 Main CSS File (`app/globals.css`)

```css
@import "tailwindcss";

/* ============================================
   RUBY-VETERINARY — FLUID TAILWIND v4 THEME
   Colour tokens identical to design-system.md
   Fluid type/spacing for public surfaces
   ============================================ */

:root {
  /* RUBY RAMP */
  --color-ruby-50: #FFF5F7;
  --color-ruby-100: #FFE4EC;
  --color-ruby-200: #FFC2D5;
  --color-ruby-300: #FF8FAD;
  --color-ruby-400: #F53D6D;
  --color-ruby-500: #E0115F;
  --color-ruby-600: #C00E52;
  --color-ruby-700: #9B111E;
  --color-ruby-800: #7B0E1B;
  --color-ruby-900: #5A0A14;

  /* SAGE RAMP */
  --color-sage-50: #F4F8F5;
  --color-sage-100: #E3EDE6;
  --color-sage-200: #C7DCCF;
  --color-sage-300: #A3C4B0;
  --color-sage-400: #7BA88C;
  --color-sage-500: #5C7C6E;
  --color-sage-600: #4A6659;
  --color-sage-700: #3B5349;
  --color-sage-800: #2E4239;
  --color-sage-900: #1F2D27;

  /* SURFACES (light) */
  --color-bg-primary: #FFFFFF;
  --color-bg-warm: #FAF7F5;
  --color-bg-secondary: #F6F2EF;
  --color-bg-tertiary: #EFE9E4;
  --color-bg-elevated: #FFFFFF;
  --color-border-primary: #E7DFD8;
  --color-border-secondary: #D4C8BE;
  --color-text-primary: #1F1A17;
  --color-text-secondary: #5C534C;
  --color-text-tertiary: #8A7F77;
  --color-text-inverse: #FFFFFF;
  --color-text-link: #9B111E;

  /* SEMANTIC (light) */
  --color-success: #2F7D5A;
  --color-success-bg: #EAF4EF;
  --color-success-border: #B7DCC7;
  --color-warning: #B7791F;
  --color-warning-bg: #FBF3E4;
  --color-warning-border: #E8C98A;
  --color-error: #B8431F;
  --color-error-strong: #D65328;
  --color-error-bg: #FDF0E8;
  --color-error-border: #F5C9B4;
  --color-info: #2B6CB0;
  --color-info-bg: #EBF2FA;
  --color-info-border: #B8D0EA;

  --color-focus-ring: rgba(224, 17, 95, 0.35);
  --color-backdrop: rgba(0, 0, 0, 0.45);

  /* FONTS */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-serif: 'Merriweather', Georgia, serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  /* ------------------------------------------
     FLUID TYPOGRAPHY (clamp)
     Min sizes respect design-system.md floors
     ------------------------------------------ */
  --text-display-large: clamp(2.75rem, 5vw, 3rem);
  --text-display-medium: clamp(2.25rem, 4vw, 2.75rem);
  --text-display-small: clamp(2rem, 3.5vw, 2.25rem);

  --text-headline-large: clamp(1.875rem, 3vw, 2.25rem);
  --text-headline-medium: clamp(1.5rem, 2.5vw, 1.75rem);
  --text-headline-small: clamp(1.375rem, 2vw, 1.375rem);

  --text-title-large: clamp(1.125rem, 1.5vw, 1.125rem);
  --text-title-medium: clamp(1rem, 1.2vw, 1rem);
  --text-title-small: clamp(0.9375rem, 1vw, 0.9375rem);

  --text-body-large: clamp(1.0625rem, 1vw, 1.125rem);
  --text-body-medium: clamp(1rem, 0.8vw, 1rem);
  --text-body-small: clamp(0.875rem, 0.7vw, 0.875rem);
  --text-body-xs: clamp(0.75rem, 0.6vw, 0.75rem);

  /* Fixed aliases (staff tables/forms, edge cases) */
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
  --text-5xl: 3rem;

  /* FLUID LINE HEIGHTS */
  --leading-display-large: 1.1;
  --leading-display-medium: 1.1;
  --leading-display-small: 1.15;
  --leading-headline-large: 1.2;
  --leading-headline-medium: 1.25;
  --leading-headline-small: 1.3;
  --leading-title-large: 1.35;
  --leading-title-medium: 1.4;
  --leading-title-small: 1.45;
  --leading-body-large: 1.5;
  --leading-body-medium: 1.5;
  --leading-body-small: 1.5;
  --leading-body-xs: 1.5;

  --tracking-tight: -0.025em;
  --tracking-heading: -0.01em;
  --tracking-normal: 0;
  --tracking-wide: 0.025em;

  /* ------------------------------------------
     FLUID SPACING
     ------------------------------------------ */
  --spacing-hero-padding: clamp(3rem, 8vw, 6rem);
  --spacing-section-gap: clamp(2.5rem, 6vw, 6rem);
  --spacing-section-small: clamp(2rem, 4vw, 4rem);
  --spacing-section-xs: clamp(1.25rem, 2vw, 2.5rem);
  --spacing-page-gutter: clamp(1rem, 3vw, 2rem);

  --spacing-items-lg: clamp(2rem, 3vw, 3rem);
  --spacing-items-md: clamp(1.25rem, 2vw, 2rem);
  --spacing-items-sm: clamp(0.75rem, 1.5vw, 1.25rem);

  --spacing-space-xs: clamp(0.75rem, 1vw, 1rem);
  --spacing-space-2xs: clamp(0.5rem, 0.8vw, 0.75rem);
  --spacing-space-3xs: clamp(0.25rem, 0.5vw, 0.375rem);
  --spacing-space-4xs: 0.125rem;

  /* Fixed aliases (4px grid) */
  --spacing-0: 0;
  --spacing-0\.5: 0.125rem;
  --spacing-1: 0.25rem;
  --spacing-1\.5: 0.375rem;
  --spacing-2: 0.5rem;
  --spacing-3: 0.75rem;
  --spacing-4: 1rem;
  --spacing-5: 1.25rem;
  --spacing-6: 1.5rem;
  --spacing-8: 2rem;
  --spacing-10: 2.5rem;
  --spacing-12: 3rem;
  --spacing-16: 4rem;
  --spacing-20: 5rem;
  --spacing-24: 6rem;

  /* RADIUS */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;
  --radius: 0.5rem;

  /* SHADOWS */
  --shadow-sm: 0 1px 3px rgba(31, 26, 23, 0.08);
  --shadow-md: 0 4px 12px rgba(31, 26, 23, 0.10);
  --shadow-lg: 0 12px 28px rgba(31, 26, 23, 0.14);
  --shadow-focus-ring: 0 0 0 3px rgba(224, 17, 95, 0.35);

  /* Z-INDEX */
  --z-base: 0;
  --z-sticky: 100;
  --z-header: 110;
  --z-dropdown: 200;
  --z-overlay: 400;
  --z-modal: 500;
  --z-toast: 600;

  /* MOTION */
  --duration-instant: 75ms;
  --duration-fast: 150ms;
  --duration-normal: 200ms;
  --duration-slow: 300ms;
  --ease-default: cubic-bezier(0.4, 0, 0.2, 1);

  /* BREAKPOINTS */
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;

  /* TOUCH TARGETS */
  --touch-target-md: 44px;
  --touch-target-lg: 48px;

  /* SHADCN SEMANTIC MAPS (light) */
  --color-background: var(--color-bg-primary);
  --color-foreground: var(--color-text-primary);
  --color-card: var(--color-bg-secondary);
  --color-card-foreground: var(--color-text-primary);
  --color-popover: var(--color-bg-elevated);
  --color-popover-foreground: var(--color-text-primary);
  --color-muted: var(--color-bg-tertiary);
  --color-muted-foreground: var(--color-text-tertiary);
  --color-border: var(--color-border-primary);
  --color-input: var(--color-border-secondary);
  --color-ring: var(--color-ruby-500);
  --color-primary: var(--color-sage-600);
  --color-primary-foreground: #FFFFFF;
  --color-brand: var(--color-ruby-700);
  --color-brand-foreground: #FFFFFF;
  --color-destructive: var(--color-error-strong);
  --color-destructive-foreground: #FFFFFF;
  --color-surface-warm: var(--color-bg-warm);
}

/* ============================================
   DARK MODE
   ============================================ */

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-primary: #171310;
    --color-bg-secondary: #1F1A16;
    --color-bg-tertiary: #2A2420;
    --color-bg-elevated: #241E1A;
    --color-bg-warm: #1F1A16;
    --color-border-primary: #3A322C;
    --color-border-secondary: #4C423A;
    --color-text-primary: #F7F3F0;
    --color-text-secondary: #C4B8AE;
    --color-text-tertiary: #94887D;
    --color-text-inverse: #171310;
    --color-ruby-700: #F53D6D;
    --color-sage-600: #A3C4B0;
    --color-sage-400: #7BA88C;
    --color-success: #4ADE9B;
    --color-success-bg: #123528;
    --color-warning: #E3B341;
    --color-warning-bg: #3B2E10;
    --color-error: #F0754A;
    --color-error-strong: #E85C30;
    --color-error-bg: #3B1D10;
    --color-info: #6BA3D6;
    --color-info-bg: #15273A;
    --color-focus-ring: rgba(245, 61, 109, 0.4);
    --color-backdrop: rgba(0, 0, 0, 0.55);
    --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.35);
    --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.35);
    --shadow-lg: 0 12px 28px rgba(0, 0, 0, 0.40);
    --shadow-focus-ring: 0 0 0 3px rgba(245, 61, 109, 0.4);
  }
}

.dark,
[data-theme="dark"] {
  --color-bg-primary: #171310;
  --color-bg-secondary: #1F1A16;
  --color-bg-tertiary: #2A2420;
  --color-bg-elevated: #241E1A;
  --color-bg-warm: #1F1A16;
  --color-border-primary: #3A322C;
  --color-border-secondary: #4C423A;
  --color-text-primary: #F7F3F0;
  --color-text-secondary: #C4B8AE;
  --color-text-tertiary: #94887D;
  --color-text-inverse: #171310;
  --color-ruby-700: #F53D6D;
  --color-sage-600: #A3C4B0;
  --color-sage-400: #7BA88C;
  --color-success: #4ADE9B;
  --color-success-bg: #123528;
  --color-warning: #E3B341;
  --color-warning-bg: #3B2E10;
  --color-error: #F0754A;
  --color-error-strong: #E85C30;
  --color-error-bg: #3B1D10;
  --color-info: #6BA3D6;
  --color-info-bg: #15273A;
  --color-focus-ring: rgba(245, 61, 109, 0.4);
  --color-backdrop: rgba(0, 0, 0, 0.55);
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.35);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.35);
  --shadow-lg: 0 12px 28px rgba(0, 0, 0, 0.40);
  --shadow-focus-ring: 0 0 0 3px rgba(245, 61, 109, 0.4);
}

@theme inline {
  --color-background: var(--color-background);
  --color-foreground: var(--color-foreground);
  --color-card: var(--color-card);
  --color-card-foreground: var(--color-card-foreground);
  --color-popover: var(--color-popover);
  --color-popover-foreground: var(--color-popover-foreground);
  --color-muted: var(--color-muted);
  --color-muted-foreground: var(--color-muted-foreground);
  --color-border: var(--color-border);
  --color-input: var(--color-input);
  --color-ring: var(--color-ring);
  --color-primary: var(--color-primary);
  --color-primary-foreground: var(--color-primary-foreground);
  --color-brand: var(--color-brand);
  --color-brand-foreground: var(--color-brand-foreground);
  --color-destructive: var(--color-destructive);
  --color-destructive-foreground: var(--color-destructive-foreground);
  --color-surface-warm: var(--color-surface-warm);
  --color-ruby-700: var(--color-ruby-700);
  --color-ruby-500: var(--color-ruby-500);
  --color-sage-600: var(--color-sage-600);
  --color-error: var(--color-error);
  --color-error-strong: var(--color-error-strong);
  --color-success: var(--color-success);
  --color-warning: var(--color-warning);
  --color-info: var(--color-info);
  --font-sans: var(--font-sans);
  --font-serif: var(--font-serif);
  --font-mono: var(--font-mono);
  --text-display-large: var(--text-display-large);
  --text-display-medium: var(--text-display-medium);
  --text-display-small: var(--text-display-small);
  --text-headline-large: var(--text-headline-large);
  --text-headline-medium: var(--text-headline-medium);
  --text-headline-small: var(--text-headline-small);
  --text-title-large: var(--text-title-large);
  --text-title-medium: var(--text-title-medium);
  --text-title-small: var(--text-title-small);
  --text-body-large: var(--text-body-large);
  --text-body-medium: var(--text-body-medium);
  --text-body-small: var(--text-body-small);
  --text-body-xs: var(--text-body-xs);
}

/* ============================================
   KEYFRAMES
   ============================================ */

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slide-up {
  from { transform: translateY(8px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes scale-in {
  from { transform: scale(0.95); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

@keyframes press {
  from { transform: scale(1); }
  to { transform: scale(0.98); }
}

/* ============================================
   ACCESSIBILITY
   ============================================ */

.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  z-index: var(--z-toast);
  padding: var(--spacing-space-xs) var(--spacing-6);
  background: var(--color-primary);
  color: var(--color-primary-foreground);
  font-weight: 600;
  text-decoration: none;
  border-radius: 0 0 var(--radius-md) var(--radius-md);
}

.skip-link:focus {
  top: 0;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  clip-path: inset(50%);
  white-space: nowrap;
  border-width: 0;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* ============================================
   FLUID SPACING UTILITIES
   ============================================ */

@utility spacing-hero-padding {
  padding-block: var(--spacing-hero-padding);
}

@utility spacing-section-gap {
  padding-block: var(--spacing-section-gap);
}

@utility spacing-section-small {
  padding-block: var(--spacing-section-small);
}

@utility spacing-section-xs {
  padding-block: var(--spacing-section-xs);
}

@utility spacing-page-gutter {
  padding-inline: var(--spacing-page-gutter);
}

@utility gap-items-lg {
  gap: var(--spacing-items-lg);
}

@utility gap-items-md {
  gap: var(--spacing-items-md);
}

@utility gap-items-sm {
  gap: var(--spacing-items-sm);
}

@utility p-space-xs {
  padding: var(--spacing-space-xs);
}

@utility p-space-2xs {
  padding: var(--spacing-space-2xs);
}

@utility p-space-3xs {
  padding: var(--spacing-space-3xs);
}

@utility touch-target-md {
  min-width: var(--touch-target-md);
  min-height: var(--touch-target-md);
}

@utility touch-target-lg {
  min-width: var(--touch-target-lg);
  min-height: var(--touch-target-lg);
}

@utility skeleton-shimmer {
  background: linear-gradient(
    90deg,
    var(--color-muted) 25%,
    var(--color-bg-tertiary) 50%,
    var(--color-muted) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
}

.container-clinic {
  width: 100%;
  margin-left: auto;
  margin-right: auto;
  padding-left: var(--spacing-page-gutter);
  padding-right: var(--spacing-page-gutter);
  max-width: var(--breakpoint-xl);
}
```

---

# 3. shadcn cva — Fluid Tokens

```tsx
// Button sizes use fluid type + space tokens
const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-body-small font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        primary: "bg-primary text-primary-foreground hover:bg-sage-700 active:bg-sage-800",
        "brand-emergency": "bg-brand text-brand-foreground hover:bg-ruby-600 active:bg-ruby-800",
        secondary: "border border-sage-200 bg-transparent text-sage-700 hover:bg-sage-50 dark:border-sage-800 dark:text-sage-300",
        ghost: "text-sage-700 hover:bg-sage-50 dark:text-sage-300",
        danger: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        link: "text-brand underline-offset-4 hover:underline dark:text-ruby-400",
      },
      size: {
        sm: "h-9 px-space-2xs text-body-xs",
        md: "h-11 px-space-xs text-body-small",
        lg: "h-12 px-space-sm text-body-medium",
        emergency: "h-12 px-space-sm text-body-medium",
        icon: "h-11 w-11",
      },
    },
    defaultVariants: { variant: "primary", size: "md" },
  }
)
```

Card padding variants: `p-space-2xs` / `p-space-xs` / `p-space-sm`. Alert padding: `p-space-xs`. Badge: `px-space-3xs py-space-4xs`.

---

# 4. Usage Examples

```tsx
// Emergency hero — display type + hero padding + brand CTA + sage Book
<section className="spacing-hero-padding bg-surface-warm">
  <div className="container-clinic">
    <h1 className="text-display-medium font-bold tracking-tight text-foreground">
      Calm care for every life stage
    </h1>
    <p className="mt-space-sm text-body-large text-muted-foreground max-w-2xl">
      Same-day sick visits, wellness plans, and a 24h emergency line.
    </p>
    <div className="mt-space-xs flex flex-wrap gap-items-sm">
      <Button variant="brand-emergency" size="emergency">
        Call now — 01632 960245
      </Button>
      <Button variant="primary" size="lg">
        Book appointment
      </Button>
    </div>
  </div>
</section>

// Service section
<section className="spacing-section-gap">
  <div className="container-clinic">
    <h2 className="text-headline-large font-semibold tracking-heading text-foreground">
      Services
    </h2>
    <div className="mt-space-sm grid gap-items-md md:grid-cols-2 lg:grid-cols-3">
      {/* service cards — text-title-large / text-body-medium */}
    </div>
  </div>
</section>

// Blog article body (serif)
<article className="font-serif text-body-large leading-body-large">
  {/* article content — Merriweather */}
</article>
```

---

# 5. Fluid Sizing Quick Reference

## 5.1 Typography Scale

| Category | Token | Min | Max | LH | Weight | Usage |
|----------|-------|-----|-----|----|--------|-------|
| Display | `text-display-large` | 44px | 48px | 1.1 | bold | Homepage hero (sparingly) |
| Display | `text-display-medium` | 36px | 44px | 1.1 | bold | Hero variants |
| Headline | `text-headline-large` | 30px | 36px | 1.2 | semibold | H1 page titles |
| Headline | `text-headline-medium` | 24px | 28px | 1.25 | semibold | H2 sections |
| Headline | `text-headline-small` | 22px | 22px | 1.3 | medium | H3 |
| Title | `text-title-large` | 18px | 18px | 1.35 | medium | H4 card headings |
| Body | `text-body-large` | 17px | 18px | 1.5 | regular | Article leads |
| Body | `text-body-medium` | 16px | 16px | 1.5 | regular | Default body — 16px floor |
| Body | `text-body-small` | 14px | 14px | 1.5 | regular | Helper text |
| Body | `text-body-xs` | 12px | 12px | 1.5 | regular | Labels, timestamps — never below 12px |

**WCAG:** interactive text on mobile forms stays ≥16px (`text-body-medium` minimum). Caption floor 12px.

## 5.2 Spacing Scale

| Category | Token | Min | Max | Usage |
|----------|-------|-----|-----|-------|
| Layout | `spacing-hero-padding` | 48px | 96px | Emergency hero vertical |
| Layout | `spacing-section-gap` | 40px | 96px | Between major sections |
| Layout | `spacing-section-small` | 32px | 64px | Related sections |
| Layout | `spacing-section-xs` | 20px | 40px | Subsections |
| Layout | `spacing-page-gutter` | 16px | 32px | Horizontal page margins |
| Items | `spacing-items-lg` | 32px | 48px | Large grid gaps |
| Items | `spacing-items-md` | 20px | 32px | Card grids |
| Items | `spacing-items-sm` | 12px | 20px | Form fields, nav links |
| Detail | `spacing-space-xs` | 12px | 16px | Card internals, button pad |
| Detail | `spacing-space-2xs` | 8px | 12px | Icon + text pairs |
| Detail | `spacing-space-3xs` | 4px | 6px | Badges, tight gaps |

## 5.3 When to Use Which Token

| Context | Typography | Spacing |
|---------|------------|---------|
| Homepage emergency hero | `text-display-medium` | `spacing-hero-padding` |
| Service page section | `text-headline-medium` | `spacing-section-gap` |
| Product card title | `text-title-large` | `gap-items-md` |
| Form fields | `text-body-medium` | `gap-items-sm` |
| Blog article | `font-serif text-body-large` | `spacing-section-small` |
| Staff table cells | fixed `text-sm` | fixed `p-4` |
| Rx decision modal | `text-body-medium` | `p-space-xs` |
| Timestamps | `text-body-xs` | `p-space-3xs` |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `shadcn-tailwind.md` | Fixed-size sibling config (same colours) |
| `design-system.md` | Authoritative token tables |
| `ui-components.md` | Component contract |
| `design.md` | Screen layouts |
| `accessibility.md` | Contrast and a11y requirements |

---

# Acceptance Criteria

- Colour hexes identical to `shadcn-tailwind.md` / `design-system.md`
- Fluid body tokens never render below 16px on mobile forms
- Fluid display tokens used only on public marketing surfaces; staff UI may use fixed scale
- Both configs honour prefers-reduced-motion and WCAG AA contrast

---

# Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Initial fluid sizing configuration (design-system.md tokens) |

---

# Guiding Principle

> **Fluid where the eye wanders; fixed where the hand works. Tokens never drift from design-system.md.**
