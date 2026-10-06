# ruby-veterinary Component Library

> **ruby-veterinary Documentation**
>
> **Document:** UI Component Library
>
> **Version:** 1.1.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** UI Standard
>
> **References:** `design-system.md`, `shadcn-tailwind.md`, `accessibility.md`, `patterns-emergency-first.md`

---

# Purpose

Reusable components built from `design-system.md`: variants, sizes, states, props, and accessibility behaviour. Implementation lives in `ruby-veterinary-web-frontend` (shadcn/ui + Tailwind v4). This document is the contract reviewers check against.

---

# Global Rules

- Touch targets ≥ 44×44px on mobile for all interactive elements
- Focus ring always visible (`ruby-500` light / `ruby-400` dark @ 40%), never `outline: none` without replacement
- Every interactive component works keyboard-only
- Colour is never the only signal for state (icon + text accompany colour)
- Disabled controls remain readable; they are not removed from layout
- **Ruby discipline:** `brand` (ruby-700 light / ruby-400 dark) reserved for emergency CTAs, links, and brand accents; everyday primary actions use sage
- **Destructive** uses error tokens (`error-strong` #D65328 / #E85C30), never brand ruby
- Minimum interactive text 16px on mobile forms
- Emergency phone icon always paired with visible number text — never icon-only for the main clinic line
- Error icons mandatory beside error text

---

# 1. Primitives

## 1.1 Button

**Purpose:** Trigger actions or events

**Variants:**

| Variant | Fill | Text | Usage |
|---------|------|------|-------|
| `primary` | sage-600 `#4A6659` | white | Book, Submit, Continue, Add to cart |
| `brand-emergency` | ruby-700 `#9B111E` | white | Emergency / Call now only |
| `secondary` | transparent | sage-700 + sage-200 border | Supporting actions |
| `ghost` | transparent | sage-700 | Low-emphasis |
| `danger` | error-strong `#D65328` | white | Reject Rx, cancel order — not ruby |
| `link` | none | brand (ruby-700) | Inline navigation |

Dark: primary sage-300 `#A3C4B0` (text `#171310`); brand ruby-400 `#F53D6D`; danger error-strong `#E85C30`.

**Sizes:** sm 36px · md 44px (default) · lg 52px · emergency lg+ with emphasised padding

**States:** default · hover (darken fill) · active (scale 0.98) · focus (ring) · disabled (opacity 0.5) · loading (spinner replaces label, `aria-busy`)

**Props:** `variant` · `size` · `disabled` · `loading` · `leftIcon` · `rightIcon` · `fullWidth` · `onClick`

**Accessibility:** Enter/Space activates; loading announced; emergency buttons include visible phone number text, not icon-only.

## 1.2 IconButton

**Purpose:** Icon-only actions (close, chevron, menu)

**Variants:** Same colour set as Button

**Sizes:** sm · md · lg

**Props:** `icon` (required) · `variant` · `size` · `ariaLabel` (required) · `disabled` · `onClick`

**Accessibility:** Must have `aria-label`; keyboard accessible; focus visible.

## 1.3 ButtonGroup

**Purpose:** Group related buttons

**Props:** `children` · `direction` horizontal|vertical · `attached` · `size`

## 1.4 Typography

**Purpose:** Render text with consistent styling

**Variants:** display · h1 · h2 · h3 · h4 · bodyLg · body · bodySm · caption · overline

**Props:** `variant` · `color` · `align` · `weight` · `truncate` · `maxLines` · `as`

**Accessibility:** Semantic HTML (h1–h6, p, span); proper heading hierarchy.

## 1.5 Icon

**Purpose:** Lucide stroke icons, 24px grid, 1.75px stroke

**Props:** `name` · `size` xs|sm|md|lg|xl · `color` · `ariaLabel`

**Accessibility:** Decorative icons `aria-hidden="true"`; informative icons need `aria-label`.

## 1.6 Avatar / AvatarGroup

**Purpose:** User/staff/pet-owner profile image or initials fallback

**Variants:** image · initials · icon

**Sizes:** xs 24 · sm 32 · md 40 · lg 48 · xl 64 · 2xl 96

**Props:** `src` · `name` · `size` · `shape` circle|square · `status` (optional)

**Accessibility:** Alt text from name or provided alt.

## 1.7 Badge / Chip

**Purpose:** Label, status, or count indicator

**Variants:** solid · subtle · outline

**Colours:**

| Colour | Usage |
|--------|-------|
| sage | Positive operational state (fulfilled, active, approved) |
| ruby | **Emergency flag only** |
| warning | Awaiting prescription, paused |
| error | Rejected, failed payment |
| neutral | Default metadata |
| success | Completed states |

**Sizes:** sm 16 · md 20 · lg 24

**Props:** `variant` · `color` · `size` · `dot` · `count` (99+ for >99) · `children`

**Accessibility:** Always text + optional icon; colour alone insufficient.

## 1.8 Tag

**Purpose:** Categorisation or filter label (blog categories, product types)

**Props:** `variant` · `color` · `size` · `closable` · `onClose` · `children`

## 1.9 Divider

**Purpose:** Visual separator

**Props:** `orientation` horizontal|vertical · `variant` solid|dashed · `color` · `spacing`

## 1.10 Skeleton

**Purpose:** Loading placeholder

**Variants:** text · circle · rectangle · custom

**Props:** `variant` · `width` · `height` · `count` · `animated` (shimmer; static under reduced-motion)

## 1.11 Spinner

**Purpose:** Loading indicator

**Sizes:** sm 16 · md 24 · lg 32

**Props:** `size` · `color` · `label`

**Accessibility:** `role="status"`; `aria-label`.

## 1.12 VisuallyHidden

**Purpose:** Content visible only to screen readers

**Props:** `children` · `as`

---

# 2. Forms

## 2.1 Input

**Purpose:** Text input field

**Variants:** outline · filled · underline

**Sizes:** sm 32 · md 44 (default) · lg 48

**States:** default · hover · focused (ring) · error (error-strong border + icon + message in `text-error`) · disabled · read-only

**Props:** `type` · `placeholder` · `value` · `name` · `id` · `size` · `variant` · `disabled` · `readOnly` · `required` · `error` · `helperText` · `leftAddon` · `rightAddon` · `onChange` · `onFocus` · `onBlur`

**Accessibility:** Label associated; error via `aria-describedby`; required indicated; autocomplete attributes (`email`, `tel`, `name`, `street-address`); min 16px text on mobile forms.

## 2.2 Textarea

**Props:** Same as Input plus `rows` · `resize` · `autoResize`

**Usage:** Intake reason, rejection reasons, blog body (staff).

## 2.3 Select / MultiSelect

**Purpose:** Dropdown selection

**Props:** `options` · `value` · `placeholder` · `size` · `variant` · `disabled` · `required` · `error` · `helperText` · `multiple` · `onChange` · `maxSelected` · `onRemove`

**Accessibility:** Native select or ARIA combobox; keyboard (arrows, Enter, Escape); type-ahead.

## 2.4 Checkbox / Radio / RadioGroup

**States:** unchecked/checked · indeterminate (checkbox) · disabled · error

**Props:** `checked` · `defaultChecked` · `indeterminate` · `disabled` · `error` · `label` · `description` · `onChange` · `name` (group) · `orientation`

**Notes:** 20px control, 44px hit area. Prescription-related toggles include explanatory microcopy.

**Accessibility:** Native inputs; label associated; indeterminate announced.

## 2.5 Switch

**States:** off · on · disabled · loading

**Props:** `checked` · `size` · `disabled` · `loading` · `label` · `description` · `onChange`

**Accessibility:** Native checkbox or ARIA switch; state announced.

## 2.6 Slider / NumberInput

**Slider:** single/range · `min` · `max` · `step` · `showValue` · `showMarks` · ARIA slider

**NumberInput:** qty steppers in cart · `min` · `max` · `step` · `precision`

## 2.7 PinInput

**Purpose:** OTP verification (account email confirm)

**Props:** `length` · `type` · `mask` · `value` · `onChange` · `onComplete`

## 2.8 DatePicker / TimePicker / DateTimePicker

**Purpose:** Appointment preferred windows, staff scheduling

**Props:** `value` · `min` · `max` · `format` · `placeholder` · `size` · `disabled` · `error` · `minuteStep` · `onChange`

**Accessibility:** ARIA calendar grid; keyboard navigation; screen reader announcements.

## 2.9 FileUpload / FileDropzone

**Purpose:** Intake medical history upload (PDF/JPEG)

**Variants:** dropzone · button · inline attachment

**States:** default · dragging · invalid · uploading · complete · error

**Props:** `accept` (PDF, JPEG) · `multiple` · `maxFiles` · `maxSize` (10 MB) · `disabled` · `value` · `onChange` · `onDrop` · `onRemove` · `onRetry`

## 2.10 Form / FormGroup / FormControl

**Props:** `onSubmit` · `initialValues` · `validationSchema` · `label` · `htmlFor` · `required` · `error` · `helperText` · `children`

**Behaviour:** Error summary at top on submit failure; focus first error field; real-time validation with debounce.

---

# 3. Data Display

## 3.1 Table

**Purpose:** Display tabular data

**Features:** sorting · pagination (cursor-based for staff) · row selection · sticky header · mobile → card-list fallback (never horizontal scroll without affordance)

**Props:** `data` · `columns` · `sortable` · `selectable` · `pagination` · `pageSize` · `stickyHeader` · `loading` · `emptyState` · `onSort` · `onSelect` · `onRowClick`

**Accessibility:** Semantic table; sort buttons with `aria-sort`; row selection `aria-selected`; keyboard navigation.

## 3.2 DataTable

**Purpose:** Advanced staff table with server-side sort/filter/pagination (Rx queue, catalogue, clients)

**Features:** column filters · bulk actions · export (CSV)

## 3.3 List / DescriptionList

**Props:** `items` · `variant` · `size` · `spacing` · `orientation`

## 3.4 Card

**Purpose:** Container for related content (services, products, blog, staff)

**Variants:** elevated (shadow-sm) · outlined · filled (surface-warm)

**Props:** `variant` · `padding` sm|md|lg · `hover` (shadow md only when clickable) · `onClick` · `children`

## 3.5 Stat

**Purpose:** Single dashboard metric (staff)

**Props:** `label` · `value` · `change` · `changeType` · `icon` · `prefix` · `suffix`

## 3.6 Timeline

**Purpose:** Appointment, Rx, or order history

**Props:** `items` · `orientation` · `size`

## 3.7 Accordion

**Purpose:** FAQ, care information, collapsible sections

**Props:** `items` · `multiple` · `defaultIndex` · `allowToggle` · `size`

**Accessibility:** ARIA accordion; Enter/Space/arrows; focus management.

## 3.8 Tabs

**Variants:** enclosed · underline · pills

**Props:** `tabs` · `defaultIndex` · `variant` · `size` · `onChange`

**Accessibility:** ARIA tablist/tab/tabpanel; arrow keys; focus management.

## 3.9 Breadcrumb

**Props:** `items` · `separator` · `maxItems` · `size`

**Accessibility:** `aria-label="Breadcrumb"`; nav element; `aria-current="page"` on last item.

## 3.10 Pagination

**Variants:** numbered · simple (prev/next) · mini

**Props:** `current` · `total` · `variant` · `size` · `showFirstLast` · `onChange`

## 3.11 Calendar / Code / CodeBlock

**Calendar:** staff scheduling grid

**Code/CodeBlock:** SKUs, order IDs in mono font; optional syntax highlight for admin logs

---

# 4. Navigation

## 4.1 Public StickyContactHeader / Navbar

**Components:** Logo · nav links (Services, Store, Blog, Staff) · Hours snippet · **Call (ruby, tel link)** · Menu

**Behaviour:** Emergency number remains visible on scroll.

**Accessibility:** `role="banner"`; search landmark if present; skip link.

## 4.2 Owner Mobile BottomNav

**Items:** **Call** (ruby) · **Book** (sage) · **Store** · **Account**

**Props:** `items` · `active` · `onChange`

**Accessibility:** Navigation landmark; keyboard accessible; active state indicated.

## 4.3 Staff Sidebar

**Variants:** fixed · collapsible · overlay (mobile)

**States:** expanded 256px · collapsed 72px · hidden (mobile)

**Items (role-scoped):** Dashboard · Rx Review · Catalogue · Content · Inbox · Clients · Reports · Settings

**Props:** `items` · `expanded` · `onToggle` · `width` · `collapsedWidth` · `position`

**Accessibility:** Navigation landmark; keyboard; focus management.

## 4.4 Menu / Dropdown

**Variants:** dropdown · context menu · nested

**Props:** `items` · `placement` · `trigger` · `closeOnSelect`

**Accessibility:** ARIA menu pattern; arrows/Enter/Escape; focus management.

## 4.5 CommandPalette (staff, optional)

**Props:** `isOpen` · `onClose` · `onSearch` · `commands`

**Accessibility:** ARIA dialog; focus trap; keyboard navigation.

## 4.6 Link

**Variants:** inline · standalone · navigation

**Props:** `href` · `target` · `variant` · `color` · `disabled` · `onClick`

**Accessibility:** Native anchor; external links indicated; brand colour for links.

---

# 5. Feedback

## 5.1 Alert

**Variants:** info · success · warning · error

**Props:** `variant` · `title` · `description` · `icon` · `closable` · `action` · `onClose`

**Accessibility:** `role="alert"` important / `role="status"` less important; closable alerts announce state.

## 5.2 Toast

**Positions:** top-right (default) · top-left · bottom-right · bottom-left

**Props:** `variant` · `title` · `description` · `duration` · `position` · `closable` · `action` · `onClose`

**Accessibility:** `role="status"` or `role="alert"`; auto-dismiss announced.

## 5.3 Modal / Dialog

**Variants:** default · confirmation · fullscreen · drawer

**Sizes:** sm 400 · md 600 · lg 800 · xl 1000 · full

**Styling:** radius xl; shadow lg; backdrop `rgba(0,0,0,0.45)`

**Props:** `isOpen` · `onClose` · `title` · `size` · `variant` · `closable` · `closeOnOverlayClick` · `closeOnEsc` · `initialFocus` · `finalFocus` · `children` · `footer`

**Accessibility:** `role="dialog"`; `aria-modal="true"`; focus trap; return focus on close; ESC to close. Prescription decision modals show pet + order context before destructive actions.

## 5.4 Popover / Tooltip

**Popover:** click/hover/focus trigger; placement; offset; arrow

**Tooltip:** hint on hover; `role="tooltip"`; `aria-describedby`; delay

## 5.5 Progress

**Variants:** linear · circular

**Props:** `value` · `variant` · `size` · `color` · `indeterminate` · `label` · `showValue`

**Accessibility:** `role="progressbar"`; `aria-valuenow/min/max`; `aria-label`.

## 5.6 SkeletonLoader / EmptyState / ErrorState / ConfirmationDialog

**SkeletonLoader:** text · avatar · card · listItem · tableRow · `animated`

**EmptyState:** illustration optional · title · description · action (sage primary) · phone fallback line · `image`

**ErrorState:** title · description · `error` · `onRetry` · `supportLink` (clinic phone)

**ConfirmationDialog:** `isOpen` · `onConfirm` · `onCancel` · `title` · `message` · `confirmLabel` · `cancelLabel` · `variant` danger|warning|info

## 5.7 EmergencyCallBar

**Purpose:** Sticky emergency contact affordance on all owner-facing routes

**Placement:** Fixed bottom mobile · persistent header strip desktop

**Content:** Phone icon + clinic emergency number (tel: link) + optional "Call now" `brand-emergency` button

**Behaviour:** Never dismissible on mobile; works without session; z-index sticky 100; main content reserves padding-bottom so forms' submit buttons are never covered

**Accessibility:** First tab stop after skip-link; announced; icon + visible number text always

---

# 6. Layout

## 6.1 Container

**Variants:** fluid · fixed max-width 1280px

**Props:** `size` · `centered` · `padding` · `children`

Gutters: 16px mobile · 24px tablet · 32px desktop (`.container-clinic`).

## 6.2 Stack / Grid / Flex / Box / AspectRatio / Center / Wrap / SimpleGrid / Responsive

Standard layout primitives. Responsive hide/show by breakpoint (sm 640 · md 768 · lg 1024 · xl 1280).

---

# 7. Media

## 7.1 Image

**Variants:** default · rounded · circle · thumbnail

**Props:** `src` · `alt` (required) · `size` · `shape` · `objectFit` · `fallback` · `loading` lazy|eager · `onError`

**Accessibility:** Alt text required; decorative images empty alt; lazy loading for performance.

## 7.2 MediaWithOverlay

**Purpose:** Image with text overlay (staff heroes, blog banners)

**Props:** `src` · `alt` · `overlay` · `position` · `gradient` · `children`

## 7.3 Lightbox (optional)

**Purpose:** Full-screen media viewer (gallery)

**Props:** `isOpen` · `items` · `activeIndex` · `onClose` · `onNavigate` · `zoom`

**Accessibility:** ARIA dialog; arrows/ESC; focus trap.

---

# 8. Domain-Specific

## 8.1 ClinicHoursChip

**Purpose:** Open/closed state near contact CTAs

**Variants:** open · closed

**Content:** "Open now · closes 18:00" / "Closed · opens 09:00 tomorrow"

**Accessibility:** Text label mandatory — colour alone insufficient.

## 8.2 AppointmentForm

**Purpose:** Owner appointment request

**Fields:** contact details · pet details · reason (required textarea) · preferred window (date + time slots) · urgency select (routine/soon/urgent — owner hint only, not clinical triage)

**Props:** `onSubmit` · `initialValues` · `pets` · `loading` · `error`

**Behaviour:** Submit = `primary` sage; inline + error summary on failure; phone fallback line beneath form.

## 8.3 IntakeStepper

**Purpose:** New-client intake progress (3–4 steps)

**Props:** `steps` · `current` · `onStepChange` · `onComplete`

**Behaviour:** Current step sage; completed steps check icon; upload step shows PDF/JPEG, 10 MB cap, per-file progress.

## 8.4 HistoryUpload

**Purpose:** Medical history file upload during intake

**Props:** `files` · `onUpload` · `onRemove` · `onRetry` · `maxSize` · `accept`

## 8.5 ProductCard

**Purpose:** Store catalogue item

**Props:** `id` · `name` · `category` · `image` · `priceFrom` · `requiresPrescription` · `onAddToCart` · `onView`

**Behaviour:** "From $X" pricing; Rx badge when `requires_prescription`; Add to cart = `primary` sage.

## 8.6 VariantPicker

**Purpose:** Size/pack selection for products

**Props:** `variants` · `selected` · `onChange` · `disabledVariants` · `petClaimRequired`

**Behaviour:** Pill buttons; disabled variants visibly marked; pet-claim selector appears for Rx items.

## 8.7 PetClaimSelector

**Purpose:** Choose which pet an Rx item is for

**Props:** `pets` · `selected` · `onChange` · `onAddPet`

## 8.8 CartLine / CheckoutSummary

**CartLine:** variant · qty stepper (44px controls) · line total · pet association chip for Rx lines · remove (error tokens + icon, never ruby)

**CheckoutSummary:** subtotal · tax · shipping or pickup (0) · grand total · **prescription-hold notice** (warning tokens + icon + explanation when Rx items present) · payment gateway down → error banner + phone fallback

## 8.9 RxStatusBadge

**Purpose:** Prescription status for owners and staff

| Status | Colour | Copy |
|--------|--------|------|
| pending | warning | "Awaiting vet approval" |
| approved | sage | "Approved" |
| rejected | error | "Not approved" + reason |
| queried | info | "Vet has a question" |

**Props:** `status` · `reason` · `size`

**Accessibility:** Plain language; colour + text always.

## 8.10 ReviewQueueRow / DecisionPanel

**ReviewQueueRow (staff):** pet name · owner · requested time · emergency pin (ruby badge) · status · claim/decision actions · claimed-by indicator for concurrent agents

**DecisionPanel (vet):** pet + prescriber context · order lines · history upload links · **Approve** (primary sage) · **Query** (info) · **Reject** (danger + required reason fields) · every action writes audit entry (silent to user)

## 8.11 InboxConversation

**Purpose:** WhatsApp + message inbox with claim/handover

**Props:** `messages` · `claimedBy` · `awaitingHandover` · `botPaused` · `onClaim` · `onReply` · `onHandover`

**Behaviour:** Composer enabled only when claimed by current user or supervisor; bot-paused badge during handover.

## 8.12 NewsletterBlock

**Props:** `onSubmit` · `consentText` · `successMessage`

**Behaviour:** Email input + sage submit; success copy offers phone fallback if email confirmation does not arrive.

## 8.13 SearchResults / FilterPanel

**SearchResults:** `results` · `total` · `query` · `filters` · `onFilter` · `onResultClick`

**FilterPanel:** slide-out/bottom-sheet filters for store, blog, staff catalogue

---

# 9. Composite Components

## 9.1 SearchBar

**Props:** `placeholder` · `value` · `suggestions` · `recentSearches` · `categories` · `loading` · `onChange` · `onSearch` · `onSelect`

## 9.2 NotificationBell / UserMenu / ThemeSwitcher

**NotificationBell:** `notifications` · `unreadCount` · `onNotificationClick` · `onMarkAllRead` · `onViewAll`

**UserMenu:** `user` · `items` · `onLogout` · `onSettings`

**ThemeSwitcher:** `theme` light|dark|system · `onChange` — 300ms crossfade; OS preference default; staff may toggle explicit dark (localStorage)

## 9.3 EmptyStateAction

**Props:** `icon` · `title` · `description` · `actionLabel` · `onAction` · `secondaryAction`

---

# 10. Responsive Behavior

## 10.1 Mobile (< 640px)

| Component | Behavior |
|-----------|----------|
| Staff Sidebar | Hidden, replaced by menu |
| Public Navbar | Compact + Call always visible |
| BottomNav | Owner app shell (Call/Book/Store/Account) |
| Cards | Full-width, stacked |
| Tables | Card-list fallback |
| Modals | Full-screen or bottom sheet |
| Dropdowns | Bottom sheet |
| Filters | Bottom sheet with apply |
| Search | Full-screen overlay |
| Pagination | Simple (prev/next) |
| Tabs | Scrollable horizontal |
| EmergencyCallBar | Sticky bottom, never dismissible |

## 10.2 Tablet (640–1024px)

| Component | Behavior |
|-----------|----------|
| Sidebar | Collapsible |
| Navbar | Full with Call |
| Cards | 2-column grid |
| Tables | Responsive (some columns hide) |
| Modals | Centered, medium |

## 10.3 Desktop (> 1024px)

| Component | Behavior |
|-----------|----------|
| Sidebar | Fixed, expandable |
| Navbar | Full; emergency strip in header |
| Cards | 3–4 column grid |
| Tables | Full-featured |
| Modals | Centered, configurable size |

---

# 11. Animation Specifications

## 11.1 Page Transitions

| Transition | Duration | Easing | Usage |
|------------|----------|--------|-------|
| Fade In | 200ms | ease-out | Page content appears |
| Slide Left | 250ms | ease-in-out | Navigate deeper |
| Slide Right | 250ms | ease-in-out | Navigate back |
| Crossfade | 300ms | ease-in-out | Theme switch |

## 11.2 Component Animations

| Animation | Duration | Easing | Usage |
|-----------|----------|--------|-------|
| Scale In | 150ms | ease-out | Modal/dialog appears |
| Fade In | 200ms | ease-out | Content loads |
| Slide Up | 200ms | ease-out | Dropdown/popover opens |
| Pulse | 1.5s | ease-in-out | Skeleton shimmer |
| Spin | 800ms | linear | Spinner |
| Press | 100ms | ease-out | Button scale 0.98 |

## 11.3 Feedback Animations

| Animation | Duration | Easing | Usage |
|-----------|----------|--------|-------|
| Toast Slide In | 300ms | ease-out | Notification appears |
| Toast Slide Out | 250ms | ease-in | Notification dismisses |
| Backdrop Fade | 200ms | ease-out | Modal backdrop |
| Success Checkmark | 400ms | spring | Action completed |
| Error Shake | 400ms | ease-in-out | Invalid input |

**Rules:** All honour `prefers-reduced-motion: reduce`. No parallax. No attention-grabbing pulse on emergency UI.

---

# 12. Accessibility Patterns

## 12.1 Keyboard Navigation

| Component | Keys | Action |
|-----------|------|--------|
| Button | Enter/Space | Activate |
| Link | Enter | Navigate |
| Input | Typing | Enter text |
| Select | Arrows | Navigate options |
| Tabs | Arrows | Switch tabs |
| Accordion | Enter/Space | Toggle |
| Menu | Arrows | Navigate items |
| Modal | Tab | Trap focus |
| Table | Arrows | Navigate cells |
| EmergencyCallBar | Tab | Reach call control |

## 12.2 ARIA Patterns

| Pattern | Components |
|---------|------------|
| alert | Toast, Alert |
| dialog | Modal, Popover, Lightbox |
| menu | Dropdown |
| tablist | Tabs |
| combobox | Select, SearchBar |
| progressbar | Progress |
| tooltip | Tooltip |

## 12.3 Focus Management

- Focus trap in modals; focus return on close
- Skip links for keyboard users
- Visible focus indicators on all themes
- Logical tab order
- Focus restoration after notifications

See `accessibility.md` for the full WCAG 2.1 AA plan.

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `design-system.md` | Tokens behind every variant |
| `shadcn-tailwind.md` | Implementation mapping (cva + `@theme`) |
| `accessibility.md` | Interaction requirements |
| `patterns-emergency-first.md` | Layout patterns using these components |
| `design.md` | Screen layouts assembling these components |
| `ui-generation-prompts/` | Visual generation of these components |
| `../api-specification.md` | Data contracts feeding forms and tables |
| `../prd.md` | Product modules |

---

# Acceptance Criteria

- Every variant above is implemented or explicitly deferred with a ticket
- Emergency CTAs use `brand-emergency`; no routine button uses ruby
- Destructive actions use error tokens + icon + text
- Form errors always show icon + text + field association
- Staff tables degrade to card lists on mobile
- EmergencyCallBar present on all owner-facing routes

---

# Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Initial component contract |
| 1.1.0 | 2026-10-06 | Expanded to full catalog (reniverse structure, veterinary domain) |

---

# Guiding Principle

> **Components exist so staff stop inventing buttons. If a flow needs a new variant, the design system needs a rule, not a one-off.**
