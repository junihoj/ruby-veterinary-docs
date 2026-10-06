# user-personas.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** User Personas
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

This document defines the people who use the ruby-veterinary website, what they are trying to achieve, and what gets in their way.

Understanding these audiences is essential for prioritising the emergency experience over everything else, for designing intake and checkout flows that reduce front-desk workload, and for deciding which capabilities staff need versus which clients need.

ruby-veterinary serves two broad groups: **pet owners** (clients, usually unregistered visitors or account holders) and **clinic staff** (internal operators with role-based access). A single owner may have several pets on file; a single staff member may hold several roles (for example, a veterinarian who is also a blog author and the prescriber who authorises pharmacy orders).

For detailed functional behaviour, see `functional-requirements.md`. For quality expectations that shape these personas (mobile-first, 3-second clarity, accessibility), see `non-functional-requirements.md`.

---

# Persona Overview

```
                        ruby-veterinary Users
                                  |
        +-------------------------+-------------------------+
        |                                                   |
  Pet Owners (Clients)                              Clinic Staff
  mostly mobile, mostly public                      role-based access
        |                                                   |
  - Emergency Seeker                              - Receptionist
  - New Client                                    - Veterinarian
  - Routine Care Owner                            - Vet Technician
  - Repeat Buyer                                  - Practice Manager
```

| Group | Account Requirement | Primary Surfaces |
|-------|--------------------|------------------|
| Pet Owners | Optional for browsing and emergency info; required for saved details and order history | Mobile site, WhatsApp, email |
| Clinic Staff | Required, role-based, staff-only admin areas | Desktop back office, shared WhatsApp inbox |

---

# Primary Personas

## Pet Owners (Clients)

### Emergency Seeker

| Attribute | Description |
|-----------|-------------|
| **Who** | A pet owner whose animal is bleeding, poisoned, limping, or otherwise in distress, arriving from a search engine, a map listing, or a shared link |
| **Goals** | Reach a human being at the clinic immediately, know whether the clinic is open now, get directions, understand whether the situation is an emergency |
| **Pain Points** | Cannot find a phone number or address quickly; pages that load slowly on mobile data; sites that bury contact details behind menus; automated systems that do not understand urgency |
| **Behaviors** | Arrives on a phone, scans rather than reads, taps the first phone number visible, may abandon and try a competitor after a few seconds |
| **Success Metrics** | Emergency phone number, address, and hours identified within 3 seconds of landing; tap-to-call used |
| **Platform Usage** | Single, short, mobile-first session; often on a slow or congested network |
| **Design Implication** | Emergency details must be persistent and visible on every page, never behind navigation. Keyword detection in the WhatsApp bot must escalate immediately. |

### New Client

| Attribute | Description |
|-----------|-------------|
| **Who** | An owner registering with the clinic for the first time, often before a first visit or after a recommendation |
| **Goals** | Register quickly, transfer past medical history so the veterinarian is prepared, understand services and pricing, book a first appointment |
| **Pain Points** | Long paper forms re-typed by reception, uncertainty about required documents, unclear pricing, no confirmation that a request was received |
| **Behaviors** | Browses service pages and staff profiles for reassurance before committing, fills the registration form on desktop or phone, uploads PDF/JPEG history files, submits an appointment request with preferred time windows |
| **Success Metrics** | Registration completed, history files uploaded, appointment request submitted, confirmation received |
| **Platform Usage** | Two to four sessions over several days; mixed mobile and desktop |
| **Design Implication** | Intake must be resumable-friendly and forgiving; uploads need clear accepted formats and a visible success state. |

### Routine Care Owner

| Attribute | Description |
|-----------|-------------|
| **Who** | An existing client booking vaccinations, dental care, surgeries, or check-ups, and researching clinic guidance |
| **Goals** | Book an appointment with minimal friction, see transparent baseline pricing, verify credentials of the staff who will treat their pet, find trustworthy care advice |
| **Pain Points** | Phone-only booking during working hours, hidden or missing pricing, generic internet advice contradicting the clinic's guidance |
| **Behaviors** | Uses the appointment request form, reads service landing pages and staff profiles, follows the clinic blog via social shares or newsletter, downloads post-op care sheets and waivers |
| **Success Metrics** | Appointment requests submitted, care documents downloaded, newsletter subscription, return visits to blog content |
| **Platform Usage** | Weekly to monthly, mostly mobile, sometimes desktop for reading longer articles |
| **Design Implication** | Trust signals (credentials, accreditations, author bylines) must be adjacent to the content they validate. |

### Repeat Buyer

| Attribute | Description |
|-----------|-------------|
| **Who** | A returning client purchasing prescription medication, therapeutic diets, preventatives, and general supplies |
| **Goals** | Reorder familiar items quickly, subscribe to monthly preventatives, choose shipping or free in-clinic pickup, complete a prescription order without a phone call |
| **Pain Points** | Checkout friction, forced re-entry of pet details, uncertainty about whether a prescription will be approved, surprise shipping costs, cards failing at checkout |
| **Behaviors** | Filters the catalog by pet type, life stage, or condition; buys variable-weight variants; adds bundles; expects orders to hold pending veterinary authorisation for prescription items |
| **Success Metrics** | Orders completed, subscriptions started and retained, orders fulfilled after Rx authorisation, repeat purchase rate |
| **Platform Usage** | Recurring, mobile-heavy, price and availability sensitive |
| **Design Implication** | The Rx hold must be communicated as normal and expected, not as an error; auto-refill must be transparent about billing dates. |

---

## Clinic Staff (Internal Operators)

### Receptionist / Front Desk

| Attribute | Description |
|-----------|-------------|
| **Who** | The first human contact for the clinic: answering calls, triaging messages, managing the schedule |
| **Goals** | Clear the queue of incoming requests quickly, keep no enquiry unanswered, hand off complex conversations without losing context |
| **Pain Points** | Repeating the same answers about hours and directions, messages arriving across disconnected channels, being unable to tell which messages are urgent |
| **Behaviors** | Monitors admin alerts for new form submissions, works from the shared WhatsApp inbox on a desktop computer, responds when the bot hands over a conversation, triages appointment requests into the schedule |
| **Success Metrics** | Response time to handovers, unanswered-message count, appointment requests converted |
| **Platform Usage** | Continuous during opening hours, desktop-first, multiple agents on the same inbox simultaneously |
| **Design Implication** | Alerts must name the source and urgency; the inbox must support concurrent agents without message collisions. |

### Veterinarian

| Attribute | Description |
|-----------|-------------|
| **Who** | A licensed veterinarian at the clinic; the clinical authority for prescriptions and medical content |
| **Goals** | Approve or reject prescription orders accurately, publish trusted educational content, appear credibly to prospective clients |
| **Pain Points** | Approving orders without the patient's record in view, content requests piling up with technical staff, liability exposure from careless publishing |
| **Behaviors** | Reviews flagged prescription orders alongside pet and owner details before authorising, writes and reviews blog articles under their own byline, holds a public staff profile with credentials and accreditations |
| **Success Metrics** | Prescription decisions made within one business day, articles published, profile completeness |
| **Platform Usage** | Several short sessions daily, desktop and mobile |
| **Design Implication** | The authorisation screen must surface patient history and prescriber identity in one view; bylined content must be attributable and editable. |

### Vet Technician / Nurse

| Attribute | Description |
|-----------|-------------|
| **Who** | Support staff assisting with treatments, recovery, and client education |
| **Goals** | Help produce accurate care content, keep client handouts current, support reception during peak messaging hours |
| **Pain Points** | No simple way to publish or update guidance, duplication of effort with other staff |
| **Behaviors** | Drafts and updates blog posts and downloadable care sheets, may answer WhatsApp conversations when licensed to do so, holds a staff profile |
| **Success Metrics** | Content updated without developer involvement, care sheets kept current |
| **Platform Usage** | Daily, desktop for authoring |
| **Design Implication** | The CMS must be operable by non-technical staff with no developer in the loop. |

### Practice Manager / Clinic Owner

| Attribute | Description |
|-----------|-------------|
| **Who** | The person accountable for the business: revenue, staffing, and the clinic's public reputation |
| **Goals** | Grow online revenue, keep the storefront and pricing accurate, measure whether the website is producing bookings and sales, stay compliant |
| **Pain Points** | Reliance on developers for every change, no visibility into form submissions or abandoned carts, compliance anxiety around payments and client data |
| **Behaviors** | Adjusts product prices and publishes content through visual tools, reviews admin dashboards and alerts, monitors subscription revenue and order volume, owns privacy and cookie-consent policy |
| **Success Metrics** | Booking and sales conversion, content output, uptime and page-load targets met, no security or compliance incidents |
| **Platform Usage** | Daily check-ins, desktop-first, occasional mobile review |
| **Design Implication** | Every operational control the manager needs must exist in a no-code back office; reporting must be readable without SQL. |

---

# Identity Model

| Account Type | Who Holds It | Capabilities |
|--------------|--------------|--------------|
| **Anonymous visitor** | Anyone landing on the site | Browse services, staff, blog, catalog; download public documents; emergency contact via tap-to-call |
| **Client account** | Registered pet owners | Appointment requests, new-client intake with file uploads, order history, subscriptions, saved pet and vet associations |
| **Staff account** | Clinic employees, role-scoped | Back office: publish content, manage catalog and pricing, manage appointments, authorise prescriptions, operate the shared WhatsApp inbox, receive alerts |

Staff roles are additive: a veterinarian is also a content author and a prescription authoriser. Client accounts never receive staff capabilities, and staff capabilities are never granted through public registration.

Pets are not users. A pet is a record owned by a client and associated with the treating veterinarian, which is what makes the mandatory patient-and-vet fields at checkout work.

---

# Cross-Cutting Concerns

## Multi-Role Behavior

| Pattern | Description |
|---------|-------------|
| **Owner + Repeat Buyer** | The same client books care and shops in one session |
| **Owner + Reader** | An owner who follows the blog and newsletter but has no active booking |
| **Veterinarian + Author + Authoriser** | One staff member writes content and approves prescription orders |
| **Receptionist + WhatsApp Agent** | One staff member handling both the shared inbox and appointment triage |

## Device Usage

| Persona | Primary Device | Secondary Device |
|---------|---------------|------------------|
| Emergency Seeker | Mobile | None (single session) |
| New Client | Mobile or desktop | Alternates during intake |
| Routine Care Owner | Mobile | Desktop for long-form reading |
| Repeat Buyer | Mobile | Desktop for larger orders |
| Receptionist | Desktop | Phone during out-of-hours rota |
| Veterinarian | Desktop | Mobile for authorisation alerts |
| Practice Manager | Desktop | Mobile for dashboards |

## Accessibility and Out-of-Hours Reality

Every persona must be served without dependence on color perception, precise pointer input, or fast networks: WCAG 2.1 AA conformance, thumb-friendly targets, and plain-language fallbacks apply to all of them. Outside opening hours, every owner persona falls back to the same out-of-hours flow pointing exclusively to emergency care.

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `vision.md` | Source of the audience segments and guiding principles these personas elaborate |
| `functional-requirements.md` | Feature checklist each persona's behaviors map to |
| `non-functional-requirements.md` | Quality bar (speed, accessibility, uptime) each persona depends on |
| `prd.md` | Consolidated product requirements referencing these personas |
| `domain-model.md` | Entities (Client, Pet, Staff, Order) derived from these personas |
