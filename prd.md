# prd.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Product Requirements Document
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Core Business Domain

---

# Purpose

This document consolidates the product requirements for the ruby-veterinary website: what is being built, for whom, in what order, and how success is judged.

It is derived from `vision.md`, `functional-requirements.md`, and `non-functional-requirements.md`, and is the reference against which scope decisions are made. Where this document and the requirements checklists disagree, the checklists are the source of truth for detail; this document is the source of truth for scope and sequence.

---

# 1. Executive Summary

## 1.1 Product Overview

ruby-veterinary is a single veterinary clinic's complete digital front door: an informational website with emergency-first contact display, online booking and client intake, an educational blog operated by clinic staff, a WhatsApp triage bot with human handover, and an online store selling general supplies, therapeutic diets, and prescription items behind a veterinarian authorisation workflow.

## 1.2 Key Value Propositions

| For | Value |
|-----|-------|
| Pet owners in an emergency | Phone, address, and hours visible within 3 seconds on any page |
| New and returning clients | Register, transfer records, book, and order without calling |
| Clinic staff | Content, pricing, and prescription decisions managed without developers |
| The clinic | Reachable 24/7, recurring product revenue, discoverable through local search |

## 1.3 Business Objectives

- Convert website visits into appointment requests and product sales
- Shift routine enquiries (hours, directions, refills) from phone calls to self-service
- Grow recurring revenue through subscriptions for preventatives and wellness products
- Build local search authority through consistently published, clinician-authored content

## 1.4 Technical Approach

A three-repository monorepo of submodules: a **Next.js 16** public storefront and CMS front end, a **NestJS 11** API, and this documentation repository, with **PostgreSQL** as the system of record. Payments, WhatsApp, and email are delegated to verified third-party services so that no card data or unverified medical messaging touches clinic infrastructure. Full detail in `architectural-decision-record.md`.

---

# 2. Vision & Mission

The vision, problem statement, guiding principles, and scope boundaries are maintained in `vision.md`. In summary:

> Every owner in the clinic's care has one trusted place to turn - at any hour, from any device - to reach a veterinarian, understand their options, and get what their animal needs.

Five principles govern every trade-off: emergency before commerce; safety rails over conversion; verified channels and privacy by default; graceful degradation never a dead end; mobile-first and accessible to everyone.

---

# 3. Problem Statement

1. **Emergency contact is hard to find.** Owners in a panic must hunt for a phone number, address, and hours.
2. **Access to care is phone-only.** Intake, record transfer, and consent paperwork are slow and manual.
3. **Repeat demand has no channel.** Refills, preventatives, and therapeutic diets require a call or a visit.
4. **No always-on presence.** At night and on weekends, owners needing triage have nowhere to turn.
5. **Staff lose hours to repetitive triage.** Common questions consume receptionist time.

---

# 4. Target Users & Personas

Full personas with goals, pain points, and platform usage are defined in `user-personas.md`.

| Group | Personas | Primary Need |
|-------|----------|--------------|
| Pet owners (public) | Emergency Seeker, New Client, Routine Care Owner, Repeat Buyer | Fast answers, self-service booking, safe online purchasing |
| Clinic staff (internal) | Receptionist, Veterinarian, Vet Technician, Practice Manager | Alerts, shared inbox, prescription authorisation, no-code back office |

Identity model: anonymous browsing for all public content; client accounts for intake, orders, and subscriptions; role-scoped staff accounts for every back-office capability.

---

# 5. Product Positioning

- **Category:** Local clinic website with integrated booking, content, messaging, and commerce
- **Differentiator:** Emergency clarity in three seconds combined with a full safety-guardrailed storefront
- **Brand personality:** Calm under pressure, clinically credible, plain-spoken, never salesy in an emergency context

---

# 6. Core Features & Capabilities

All requirements are itemised as checklists in `functional-requirements.md`. This section groups them into modules and sequences them.

## 6.1 Module Map

| # | Module | Capability Summary |
|---|--------|--------------------|
| 1 | Public site & intake | Persistent emergency header (phone, address, hours), service catalog pages with baseline pricing, staff profile grids, appointment request form, new-client registration with PDF/JPEG history uploads, public document downloads |
| 2 | Blog & CMS | Rich text editor with images, video, and links; categories and tags; author profiles per article; social sharing (Facebook, WhatsApp, Pinterest); newsletter sign-up embed; related-article feed; per-post SEO meta titles, descriptions, and clean URLs |
| 3 | WhatsApp care bot | Keyword auto-responders, panic-keyword emergency routing, numbered pre-screening menus, human handover paging, out-of-hours away-message flow |
| 4 | E-commerce | Single, variable, and composite products; categorized inventory grid (Prescription Medication, Therapeutic Diets, General Pet Supplies); detailed product pages; search and filters by pet type, life stage, condition; persistent cart with tax/shipping calculation; payment gateway checkout; shipping or free in-clinic pickup; Rx authorisation hold; mandatory patient and vet association fields; auto-refill and subscription billing |
| 5 | Operations & admin | Admin alert dashboard for form submissions and bot escalations; central shared WhatsApp inbox for multiple receptionists |
| 6 | Security & compliance | HTTPS everywhere, cookie consent banner, visible privacy policy, PCI-DSS handled entirely by tokenised gateways, verified WhatsApp Business API for medical context |

## 6.2 Sequencing

| Phase | Scope | Rationale |
|-------|-------|-----------|
| **P1 - Foundation** | Module 1, Module 6, Module 5 (email alerts only) | Emergency clarity and lead capture deliver value with no payment or messaging dependencies |
| **P2 - Content** | Module 2 | Builds local search authority and gives staff a reason to return to the back office daily |
| **P3 - Messaging** | Module 3 (bot) + Module 5 (shared inbox) | Requires verified WhatsApp Business API approval; automated triage reduces phone load |
| **P4 - Commerce** | Module 4, general products first | Storefront and payments are the largest integration surface |
| **P5 - Clinical commerce** | Rx authorisation workflow, patient/vet fields, subscriptions | Ships only once prescription review UX is proven with staff; highest risk, therefore last |

## 6.3 Explicit Non-Goals

Online video consultations, pet insurance, multi-clinic or chain support, native mobile apps, in-house payment processing. See `vision.md` section 8.

---

# 7. Non-Functional Requirements

Full quality checklist in `non-functional-requirements.md`. Binding commitments:

| Area | Commitment |
|------|-----------|
| Performance | Homepage and emergency pages load under 3s on standard 4G; bot responses within 2s |
| Availability | Minimum 99.9% uptime; graceful degradation to plain-text fallbacks when integrations fail |
| Accessibility | WCAG 2.1 AA, keyboard navigation, alt text, high contrast; 3-second UX clarity on emergency info |
| Security | TLS on all traffic; PCI-DSS via tokenised gateways only; verified WhatsApp Business API for medical content |
| Scalability | No-code back office for staff; catalog scales to hundreds of SKUs without latency |
| Media | Automatic image compression (WebP) across CMS and catalog |

---

# 8. Technical Architecture

Decisions and rationale in `architectural-decision-record.md`; API contracts in `api-specification.md`.

```
Browser (mobile-first)
   |  HTTPS (edge TLS)
   v
Cloudflare CDN / DNS  --->  static caching, DDoS filter
   |  HTTPS (origin)
   v
Single VPS: nginx reverse proxy
   |
Next.js 16 storefront + admin UI
   |  REST (JWT)
   v
NestJS 11 API (modular monolith)
   |            |             |              |
PostgreSQL   MinIO object   Stripe-class    WhatsApp
 (system of  storage        payment         Business API
  record)   (uploads)       (tokenised)     (verified)
                \             |             /
                 ---- Email / alert delivery ----

Daily encrypted backups -> off-site S3-compatible bucket (separate provider)
```

| Concern | Choice |
|---------|--------|
| Front end | Next.js 16, React 19, Tailwind 4, TypeScript |
| API | NestJS 11, TypeScript, REST-first, modular monolith |
| Database | PostgreSQL, single system of record |
| Payments | External tokenised gateway; card data never stored |
| Messaging | Verified WhatsApp Business API plus a shared multi-agent inbox |
| Storage | MinIO (S3-compatible) on the VPS for medical-history uploads and media |
| Hosting | Single VPS under Docker Compose, behind Cloudflare (ADR-0014) |
| Repositories | Three git submodules under `ruby-veterinary-service` |

---

# 9. User Experience

Design principles derive from `vision.md`:

- **Emergency before commerce** - emergency contact is persistent chrome on every page
- **3-second clarity** - location, hours, and phone identifiable without scrolling or opening menus
- **Mobile-first** - thumb-friendly targets, sticky contact options, tap-to-call
- **Plain language** - no jargon in owner-facing flows; Rx holds explained as normal and expected
- **Accessible by default** - WCAG 2.1 AA as a floor, not an audit-time afterthought

Key journeys: emergency discovery; new-client intake with uploads; appointment request; blog discovery and newsletter signup; browse-filter-buy with Rx hold; human handover from the bot.

---

# 10. Roadmap

| Phase | Outcome | Exit Criteria |
|-------|---------|---------------|
| P1 Foundation | Public site live with intake forms and compliance pages | All Module 1 and 6 checklist items verified; alerts reaching staff |
| P2 Content | Staff publishing independently | An article drafted, edited, and published by non-technical staff without a developer |
| P3 Messaging | Bot triaging, humans taking over | Emergency keywords escalate in under 2s; handover reaches the shared inbox |
| P4 Commerce | Storefront transacting | Cart, tax, payment, shipping/pickup flows complete end to end |
| P5 Clinical commerce | Rx workflow and subscriptions live | A prescription order is blocked, reviewed, authorised, and fulfilled by real staff |

Each phase is gated on the acceptance criteria in `functional-requirements.md` rather than on calendar dates.

---

# 11. Success Metrics

Qualitative signals and the binding numeric targets live in `vision.md` section 6. Additional business metrics are named here for baseline tracking; targets are set only once Phase 1 traffic exists.

| Category | Metric |
|----------|--------|
| Reachability | Emergency phone/address/hours identified in under 3 seconds; tap-to-call usage |
| Reliability | 99.9% uptime; homepage and emergency pages under 3s on 4G; bot under 2s |
| Conversion | Appointment requests submitted; registrations completed; carts completed |
| Commerce | Orders fulfilled post-authorisation; subscription retention; Rx turnaround |
| Operations | Unanswered WhatsApp messages; staff content published without developer involvement |
| Compliance | Zero card-data incidents; zero unverified medical messages |

---

# 12. Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| WhatsApp Business API approval delayed | Phase 3 slips | Ship P1-P2 without it; fall back to click-to-chat deep links; degradation path defined |
| Payment gateway integration issues | Revenue blocked | Use a mainstream tokenised provider; test with sandbox first; never handle card data ourselves |
| Prescription workflow rejected by staff UX | P5 slips | Prototype review screen early with a real veterinarian; keep manual override always available |
| Content stagnation after launch | SEO value lost | Category/tag structure and author profiles ship with the CMS so publishing is frictionless |
| Integration failure at runtime | Owners left stranded | Graceful degradation requirement: plain-text instructions and a phone number always available |
| Scope creep toward multi-clinic features | Delay to core value | Non-goals fixed in `vision.md` section 8; changes require an update to this document |

---

# 13. Dependencies

| Dependency | Type | Used By |
|------------|------|---------|
| Verified WhatsApp Business API account | External approval | Module 3, emergency routing |
| Multi-agent WhatsApp inbox tool (e.g., ManyChat, Sirena, WhatsApp Business App) | Third-party SaaS | Module 5 shared inbox |
| Payment gateway (Stripe, PayPal, or Apple Pay class) | Third-party SaaS | Module 4 checkout |
| Transactional email delivery | Third-party SaaS | Form routing, alerts, newsletter |
| Single VPS hosting provider (Ubuntu, Docker Compose) | Infrastructure | All modules (`deployment-architecture.md`) |
| Cloudflare DNS/CDN | Third-party SaaS (free tier) | Edge TLS, static caching, DDoS protection |
| Off-site backup bucket on a separate provider | Third-party SaaS | Daily PostgreSQL and MinIO backups |

---

# 14. Open Questions

1. Which payment gateway is selected as primary, and which as secondary?
2. Which multi-agent WhatsApp inbox tool is adopted?
3. Does the clinic have existing inventory and SKU data to import, or is the catalog built from scratch?
4. What is the baseline phone-call volume, so self-service impact can be measured?
5. Which staff member owns the newsletter, and at what cadence?
6. Which VPS provider and plan (RAM/vCPU) are selected, and where is the domain's DNS hosted?

---

# 15. Appendix

## 15.1 Related Documents

| Document | Relationship |
|----------|-------------|
| `vision.md` | Purpose, problem, principles, scope boundaries |
| `functional-requirements.md` | Detailed feature checklists (source of truth for detail) |
| `non-functional-requirements.md` | Quality attributes and targets (source of truth for quality) |
| `user-personas.md` | Detailed personas and identity model |
| `domain-model.md` | Entities and domains behind these features |
| `architectural-decision-record.md` | Why the stack was chosen |
| `api-specification.md` | Contracts between the front end and API |

## 15.2 Revision History

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-10-06 | Initial PRD derived from vision and requirements checklists |
