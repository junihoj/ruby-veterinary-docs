# ruby-veterinary shadcn/ui + Tailwind CSS v4 Configuration

> **ruby-veterinary Documentation**
>
> **Document:** shadcn/ui + Tailwind CSS v4 Configuration
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** UI Standard
>
> **References:** `design-system.md`, `design.md`, `ui-components.md`, `shadcn-tailwindcss-custom-sizing.md`

---

# Purpose

Implementation-ready shadcn/ui + Tailwind CSS v4 setup for `ruby-veterinary-web-frontend`. Colour tokens map 1:1 from `design-system.md` (source of truth). Use this file for fixed-size theming (public site + staff back office). For fluid `clamp()` typography/spacing on marketing surfaces, see `shadcn-tailwindcss-custom-sizing.md`.

**Ruby discipline:** `brand` (ruby) is reserved for emergency CTAs, links, focus, and documented brand accents. Everyday actions use `primary` (sage). Destructive uses `error-strong` — never brand ruby.

---

# 1. Installation & Setup

## 1.1 Prerequisites

```bash
# ruby-veterinary-web-frontend — Next.js 16 + React 19 + Tailwind CSS 4
cd ruby-veterinary-web-frontend
pnpm install
```

## 1.2 Install shadcn/ui

```bash
pnpm dlx shadcn@latest init
```

When prompted:

- Style: **New York**
- Base color: **Slate** (will override with custom tokens)
- CSS variables: **Yes**

## 1.3 Install shadcn Components

```bash
pnpm dlx shadcn@latest add button input card dialog dropdown-menu \
  tabs accordion avatar badge separator sheet tooltip \
  select textarea checkbox radio-group switch progress \
  skeleton table popover form label calendar toast \
  sonner collapsible
```

## 1.4 Install Dependencies

```bash
pnpm add lucide-react class-variance-authority clsx tailwind-merge
```

Component sources land in `components/ui/`. Utility helper: `lib/utils.ts`.

---

# 2. Tailwind CSS v4 Theme Configuration

Tailwind CSS v4 uses CSS-first configuration via `@theme` in the main CSS file.

## 2.1 Main CSS File (`app/globals.css`)

```css
@import "tailwindcss";

/* ============================================
   RUBY-VETERINARY DESIGN SYSTEM — TAILWIND v4
   Tokens map 1:1 from design-system.md
   ============================================ */

:root {
  /* ------------------------------------------
     RUBY RAMP (brand — emergency + links)
     ------------------------------------------ */
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

  /* ------------------------------------------
     SAGE RAMP (everyday actions)
     ------------------------------------------ */
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

  /* ------------------------------------------
     SURFACES & NEUTRALS (light)
     ------------------------------------------ */
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

  /* ------------------------------------------
     SEMANTIC (light)
     ------------------------------------------ */
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

  /* ------------------------------------------
     FOCUS / OVERLAY
     ------------------------------------------ */
  --color-focus-ring: rgba(224, 17, 95, 0.35);
  --color-backdrop: rgba(0, 0, 0, 0.45);

  /* ------------------------------------------
     TYPOGRAPHY
     ------------------------------------------ */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-serif: 'Merriweather', Georgia, serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
  --text-5xl: 3rem;

  /* ------------------------------------------
     SPACING (4px base)
     ------------------------------------------ */
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

  /* ------------------------------------------
     BORDER RADIUS
     ------------------------------------------ */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;
  --radius: 0.5rem;

  /* ------------------------------------------
     SHADOWS (warm near-black)
     ------------------------------------------ */
  --shadow-sm: 0 1px 3px rgba(31, 26, 23, 0.08);
  --shadow-md: 0 4px 12px rgba(31, 26, 23, 0.10);
  --shadow-lg: 0 12px 28px rgba(31, 26, 23, 0.14);
  --shadow-focus-ring: 0 0 0 3px rgba(224, 17, 95, 0.35);

  /* ------------------------------------------
     Z-INDEX
     ------------------------------------------ */
  --z-base: 0;
  --z-sticky: 100;
  --z-header: 110;
  --z-dropdown: 200;
  --z-overlay: 400;
  --z-modal: 500;
  --z-toast: 600;

  /* ------------------------------------------
     MOTION
     ------------------------------------------ */
  --duration-instant: 75ms;
  --duration-fast: 150ms;
  --duration-normal: 200ms;
  --duration-slow: 300ms;
  --ease-default: cubic-bezier(0.4, 0, 0.2, 1);

  /* ------------------------------------------
     BREAKPOINTS
     ------------------------------------------ */
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;

  /* ------------------------------------------
     TOUCH TARGETS
     ------------------------------------------ */
  --touch-target-md: 44px;
  --touch-target-lg: 48px;

  /* ------------------------------------------
     SHADCN SEMANTIC MAPS (light)
     ------------------------------------------ */
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
   DARK MODE — OS preference (design-system.md)
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

    /* Interactive swaps on dark */
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

/* Staff explicit toggle — same overrides as OS dark */
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
}

/* ============================================
   KEYFRAMES
   ============================================ */

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fade-out {
  from { opacity: 1; }
  to { opacity: 0; }
}

@keyframes slide-up {
  from { transform: translateY(8px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes slide-down {
  from { transform: translateY(-8px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes scale-in {
  from { transform: scale(0.95); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%, 60% { transform: translateX(-4px); }
  40%, 80% { transform: translateX(4px); }
}

@keyframes press {
  from { transform: scale(1); }
  to { transform: scale(0.98); }
}

@keyframes toast-in {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

@keyframes success-check {
  0% { transform: scale(0); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* ============================================
   ACCESSIBILITY UTILITIES
   ============================================ */

.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  z-index: var(--z-toast);
  padding: var(--spacing-4) var(--spacing-6);
  background: var(--color-primary);
  color: var(--color-primary-foreground);
  font-weight: 600;
  text-decoration: none;
  border-radius: 0 0 var(--radius-md) var(--radius-md);
  transition: top var(--duration-fast) var(--ease-default);
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

.focus-visible-ring:focus-visible {
  outline: 2px solid var(--color-ring);
  outline-offset: 2px;
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
   LAYOUT UTILITIES
   ============================================ */

.container-clinic {
  width: 100%;
  margin-left: auto;
  margin-right: auto;
  padding-left: var(--spacing-4);
  padding-right: var(--spacing-4);
  max-width: var(--breakpoint-xl);
}

@media (min-width: 768px) {
  .container-clinic {
    padding-left: var(--spacing-6);
    padding-right: var(--spacing-6);
  }
}

@media (min-width: 1024px) {
  .container-clinic {
    padding-left: var(--spacing-8);
    padding-right: var(--spacing-8);
  }
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

@utility scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

@utility line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

@utility hide-below-md {
  @media (max-width: 767px) {
    display: none;
  }
}

@utility hide-above-md {
  @media (min-width: 768px) {
    display: none;
  }
}
```

---

# 3. shadcn/ui Component Theming

## 3.1 Button Variants

```tsx
// components/ui/button.tsx
import { cva, type VariantProps } from "class-variance-authority"

const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        primary:
          "bg-primary text-primary-foreground hover:bg-sage-700 active:bg-sage-800",
        "brand-emergency":
          "bg-brand text-brand-foreground hover:bg-ruby-600 active:bg-ruby-800",
        secondary:
          "border border-sage-200 bg-transparent text-sage-700 hover:bg-sage-50 dark:border-sage-800 dark:text-sage-300 dark:hover:bg-sage-900/40",
        ghost:
          "text-sage-700 hover:bg-sage-50 dark:text-sage-300 dark:hover:bg-sage-900/40",
        danger:
          "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        link:
          "text-brand underline-offset-4 hover:underline dark:text-ruby-400",
      },
      size: {
        sm: "h-9 px-3 text-xs",
        md: "h-11 px-4",
        lg: "h-12 px-6 text-base",
        emergency: "h-12 px-8 text-base",
        icon: "h-11 w-11",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  }
)
```

**Rules:** `brand-emergency` only for Call now / emergency CTAs. `danger` for reject Rx, cancel order — never brand ruby. Emergency buttons include visible phone number text, not icon-only.

## 3.2 Input Variants

```tsx
const inputVariants = cva(
  "flex w-full rounded-md border bg-background text-foreground placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50",
  {
    variants: {
      variant: {
        outline: "border-input",
        filled: "border-transparent bg-muted",
        underline: "border-0 border-b rounded-none",
      },
      inputSize: {
        sm: "h-9 px-3 text-xs",
        md: "h-11 px-4 text-base",
        lg: "h-12 px-4 text-base",
      },
    },
    defaultVariants: {
      variant: "outline",
      inputSize: "md",
    },
  }
)
```

Error state: `border-error-strong` + error icon + message in `text-error`. Labels always visible above fields.

## 3.3 Card Variants

```tsx
const cardVariants = cva(
  "rounded-lg border bg-card text-card-foreground",
  {
    variants: {
      variant: {
        elevated: "shadow-sm",
        outlined: "border-border",
        filled: "bg-surface-warm border-transparent",
      },
      padding: {
        sm: "p-3",
        md: "p-4",
        lg: "p-6",
      },
    },
    defaultVariants: {
      variant: "elevated",
      padding: "md",
    },
  }
)
```

## 3.4 Badge Variants

```tsx
const badgeVariants = cva(
  "inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-semibold transition-colors",
  {
    variants: {
      variant: {
        solid: "border-transparent",
        subtle: "bg-muted border-transparent",
        outline: "bg-transparent",
      },
      color: {
        sage: "bg-sage-600 text-white border-transparent",
        ruby: "bg-brand text-white border-transparent",
        warning: "bg-warning text-white border-transparent",
        error: "bg-destructive text-white border-transparent",
        neutral: "bg-muted text-muted-foreground border-transparent",
        success: "bg-success text-white border-transparent",
      },
    },
    defaultVariants: {
      variant: "solid",
      color: "neutral",
    },
  }
)
```

`ruby` badge = emergency flag only. Always pair colour with text/icon.

## 3.5 Alert Variants

```tsx
const alertVariants = cva(
  "relative w-full rounded-lg border p-4",
  {
    variants: {
      variant: {
        info: "bg-info-bg text-info border-info-border",
        success: "bg-success-bg text-success border-success-border",
        warning: "bg-warning-bg text-warning border-warning-border",
        error: "bg-error-bg text-error border-error-border",
      },
    },
    defaultVariants: {
      variant: "info",
    },
  }
)
```

---

# 4. Utility Functions

```tsx
// lib/utils.ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

---

# 5. Theme Switching Implementation

```tsx
// components/theme-switcher.tsx
"use client"

import { useEffect, useState } from "react"
import { Sun, Moon, Monitor } from "lucide-react"
import { cn } from "@/lib/utils"

type Theme = "light" | "dark" | "system"

export function ThemeSwitcher() {
  const [theme, setTheme] = useState<Theme>("system")

  useEffect(() => {
    const root = document.documentElement

    if (theme === "system") {
      root.classList.remove("dark")
      root.removeAttribute("data-theme")
    } else {
      root.classList.toggle("dark", theme === "dark")
      root.setAttribute("data-theme", theme)
    }
  }, [theme])

  return (
    <div className="flex gap-2" role="group" aria-label="Theme">
      {([
        ["light", Sun, "Light mode"],
        ["dark", Moon, "Dark mode"],
        ["system", Monitor, "System default"],
      ] as const).map(([value, Icon, label]) => (
        <button
          key={value}
          onClick={() => setTheme(value)}
          className={cn(
            "rounded-md p-2 transition-colors touch-target-md",
            theme === value
              ? "bg-primary text-primary-foreground"
              : "hover:bg-muted"
          )}
          aria-label={label}
          aria-pressed={theme === value}
        >
          <Icon className="h-4 w-4" aria-hidden="true" />
        </button>
      ))}
    </div>
  )
}
```

Theme transition: 300ms colour crossfade (CSS `transition-colors` on body); no harsh contrast shifts. OS preference is the default; staff back office may offer this toggle (stored in localStorage).

---

# 6. Design Token → Tailwind Class Mapping

| Design Token | Tailwind Class | Notes |
|---|---|---|
| `--color-primary` (sage-600) | `bg-primary text-primary` | Everyday CTAs |
| `--color-brand` (ruby-700) | `bg-brand text-brand` | Emergency + links only |
| `--color-destructive` (error-strong) | `bg-destructive text-destructive` | Reject/cancel — never ruby |
| `--color-error` | `text-error` | Error text + icon |
| `--color-success` | `text-success` | |
| `--color-warning` | `text-warning` | |
| `--color-info` | `text-info` | |
| `--color-bg-primary` | `bg-background` | |
| `--color-bg-warm` | `bg-surface-warm` | Alternate sections |
| `--color-bg-secondary` | `bg-card` | |
| `--color-bg-tertiary` | `bg-muted` | |
| `--color-text-primary` | `text-foreground` | |
| `--color-text-secondary` | `text-foreground` / muted pairing | |
| `--color-text-tertiary` | `text-muted-foreground` | |
| `--color-text-link` | `text-brand` | Links |
| `--color-border-primary` | `border-border` | |
| `--color-border-secondary` | `border-input` | |
| `--radius-sm` | `rounded-sm` | Badges |
| `--radius-md` | `rounded-md` | Buttons/inputs |
| `--radius-lg` | `rounded-lg` | Cards |
| `--radius-xl` | `rounded-xl` | Modals |
| `--shadow-sm` | `shadow-sm` | |
| `--shadow-md` | `shadow-md` | |
| `--shadow-lg` | `shadow-lg` | |
| `--font-sans` | `font-sans` | Inter |
| `--font-serif` | `font-serif` | Blog bodies |
| `--font-mono` | `font-mono` | SKUs, order IDs |
| `--duration-fast` | `duration-150` | |
| `--duration-normal` | `duration-200` | |
| `--duration-slow` | `duration-300` | |
| `--touch-target-md` | `touch-target-md` | 44px min |

---

# 7. Component Usage Examples

```tsx
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert"

// Everyday primary — sage
<Button variant="primary">Book appointment</Button>
<Button variant="primary" size="lg">Add to cart</Button>

// Emergency — ruby brand only
<Button variant="brand-emergency" size="emergency">
  Call now — 01632 960245
</Button>

// Destructive — error tokens, never ruby
<Button variant="danger">Reject prescription</Button>

// Card
<Card variant="elevated" padding="lg">
  <CardHeader>
    <CardTitle>Heartgard Plus</CardTitle>
  </CardHeader>
  <CardContent>
    <p className="text-muted-foreground">From $28.99 · 6-month supply</p>
    <Badge color="ruby">Prescription required</Badge>
  </CardContent>
</Card>

// Alert
<Alert variant="warning">
  <AlertTitle>Prescription on hold</AlertTitle>
  <AlertDescription>
    This item needs vet authorisation. Call 01632 960245 if this is urgent.
  </AlertDescription>
</Alert>
```

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `design-system.md` | Authoritative token tables (hex source of truth) |
| `shadcn-tailwindcss-custom-sizing.md` | Fluid sizing variant of this config |
| `ui-components.md` | Component contract built on these tokens |
| `design.md` | Screen layouts using these classes |
| `accessibility.md` | Contrast/focus requirements |
| `../non-functional-requirements.md` | WCAG 2.1 AA, mobile-first |

---

# Acceptance Criteria

- Every token from `design-system.md` exists in `globals.css` `@theme` before themed components merge
- `--color-primary` maps to sage-600; `--color-brand` maps to ruby-700; destructive maps to error-strong
- Ruby appears only on emergency CTAs, links, and documented brand accents
- Light and dark both pass WCAG AA contrast checks
- Staff `.dark` toggle and OS `prefers-color-scheme` produce identical dark tokens

---

# Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Initial shadcn + Tailwind v4 configuration (design-system.md tokens) |

---

# Guiding Principle

> **Sage books the visit. Ruby answers the emergency. Errors never borrow the brand colour.**
