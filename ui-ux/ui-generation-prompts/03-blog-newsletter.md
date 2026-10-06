# Google Stitch Prompt — ruby-veterinary Blog & Newsletter

> **Purpose:** Paste this prompt into Google Stitch to generate the clinic blog (CMS) reading surfaces and the newsletter sign-up for ruby-veterinary.
>
> **Coverage:** 5 screens — blog index, article page, category view, tag view, newsletter subscribe section.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. Article bodies and article card titles use Merriweather serif — everything else stays Inter.

---

## Design System Context (Paste First)

```
I'm designing ruby-veterinary — a single veterinary clinic's website: emergency-first public site (phone, hours, address always visible), services and staff pages, clinic blog, WhatsApp triage bot with human handover, online store (general supplies, therapeutic diets, prescription items behind vet authorisation), client intake + appointment requests, and a role-scoped staff back office.

TARGET PLATFORM: Responsive web — mobile-first for pet owners, desktop-first for staff tools. Next.js 16 + React 19 + Tailwind CSS 4. Light mode AND dark mode are both first-class.

BRAND PERSONALITY: Calm, caring, trustworthy, urgent only when needed. Plain-spoken, never salesy in an emergency context.

COLOR SYSTEM (light / dark):
- Page background: #FFFFFF / #171310
- Warm alternate sections: #FAF7F5 / #1F1A16
- Cards and panels: #F6F2EF / #1F1A16
- Subtle fills: #EFE9E4 / #2A2420
- Elevated (modals, popovers): #FFFFFF / #241E1A
- Borders: #E7DFD8 / #3A322C · stronger: #D4C8BE / #4C423A
- Text primary: #1F1A17 / #F7F3F0 · secondary: #5C534C / #C4B8AE · tertiary: #8A7F77 / #94887D · inverse: #FFFFFF / #171310
- Links: #9B111E / #F53D6D
- Emergency ruby fill (Call now): #9B111E / #F53D6D with white text; hover #C00E52 / #E0115F
- Ruby bright accent and focus ring: #E0115F / #F53D6D
- Ruby soft surfaces: #FFF5F7 / #FFE4EC (light) · #1F1A16 with #F53D6D accents (dark)
- Everyday primary buttons (Book appointment, Add to cart, Submit, Continue, Save, Publish): #4A6659 with white text / #A3C4B0 with #171310 text; hover #3B5349 / #7BA88C
- Sage soft panels: #F4F8F5 and #E3EDE6 / #241E1A and #2A2420 · sage strong text and secondary links: #3B5349 / #A3C4B0
- Success: #2F7D5A on #EAF4EF / #4ADE9B on #123528
- Warning: #B7791F on #FBF3E4 / #E3B341 on #3B2E10
- Error text: #B8431F / #F0754A · destructive fill: #D65328 / #E85C30 with white text · error surfaces: #FDF0E8 / #3B1D10
- Info: #2B6CB0 on #EBF2FA / #6BA3D6 on #15273A
- Disabled: #EFE9E4 fill with #8A7F77 text / #2A2420 fill with #94887D text
- Focus ring: 3px #E0115F light / 3px rgba(245,61,109,0.4) dark

RUBY DISCIPLINE (hard rule): ruby #9B111E / #F53D6D only on emergency CTAs, "Call now" buttons, links, and documented brand accents. Everyday actions use sage #4A6659 / #A3C4B0. Errors and destructive confirms use #B8431F text / #D65328 fill plus an icon and text — never brand ruby. No full-width ruby hero backgrounds; never every button ruby.

TYPOGRAPHY:
- UI: Inter (system-ui, sans-serif) · Blog article bodies and article card titles: Merriweather (Georgia, serif) · SKUs/IDs: JetBrains Mono
- Display 48/56 · H1 36/44 · H2 28/36 · H3 22/30 · H4 18/26 · Body large 18/28 · Body 16/24 (default) · Body small 14/20 · Caption 12/16
- Headings 600 weight, -0.01em tracking · body 400 · min 16px interactive text in mobile forms

SPACING (4px base): 0, 2, 4, 6, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96

RADIUS: 4px badges · 8px buttons/inputs · 12px cards · 16px modals · 9999px avatars/pills

SHADOWS: sm 0 1px 3px rgba(31,26,23,.08) · md 0 4px 12px rgba(31,26,23,.10) · lg 0 12px 28px rgba(31,26,23,.14) · dark: raise opacity to 0.35+

Z-INDEX: base 0 · sticky 100 · header 110 · dropdown 200 · overlay 400 · modal 500 · toast 600

MOTION: 75/150/200/300ms, easing cubic-bezier(0.4, 0, 0.2, 1); respect prefers-reduced-motion

BREAKPOINTS: 640 / 768 / 1024 / 1280 · 12-col grid · max width 1280px · gutters 16/24/32

EMERGENCY PATTERN: sticky mobile tap-to-call bar with the clinic phone; phone, hours and address visible without opening a menu; phone fallback text on every degraded, empty, or error state; phone icon always paired with the visible number.

ACCESSIBILITY (WCAG 2.1 AA): keyboard operable, visible focus ring, alt text on meaningful images, ≥4.5:1 normal text contrast (both modes), tap targets ≥44px on owner-facing mobile, never colour alone — errors carry an icon and text.

Generate a DESIGN.md from this specification first, then proceed to the screen prompts below.
```

---

## Dark Mode Color Mapping (Apply to all screens)

Every screen below must be generated in BOTH light and dark mode. Same layout for both — only colours change.

- Page background: `#FFFFFF` → `#171310`
- Warm sections: `#FAF7F5` → `#1F1A16`
- Cards/panels: `#F6F2EF` → `#1F1A16` · subtle fills `#EFE9E4` → `#2A2420` · elevated `#FFFFFF` → `#241E1A`
- Borders: `#E7DFD8` → `#3A322C` · stronger `#D4C8BE` → `#4C423A`
- Text: primary `#1F1A17` → `#F7F3F0` · secondary `#5C534C` → `#C4B8AE` · tertiary `#8A7F77` → `#94887D` · inverse `#FFFFFF` → `#171310`
- Links: `#9B111E` → `#F53D6D`
- Emergency ruby CTAs ("Call now"): fill `#9B111E` → `#F53D6D`, white text in both modes · hover `#C00E52` → `#E0115F`
- Everyday primary buttons (Book appointment, Submit, Continue): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310` · hover `#3B5349` → `#7BA88C`
- Sage panels: `#F4F8F5` / `#E3EDE6` → `#241E1A` / `#2A2420` · sage text `#3B5349` → `#A3C4B0`
- Success: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Warning: `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces
- Serif note: article body text uses `#1F1A17` light / `#F7F3F0` dark — Merriweather keeps the same sizes in both modes

---

## Group 1: Blog Discovery

### Screen 1 — Blog Index

```
Generate the blog index page for ruby-veterinary — the clinic's educational content hub.

DESKTOP LAYOUT:
- Breadcrumb: Home / Blog
- Page header on warm #FAF7F5: H1 "Pet care advice from our clinic" (36px), subline 18px #5C534C "Written by our veterinarians and vet techs — practical, local, and plain-spoken."
- Category chip row: All · Dog Care · Cat Care · Puppy/Kitten Tips · Nutrition · Seasonal Safety — selected chip sage #4A6659 fill/white text; unselected white fill, 1px #E7DFD8, #5C534C text
- Featured article (large, full width): 2-column card — image placeholder left (16:9, 12px radius), right side: category label 12px uppercase #3B5349, title in Merriweather 30px #1F1A17, dek 16px #5C534C, byline row "Dr Amara Okoye · 6 min read · Mar 2026" 14px #8A7F77, ruby #9B111E link "Read article →"
- Article card grid below: 3 columns, white cards, 1px #E7DFD8, 12px radius, shadow sm
  - Image placeholder top (4:3), category pill over image (white 90% bg, #3B5349 text)
  - Card title Merriweather 20px weight 600 #1F1A17 (serif titles are the blog's signature)
  - Dek 14px #5C534C, byline 13px #8A7F77 with author name and read time
  - Card footer: tag chips 12px #5C534C on #F6F2EF
- Right sidebar (3 cols): newsletter subscribe card (Screen 5 content) + "Popular this month" list with ruby numbered links

MOBILE LAYOUT (primary):
- H1 + subline, chips horizontally scrollable
- Featured card stacks: image, then category, Merriweather title 24px, dek, byline, "Read article →" full-width row 48px
- Cards single column; sidebar newsletter card moves below the 3rd card; "Popular this month" last
- Serif titles remain serif on mobile — do not swap to Inter

ACCESSIBILITY:
- Article cards are single focus targets with accessible names including the title; focus ring 3px #E0115F (dark rgba(245,61,109,0.4))
- Image placeholders have alt text noted ("Golden retriever puppy at a wellness exam")
- Chip filters are buttons with aria-pressed; selected state also shown by weight/border, not colour alone
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · Merriweather #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (meta only, never for links); dark mode #F7F3F0 on #171310 ≈ 17:1, #C4B8AE ≈ 10:1
- Tap targets ≥44px; keyboard-scrollable chip row without scroll traps

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 2 — Article Page

```
Generate the article page for ruby-veterinary — a single long-form clinic blog post.

DESKTOP LAYOUT:
- Breadcrumb: Home / Blog / Nutrition / article title
- Reading column max-width 720px centred; page background white #FFFFFF
- Article header: category label 12px uppercase #3B5349, H1 title Merriweather 36px/44px #1F1A17 ("How to switch your dog to a therapeutic diet without an upset stomach")
- Byline row: author avatar (40px circle, alt noted) + "Dr Amara Okoye, DVM" 14px weight 600 #1F1A17 + credential chip "Veterinarian" sage-50 #F4F8F5/#3B5349 + "Published Mar 14, 2026 · 6 min read" 14px #8A7F77
- Hero image placeholder (16:9, 12px radius) with caption 13px #8A7F77
- Body in Merriweather 18px/30px #1F1A17: intro paragraph, H2 subheads (Merriweather 24px), ordered list of the 7-day transition schedule, a pull-quote on sage-50 #F4F8F5 with 4px left border #4A6659, an inline info callout on #EBF2FA with info icon #2B6CB0 ("Always transition gradually — sudden diet changes can cause vomiting or diarrhoea."), and an embedded video placeholder with play icon
- Inline links #9B111E with underline
- Social share row after body: "Share:" + outlined icon buttons for Facebook, WhatsApp, Pinterest (40px, 1px #E7DFD8 border, #5C534C icons)
- Author card: sage-50 #F4F8F5 panel — avatar, name, one-line bio, ruby text link "More articles by Dr Okoye →"
- Related articles: H2 "You might also like" + 3 compact cards (Merriweather 18px titles)
- Newsletter block (Screen 5) at the end
- Right rail (optional, sticky): "Emergency? Call 010 555 0199" small ruby link + table of contents

MOBILE LAYOUT (primary):
- Single column, 16px padding, reading column full width
- H1 Merriweather 28px/36px; byline stacks avatar+name, then date
- Body Merriweather 18px/30px, line length comfortable at 360px
- Share row full-width buttons (48px) wrapping to 3 across
- Related articles single column; newsletter block full width
- Sticky tap-to-call bar from the design system persists below content

ACCESSIBILITY:
- Semantic article markup with H1 → H2 → H3 hierarchy; "You might also like" is an H2 landmark section
- Share buttons have accessible names ("Share on WhatsApp"); icons aria-hidden
- Alt text on hero and inline images; video placeholder labelled "Video: switching diets — 3 min"
- Contrast: #1F1A17 Merriweather on #FFFFFF ≈ 16:1 · #9B111E links on #FFFFFF ≈ 8.4:1 · #2B6CB0 on #EBF2FA ≈ 5.6:1 · #3B5349 on #F4F8F5 ≈ 6.9:1; dark mode #F7F3F0 on #171310 ≈ 17:1, #F53D6D links ≈ 5.4:1, #6BA3D6 on #15273A ≈ 5.5:1
- Keyboard: skip-to-content link, focus ring 3px #E0115F on links/buttons, no keyboard traps in embeds
- Tap targets ≥44px for share, related, and CTA links

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Taxonomy & Subscribe

### Screen 3 — Category View

```
Generate the category view page for ruby-veterinary — the listing for one blog category (e.g. "Dog Care").

DESKTOP LAYOUT:
- Breadcrumb: Home / Blog / Dog Care
- Category header on warm #FAF7F5: category label 12px uppercase #3B5349, H1 "Dog Care" (36px), description 18px #5C534C "Vaccination schedules, dental health, behaviour, and nutrition advice for dogs of every age." + article count "18 articles" 14px #8A7F77
- Sort control right-aligned: outlined select "Sort: Newest first" (40px, 1px #D4C8BE, 8px radius)
- Article list as horizontal cards (8 cols): image left (160×120, 8px radius), content right — category+date 12px #8A7F77, title Merriweather 22px #1F1A17, dek 14px #5C534C, byline 13px #8A7F77, tag chips; rows separated by 1px #E7DFD8
- Sidebar (4 cols): other categories list with counts (ruby text links), newsletter card
- Pagination: numbered buttons (40px, 8px radius, current page sage #4A6659 fill white text, others white with 1px #E7DFD8)

MOBILE LAYOUT (primary):
- Category header stacked with description and count
- Cards stack: image full width 16:9, then title, dek, byline — no side-by-side
- Sort control full width (48px)
- Pagination: "Previous · 1 2 3 · Next" wrapped, ≥44px targets
- Other categories as a full-width list below the last card

ACCESSIBILITY:
- Category header uses H1; article titles H2/H3 — logical hierarchy
- Pagination uses nav landmark with aria-label "Blog pagination"; current page aria-current="page"
- Focus ring 3px #E0115F visible on cards, sort, pagination
- Contrast: #1F1A17 on #FAF7F5 ≈ 15:1 · #5C534C on #FFFFFF ≈ 7.4:1 · white on #4A6659 ≈ 5.9:1; dark equivalents ≥4.5:1
- Tap targets ≥44px; article cards single focus target each

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — Tag View

```
Generate the tag view page for ruby-veterinary — the listing for one tag (e.g. "nutrition").

DESKTOP LAYOUT:
- Breadcrumb: Home / Blog / Tags / nutrition
- Tag header: pill badge (radius 9999px, sage-50 #F4F8F5 background, #3B5349 text, 16px) reading "# nutrition", H1 "Articles tagged 'nutrition'" (36px, #1F1A17), count "9 articles · updated weekly" 14px #8A7F77
- Compact article grid: 3 columns of vertical cards (white, 1px #E7DFD8, 12px radius) — image 4:3, Merriweather 20px title, one-line dek 14px #5C534C, byline 13px #8A7F77, category link #9B111E 13px
- Related tags cloud row: small pill chips 13px, unselected #F6F2EF/#5C534C, "nutrition" chip selected sage #4A6659/white
- Empty-state variant (show as a secondary panel): sage-50 #F4F8F5 card with line-art icon, H2 "No articles with this tag yet", body "Try a category instead, or call us on 010 555 0199.", links to Blog index + ruby phone link

MOBILE LAYOUT (primary):
- Tag pill + H1 + count stacked
- Single-column cards, full width, 16px padding, title Merriweather 20px
- Related tags wrap into 2 rows of chips (44px tall each)
- Empty-state panel full width with full-width outlined "Browse all articles" button and full-width ruby "Call 010 555 0199" row

ACCESSIBILITY:
- H1 describes the tag; article titles are H2; tag chips are links with accessible names including the tag
- Selected tag chip distinguished by weight and text, not colour alone (aria-current="true" noted)
- Focus ring 3px #E0115F; contrast #3B5349 on #F4F8F5 ≈ 6.9:1, #5C534C on #F6F2EF ≈ 6.8:1, white on #4A6659 ≈ 5.9:1; dark ≥4.5:1 across the board
- Empty state pairs icon + heading + text (never colour alone); phone fallback line present
- Tap targets ≥44px on all chips and cards

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 5 — Newsletter Subscribe Section

```
Generate the newsletter subscribe section for ruby-veterinary — the reusable sign-up block that appears in the sidebar, at the end of articles, and on the blog index.

DESKTOP LAYOUT (sidebar variant, ~340px wide card):
- Card: sage-50 #F4F8F5 background (dark: #241E1A), 12px radius, padding 24px, no heavy shadow
- Small mail/envelope icon 24px in sage #4A6659 (dark #A3C4B0)
- Heading H3 22px #1F1A17: "Monthly clinic notes"
- Body 14px #5C534C: "One email a month: seasonal warnings, vaccination reminders, and new services. No sales spam."
- Email input: full width, 44px height, white #FFFFFF background, 1px #E7DFD8 border, 8px radius, placeholder #8A7F77 "you@example.com", label above in 13px weight 600 #1F1A17 ("Email address")
- Sage #4A6659 filled "Subscribe" button full width, 44px, white text, 8px radius (this is the everyday primary action — NOT ruby)
- Privacy note 12px #8A7F77 under the button: "We store your email only to send this newsletter. Unsubscribe any time. See our privacy policy." with an underlined link
- Success state variant: green check icon #2F7D5A + "You're subscribed. Check your inbox to confirm." 14px on #EAF4EF panel
- Error state variant: error icon + text #B8431F on #FDF0E8 "Enter a valid email address." below the field, field border #B8431F

MOBILE LAYOUT (primary, full-width block variant):
- Full-width panel on sage-50 #F4F8F5, 16px padding, 12px radius
- Icon + heading row, body text, then label, full-width email input (48px), full-width sage "Subscribe" button (52px), privacy note 12px
- Success and error states stack full width, keeping the icon + text pairing

ACCESSIBILITY:
- Visible label tied to the input (not placeholder-only); autocomplete="email"
- Submit button announces state ("Subscribe"); success message in role="status", error in role="alert"
- Focus ring 3px #E0115F on input and button; keyboard order label → input → button → privacy link
- Contrast: #1F1A17 on #F4F8F5 ≈ 14:1 · #5C534C on #F4F8F5 ≈ 6.6:1 · white on #4A6659 ≈ 5.9:1 · #8A7F77 privacy text on #F4F8F5 ≈ 4.6:1 · error #B8431F on #FDF0E8 ≈ 5.4:1; dark mode: #F7F3F0 on #241E1A ≈ 15:1 · #A3C4B0 button fill with #171310 text ≈ 9.4:1 · #F0754A on #3B1D10 ≈ 5.6:1
- Tap targets ≥44px (inputs 44–52px); error state never relies on red border alone — icon + message required

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Blog Discovery | 2 | ~10 |
| Taxonomy & Subscribe | 3 | ~15 |
| **Total** | **5** | **~25** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one at a time.
3. Verify serif usage: Merriweather on article bodies and article card titles only; if Stitch swaps it for Inter, follow up: "Article titles and body must use Merriweather (Georgia, serif)."
4. Useful follow-ups: "Make the newsletter submit button sage #4A6659" · "Add the error state to the email field" · "Apply the dark mode mapping."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens (serif typography rules)
- `09-staff-backoffice.md` — article editor where these posts are authored
- `01-emergency-homepage.md` — persistent emergency chrome reused on blog pages
