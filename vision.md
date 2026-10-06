# Vision — ruby-veterinary

## 1. Purpose

ruby-veterinary is the clinic's always-open front door: where a pet owner finds help in an emergency, books care, learns from the clinic's own experts, and can buy from the clinic with safety built into every transaction.

## 2. The Problem

- **Emergency contact is hard to find.** Owners in a panic have to hunt for a phone number, an address, and opening hours, and that search costs them the minutes that matter most.
- **Access to care is phone-only.** Intake, record transfer, and consent paperwork are slow and manual, so the front desk absorbs work the site should carry.
- **Repeat demand has no channel.** Refills, preventatives, and therapeutic diets can only be had by making a call or a trip, so recurring revenue leaks away.
- **The clinic has no always-on presence.** At night, on weekends, or in a phone-line spike, owners who need triage have nowhere to turn.
- **Staff lose hours to repetitive triage.** Common questions consume receptionist time that should go to the animals on the schedule.

## 3. Who We Serve

- **Pet owners in an emergency** — usually on a phone, stressed, needing a number to call and directions immediately.
- **New and routine-care clients** — people registering, booking, or sending records ahead of a first or annual visit.
- **Returning clients** — owners buying preventatives, refilling prescriptions, or subscribing to monthly wellness products.
- **Clinic staff** — receptionists, veterinarians, and technicians who run the front desk, publish content, authorize prescriptions, and answer conversations.

Detailed personas live in `user-personas.md`.

## 4. Vision Statement

> Every owner in the clinic's care has one trusted place to turn — at any hour, from any device — to reach a veterinarian, understand their options, and get what their animal needs, without having to negotiate a system that was not designed for them.

## 5. Strategic Pillars

- **Emergency clarity in three seconds.** Any visitor can identify where the clinic is, when it is open, and how to reach it within three seconds of landing.
- **Frictionless access to care.** Booking, registration, record upload, and care documents happen online, so a visit starts without paperwork.
- **Trusted expertise on demand.** The clinic publishes its own veterinary guidance, authored by named clinicians and structured for search, so owners get informed answers instead of guesswork.
- **Always-on conversational triage.** Common questions are answered instantly, emergencies are detected and escalated, and a human is always one request away.
- **Safe, guardrailed commerce.** Products are easy to buy, but prescription items move only after a veterinarian authorizes them, and payments are handled by a tokenized gateway we never touch.
- **Operable without technical staff.** Staff publish articles, change prices, approve prescriptions, and answer messages through visual tools rather than a developer.

## 6. What Success Looks Like

Success is measured against the commitments already made in `non-functional-requirements.md`:

- A visitor finds the emergency phone number, address, and hours within three seconds, on a phone, on a slow connection.
- The homepage and emergency pages load in under 3 seconds on 4G; the WhatsApp bot answers in under 2 seconds.
- The site holds 99.9% uptime and is reachable to WCAG 2.1 AA.
- Every submission — appointment request, new-client form, WhatsApp handover — reaches a person, and no integration failure ever leaves an owner without a phone number to call.
- No prescription-only product reaches a customer without veterinary authorization.
- Staff ship content and pricing changes without a developer, and the catalog scales to hundreds of SKUs without slowing the storefront down.

## 7. Guiding Principles

1. **Emergency before commerce.** When an emergency and a sale compete for the same screen, the emergency wins.
2. **Safety rails over conversion.** A blocked checkout is a better outcome than a prescription dispensed to the wrong patient.
3. **Verified channels, privacy by default.** Medical conversations run through official business channels; client and pet data is protected by default.
4. **Graceful degradation, never a dead end.** When an integration fails we degrade to plain instructions and a phone number, never a blank screen.
5. **Mobile-first, accessible to everyone.** Our most urgent traffic arrives on a phone, in a hurry, possibly one-handed.

## 8. Scope

**In scope:** the public clinic site and emergency information, service and staff pages, appointment requests and new-client intake, downloadable care documents, the blog CMS, the WhatsApp triage bot with human handover, the product storefront with prescription authorization, and the staff back office.

**Deferred:** online video consultations, pet insurance, multi-clinic or chain support, native mobile apps, and in-house payment processing.

## 9. Related Documents

- `functional-requirements.md` — what the system must do
- `non-functional-requirements.md` — the quality bar it must meet
- `prd.md` — product requirements
- `user-personas.md` — detailed personas
