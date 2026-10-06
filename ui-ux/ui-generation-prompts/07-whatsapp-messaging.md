# Google Stitch Prompt — ruby-veterinary WhatsApp Triage & Messaging

> **Purpose:** Paste this prompt into Google Stitch to generate the WhatsApp triage bot surfaces and the staff shared inbox for ruby-veterinary.
>
> **Coverage:** 5 screens — bot conversation preview, panic escalation state, staff inbox list, conversation view, handover claimed state.
>
> **Tip:** Paste the Design System Context first, then generate one screen at a time. The phone fallback line is present in every chat state — degraded messaging never leaves an owner stranded.

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
- Chat canvas: `#FAF7F5` → `#1F1A16`
- Bubbles — bot messages: `#FFFFFF` → `#241E1A` (border `#E7DFD8` → `#3A322C`)
- Bubbles — owner messages: `#E3EDE6` → `#2A2420` (dark sage tint, text `#1F1A17` → `#F7F3F0`)
- Bubbles — agent messages: `#EBF2FA` → `#15273A` (info tint, text `#1F1A17` → `#F7F3F0`)
- Cards/panels: `#F6F2EF` → `#1F1A16` · elevated `#FFFFFF` → `#241E1A`
- Borders: `#E7DFD8` → `#3A322C` · stronger `#D4C8BE` → `#4C423A`
- Text: primary `#1F1A17` → `#F7F3F0` · secondary `#5C534C` → `#C4B8AE` · tertiary `#8A7F77` → `#94887D` · inverse `#FFFFFF` → `#171310`
- Links: `#9B111E` → `#F53D6D`
- Emergency ruby elements (panic banner, "Call now" in chat): fill `#9B111E` → `#F53D6D`, white text both modes
- Everyday primary buttons (Claim, Send, Continue): fill `#4A6659` → `#A3C4B0`, text `#FFFFFF` → `#171310`
- Priority/emergency pin badges: `#FFF5F7` bg + `#9B111E` text → `#1F1A16` bg + `#F53D6D` text, border `#FFC2D5` → `#3A322C`
- Unread count badges: `#9B111E` fill white text → `#F53D6D` fill `#171310` text
- Info / bot-paused badges: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Warning / out-of-hours badges: `#B7791F` on `#FBF3E4` → `#E3B341` on `#3B2E10`
- Success / claimed badges: `#2F7D5A` on `#EAF4EF` → `#4ADE9B` on `#123528`
- Error text: `#B8431F` → `#F0754A` · destructive fill `#D65328` → `#E85C30` · error surface `#FDF0E8` → `#3B1D10`
- Info: `#2B6CB0` on `#EBF2FA` → `#6BA3D6` on `#15273A`
- Disabled: `#EFE9E4` / `#8A7F77` → `#2A2420` / `#94887D`
- Focus ring: 3px `#E0115F` → 3px `rgba(245,61,109,0.4)`
- Shadows: raise opacity to 0.35+ on dark surfaces

---

## Group 1: Owner-Facing Bot

### Screen 1 — Bot Conversation Preview (menus + keyword replies)

```
Generate the WhatsApp bot conversation preview for ruby-veterinary — the automated triage chat as an owner sees it on the website chat widget.

DESKTOP LAYOUT (chat widget, 380px wide, bottom-right, 16px radius, shadow lg, backdrop panel white #FFFFFF, header bar sage-700 #3B5349 with white text):
- Header: clinic avatar circle, "ruby-veterinary triage bot" 15px weight 600 white, subline "Usually replies instantly" 12px rgba(255,255,255,.8), minimise × button (44px)
- Persistent phone fallback bar pinned under the header: #FFF5F7 background (dark #1F1A16), 1px #FFC2D5 border, phone icon #9B111E + "Prefer to talk? Call 010 555 0199" ruby #9B111E link 13px — always visible, never scrollable away
- Message list on chat canvas #FAF7F5 (dark #1F1A16), 16px padding, 12px gap:
  - Bot bubble (white, 12px radius with 4px tail corner, shadow sm, max-width 85%): "Hi! I'm the ruby-veterinary assistant. I can help with hours, appointments, refills, and directions. What do you need?"
  - Bot bubble with numbered menu: "Reply with a number:\n1 — Book an appointment\n2 — Prescription refill\n3 — Opening hours & directions\n4 — Speak to a human" (each line 15px/22px #1F1A17)
  - Owner bubble (right-aligned, #E3EDE6 fill, #1F1A17 text, 12px radius): "3"
  - Bot keyword reply: "We're open Mon–Fri 08:00–18:00, Sat 09:00–13:00, Sun closed. 14 Maple Street, Rosebank. Tap for directions" (directions as a #9B111E link) + quick-reply chips: "Book" · "Directions" · "4 — Human" (chips: white fill, 1px #E7DFD8, 8px radius, 44px min)
  - Bot menu reminder bubble with "4 — Speak to a human" emphasized
- Composer bar (white, 1px #E7DFD8 top border): text input "Type a message…" 44px, sage #4A6659 send button (44px, paper-plane icon)
- Out-of-hours variant badge: amber pill "After hours — we'll reply tomorrow 08:00" #B7791F on #FBF3E4 + line "For emergencies call 010 555 0199 or go to the nearest ER."

MOBILE LAYOUT (primary):
- Full-screen chat sheet: header, persistent phone fallback bar (full width, 48px, tappable), messages scroll, composer fixed at bottom above keyboard, quick-reply chips wrap
- Bubbles max-width 88%; numbered menu uses generous line height; chips full-width where they'd wrap
- 44px minimum on chips, send button, and the phone fallback row

ACCESSIBILITY:
- Chat widget is a labelled region; messages in a scrollable list with role="log" aria-live="polite" so new messages are announced
- Quick-reply chips are buttons with clear accessible names ("Reply: Speak to a human")
- Phone fallback is a real tel: link with the number in the accessible name; sits early in tab order
- Focus ring 3px #E0115F (dark rgba(245,61,109,0.4)); Esc closes widget, focus returns to launcher
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #1F1A17 on #E3EDE6 ≈ 14:1 · #9B111E on #FFF5F7 ≈ 7.9:1 · white on #4A6659 ≈ 5.9:1 · white on #3B5349 ≈ 8.1:1 · #B7791F on #FBF3E4 ≈ 4.7:1; dark: #F7F3F0 on #241E1A ≈ 15:1 · #F7F3F0 on #2A2420 ≈ 14:1 · #F53D6D on #1F1A16 ≈ 5.4:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #C4B8AE on #241E1A ≈ 9:1
- Tap targets ≥44px; text scaling to 200% must not clip menu lines

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 2 — Panic Escalation State

```
Generate the panic escalation state for ruby-veterinary — the bot detects a distress keyword ("bleeding", "poison", "can't breathe") and escalates immediately.

DESKTOP LAYOUT (same chat widget shell as Screen 1):
- Emergency banner inserted at the TOP of the message list, above the last message: full-width within the widget, background #FFF5F7 (dark #1F1A16), 4px top border ruby #9B111E (dark #F53D6D), 12px radius, padding 12px:
  - Row 1: alert-triangle icon 20px #9B111E + heading 15px weight 700 #9B111E "This sounds urgent"
  - Row 2: 14px/20px #1F1A17 "Call us now on 010 555 0199 — a person will pick up. If you can't reach us, go straight to the nearest emergency veterinary hospital."
  - Row 3: ruby #9B111E filled "Call 010 555 0199" button (48px, white text, phone icon) + outlined secondary "Find nearest ER" (44px)
  - Banner is pinned (not dismissible while the conversation continues)
- Priority badge next to the bot name in the header: pill #FFF5F7 bg, #9B111E text, 11px weight 700, "EMERGENCY PRIORITY" (dark: #1F1A16 / #F53D6D)
- Bot bubble after banner: "I've flagged this as urgent and paged the clinic team. While you wait, please call — don't rely on this chat." (15px/22px)
- Bot bubble with the number repeated as a tel: link in #9B111E weight 600
- Escalation confirmation chip: amber pill "Paged a receptionist · 14:32" #B7791F on #FBF3E4 with clock icon
- Phone fallback bar under the header remains present (now redundant but persistent)

MOBILE LAYOUT (primary):
- Banner sits directly under the persistent phone bar, full width, stacks: icon+heading, body text, full-width ruby "Call 010 555 0199" (56px), full-width outlined "Find nearest ER" (48px)
- Priority badge shrinks to "URGENT" but keeps icon + text
- Composer remains usable but a sticky ruby "Call now" action bar may sit above it (56px) — the chat never blocks calling
- Everything ≥44px tap targets

ACCESSIBILITY:
- Escalation banner is a complementary/alert region (role="alert" on first appearance, not re-announced on every message)
- Ruby is used ONLY here for the emergency CTA — correct any bleed into other buttons
- Icon + heading + text carry the meaning; badge text "EMERGENCY PRIORITY" not colour alone
- Focus moves to the banner's Call button when escalation occurs (or is reachable immediately after the triggering message)
- Contrast: #9B111E on #FFF5F7 ≈ 7.9:1 · #1F1A17 on #FFF5F7 ≈ 15:1 · white on #9B111E ≈ 8.4:1 · #B7791F on #FBF3E4 ≈ 4.7:1; dark: #F53D6D on #1F1A16 ≈ 5.4:1 · #F7F3F0 on #1F1A16 ≈ 16:1 · white on #F53D6D ≈ 4.6:1 (large button text; number also in body text for redundancy) · #E3B341 on #3B2E10 ≈ 7.4:1
- No flashing or pulsing animation; respect prefers-reduced-motion

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Group 2: Staff Inbox

### Screen 3 — Staff Inbox List

```
Generate the staff shared WhatsApp inbox list for ruby-veterinary — multiple receptionists seeing who has claimed what. Desktop-first back office.

DESKTOP LAYOUT (three-pane shell: 280px filter rail · list 360px · preview/empty right pane — show rail + list + placeholder pane):
- App header: "Shared inbox" H1 (24px in the shell) + channel badge "WhatsApp Business · verified" info pill (#EBF2FA/#2B6CB0) + agent identity "K. Patel (you)" avatar + status select (Available/Away)
- Filter rail: conversation counts — All 24 · Unclaimed 6 · Claimed by me 4 · Emergency 2 · Unread 9; below: "Out-of-hours mode" toggle card on #FBF3E4 with warning icon #B7791F "Away message active 18:00–08:00"
- Conversation list (white, 1px #E7DFD8 column border):
  - Emergency pins section FIRST: header "Pinned · emergency" 12px uppercase #9B111E weight 700; rows carry a 4px left border #9B111E and a ruby pill "EMERGENCY" (#FFF5F7/#9B111E, 11px weight 700) — the only ruby in the list
  - Row anatomy (72px tall, 1px #E7DFD8 bottom border, hover #FAF7F5): avatar circle, name "Naledi Dlamini" 15px weight 600 #1F1A17, snippet "…my dog is bleeding a lot…" 13px #5C534C (truncate one line), timestamp 12px #8A7F77
  - Claim status chip per row: "Unclaimed" grey (#EFE9E4/#5C534C, dot icon) · "Claimed · J. Mbeki" green (#EAF4EF/#2F7D5A, user-check icon) · "Bot only" info (#EBF2FA/#2B6CB0, bot icon)
  - Unread badge: ruby-ish count pill — use ruby #9B111E fill with white text ONLY for the unread count badge (acceptable accent) 12px weight 700
  - Selected row: 2px left border #4A6659, background #F4F8F5
- Right pane empty state: sage-50 panel, message icon, "Select a conversation", body "Emergencies are pinned to the top so nothing urgent waits behind routine questions."
- Search input at top of list (40px), sort select "Newest activity"

MOBILE LAYOUT (mobile note, not primary):
- Single column: shell header, emergency section first, filter chips horizontal scroll, rows full width with chips on a second line, unread badge right, selected row opens full-screen conversation; out-of-hours toggle accessible from header

ACCESSIBILITY:
- List is a listbox/list of links with aria-current on the selected row; claim status and emergency state always include text labels (icons aria-hidden)
- Unread counts announced as text in the row's accessible name ("Naledi Dlamini, 2 unread messages, unclaimed")
- Focus ring 3px #E0115F on rows, chips, toggle; keyboard nav through the list (up/down arrows or tab, no scroll traps)
- Contrast: #1F1A17 on #FFFFFF ≈ 16:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (timestamps only) · white on #9B111E ≈ 8.4:1 · #2F7D5A on #EAF4EF ≈ 4.6:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #5C534C on #EFE9E4 ≈ 5.4:1 · #B7791F on #FBF3E4 ≈ 4.7:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #C4B8AE on #1F1A16 ≈ 9:1 · #94887D ≈ 4.6:1 (metadata) · #F53D6D fill with #171310 count ≈ 5.4:1 · #4ADE9B on #123528 ≈ 8:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #C4B8AE on #2A2420 ≈ 8:1 · #E3B341 on #3B2E10 ≈ 7.4:1
- Rows ≥64px (≥72px touch on mobile collapsed cards); row is fully keyboard reachable

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 4 — Conversation View with Claim Button

```
Generate the staff conversation view for ruby-veterinary — timeline, claim action, and agent composer.

DESKTOP LAYOUT (continues the three-pane shell; centre pane is the conversation):
- Conversation header: name "Naledi Dlamini" 16px weight 600 + "· +27 72 555 0134" 13px #8A7F77 mono-ish, channel icon, priority chip "EMERGENCY" ruby pill (#FFF5F7/#9B111E) or "Routine" grey pill, waiting timer "Unclaimed · 4m" 13px #B7791F weight 600, right actions: sage #4A6659 filled "Claim conversation" (44px, user-check icon) — primary action until claimed; outlined "Assign to…" dropdown; more ⋯ button
- Emergency pin banner (if pinned): #FFF5F7 bg, 4px left border #9B111E, alert icon, "Flagged by the bot: 'bleeding'. Call the owner if no reply in 2 minutes." + ruby "Call owner" link
- Timeline on #FAF7F5 (dark #1F1A16): date separators centred 12px #8A7F77 on a hairline; bubbles —
  - Bot messages (left, white, 12px radius, small "BOT" caption 10px uppercase #8A7F77)
  - Owner messages (left, white too, with sender caption "Owner")
  - Agent messages (right, #EBF2FA info tint, caption "K. Patel" 10px #8A7F77)
  - System events centred chips: "Handover requested 14:31" · "Claimed by K. Patel 14:32" · "Bot paused" — grey pill #EFE9E4/#5C534C, 12px
  - Timestamps 11px #8A7F77 under bubbles, right-aligned on own bubbles
- Internal note divider: dashed 1px #D4C8BE with centred label "Internal — not visible to the client" 11px uppercase #8A7F77; note bubble on #FBF3E4 with warning-free styling, text 14px #1F1A17
- Composer (white, 1px #E7DFD8 top): internal-note toggle switch (label "Internal note"), textarea "Reply…" 44px min, sage #4A6659 "Send" button (44px) with "Enter to send · Shift+Enter for newline" helper 11px #8A7F77

MOBILE LAYOUT (mobile note, not primary):
- Two-pane collapses: header with name + chips + Claim button (full width sticky under header, sage), timeline full width, composer sticky at bottom with internal-note toggle above the field, "Call owner" ruby action available in the header overflow

ACCESSIBILITY:
- Claim button announces state change ("Claimed by K. Patel") via role="status"; while unclaimed it is early in the tab order
- Timeline is a log region (role="log", aria-live="polite"); system events are text chips, not colour-only
- Internal vs client-visible messages are labelled in text ("Internal") — never by tint alone
- Focus ring 3px #E0115F on header actions, bubbles' actions, toggle, send; keyboard completes claim → type → send
- Contrast: #1F1A17 on #FAF7F5 ≈ 15:1 · #1F1A17 on #EBF2FA ≈ 15:1 · #8A7F77 timestamps ≈ 4.0:1 on #FAF7F5 (supplementary only) · white on #4A6659 ≈ 5.9:1 · #9B111E on #FFF5F7 ≈ 7.9:1 · #5C534C on #EFE9E4 ≈ 5.4:1 · #1F1A17 on #FBF3E4 ≈ 14:1; dark: #F7F3F0 on #1F1A16 ≈ 16:1 · #F7F3F0 on #15273A ≈ 14:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #F53D6D on #1F1A16 ≈ 5.4:1 · #C4B8AE on #2A2420 ≈ 8:1 · #F7F3F0 on #3B2E10 ≈ 13:1
- Tap targets ≥44px (composer send 44px, claim 48px on mobile)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

### Screen 5 — Handover Claimed State (bot paused)

```
Generate the claimed handover state for ruby-veterinary — a human owns the conversation, the bot is visibly paused.

DESKTOP LAYOUT (same conversation pane, post-claim):
- Header transforms: "Claim conversation" replaced by a green success chip "Claimed · K. Patel" (#EAF4EF fill, #2F7D5A text, user-check icon, pill) + secondary chip "You are the assigned agent" 12px #5C534C; outlined "Release" button (44px) with helper tooltip "Returns the conversation to the unclaimed queue"; outlined "Transfer" dropdown; emergency chip stays if pinned
- Bot paused badge (prominent, directly under the header): full-width strip on #EBF2FA with info icon #2B6CB0 + text 14px weight 600 #1F1A17: "Bot paused — K. Patel is replying. Automated replies resume when you release the conversation." + outlined "Resume bot" small button (36px) with confirm note 12px #8A7F77 "Only if you mean to hand back to automation"
- Timeline additions:
  - System chip: "Handover claimed by K. Patel · 14:32" (green dot + text)
  - New agent bubble "Thanks Naledi — I've got you. Is the bleeding from a cut or is she coughing blood? Meanwhile keep her warm and don't clean the wound yet." right-aligned, #EBF2FA
  - Owner typing indicator chip: three dots + "Naledi is typing…" 12px #8A7F77
- SLA/waiting footer under timeline: "First human reply in 47s · Target under 2 minutes" 12px #2F7D5A with check icon (amber #B7791F if over target)
- Composer now enabled with full send; internal-note toggle available; quick canned replies button ("Canned replies ⌘K")

MOBILE LAYOUT (mobile note, not primary):
- Header stacks: claim chip row + release/transfer icons; bot-paused strip full width (icon above text); timeline full width; composer sticky with send 44px; release requires confirmation sheet (error-free — informational, sage confirm)

ACCESSIBILITY:
- Claim state announced once via role="status" ("Claimed by K. Patel"); bot-paused strip is a live region only when it changes
- Paused state conveyed by icon + text ("Bot paused"), never by colour alone; release/resume are labelled buttons
- Focus ring 3px #E0115F; keyboard order: claim/release → bot controls → timeline → composer
- Contrast: #2F7D5A on #EAF4EF ≈ 4.6:1 · #2B6CB0 on #EBF2FA ≈ 6.2:1 · #1F1A17 on #EBF2FA ≈ 15:1 · #5C534C on #FFFFFF ≈ 7.4:1 · #8A7F77 on #FFFFFF ≈ 3.9:1 (helper text) · white on #4A6659 ≈ 5.9:1 · #B7791F on #FBF3E4 ≈ 4.7:1 (SLA warning); dark: #4ADE9B on #123528 ≈ 8:1 · #6BA3D6 on #15273A ≈ 5.5:1 · #F7F3F0 on #15273A ≈ 14:1 · #F7F3F0 on #241E1A ≈ 15:1 · #94887D ≈ 4.6:1 · #A3C4B0 fill with #171310 ≈ 9.4:1 · #E3B341 on #3B2E10 ≈ 7.4:1
- Tap targets ≥44px; reduced-motion: no bouncing typing dots animation (static indicator)

Generate in BOTH light and dark mode, for desktop and mobile (4 total: Desktop Light, Desktop Dark, Mobile Light, Mobile Dark). Apply the Dark Mode Color Mapping above. Same layout for all — only colours change.
```

---

## Credits Estimate

| Group | Screens | Estimated Credits |
|-------|---------|-------------------|
| Design System Context (paste first, not generated) | — | 0 |
| Owner-Facing Bot | 2 | ~10 |
| Staff Inbox | 3 | ~15 |
| **Total** | **5** | **~25** |

---

## Usage Instructions

1. Paste `master-prompt.md` first (once per session) so DESIGN.md exists on the canvas.
2. Paste the Design System Context block above, then generate screens one at a time.
3. Verify ruby discipline after each: ruby appears only on emergency/Call elements and the unread count badge; Claim/Send/Resume must be sage #4A6659.
4. Useful follow-ups: "Keep the phone fallback bar pinned above the composer" · "Make the panic banner non-dismissible" · "Apply the dark mode mapping to the chat bubbles."

---

## Related Documents

- `../design-system.md` — authoritative brand palette and tokens
- `09-staff-backoffice.md` — admin alerts and the embedded shared inbox note
- `10-shared-states-mobile.md` — degraded WhatsApp banner with phone fallback
