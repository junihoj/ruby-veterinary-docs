# Care Messaging Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Care Messaging Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Care Messaging Bounded Context
>
> **Classification:** Core Domain
>
> **Schema:** `messaging`

---

# Purpose

Defines the tables behind the WhatsApp front door: conversations through the verified Business API, keyword auto-responder rules, panic detection, numbered pre-screening menus, bot-to-human handover, out-of-hours schedules, and the shared multi-agent inbox.

---

# Responsibilities

The Care Messaging context owns:

- WhatsApp conversations and inbound message log
- Bot rules (keywords, panic phrases, auto-responses)
- Menu flows and in-conversation menu state
- Handover sessions and claim state for the shared inbox
- Messaging-side business-hours profiles and away messages
- Inbox message rows used by the staff back office

It does **not** own appointment records (raises events to Care Coordination on handover intent) or client pet data. Message bodies may contain clinical context; retention and access follow clinic policy and the audit requirements in Notifications/Operations.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Conversation | `conversations` | Messages, menu state, and handover state move together |

---

# Entities

## conversations

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `cnv_…` |
| wa_phone_number | text | no | Clinic Business API number (E.164) |
| contact_phone | text | no | Owner's WhatsApp number (E.164) |
| contact_name | text | yes | Push name when provided |
| client_id | uuid | yes | Cross-schema ref by id → `clients.id` when correlated |
| user_id | uuid | yes | Cross-schema ref by id → `identity.users.id` when signed-in correlation exists |
| direction | text | no | `inbound` \| `outbound` (thread direction semantics) |
| status | text | no | `bot_active` \| `awaiting_handover` \| `human_active` \| `closed` \| `blocked` |
| emergency_flag | boolean | no | True when panic keywords detected |
| last_inbound_at | timestamptz | no | |
| last_outbound_at | timestamptz | yes | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**

- `ix_conversations_contact` on `(contact_phone, last_inbound_at desc)`
- `ix_conversations_status` on `(status, last_inbound_at)`
- `ix_conversations_emergency` on `(emergency_flag, status)`

**Invariants**

- Panic detection sets `emergency_flag` and priority queue ordering immediately
- A conversation in `human_active` never auto-resumes bot replies until handover closes

## inbox_messages

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `msg_…` |
| conversation_id | uuid | no | FK → `conversations.id` |
| provider_message_id | text | no | WhatsApp message id; unique for dedup |
| direction | text | no | `inbound` \| `outbound` |
| sender_type | text | no | `owner` \| `bot` \| `staff` |
| sender_user_id | uuid | yes | Cross-schema ref by id for staff sends |
| body_text | text | no | Plain text (rich media referenced separately) |
| media_object_key | text | yes | MinIO key for media; never inline bytes |
| media_mime | text | yes | |
| status | text | no | `received` \| `sent` \| `delivered` \| `read` \| `failed` |
| dedupe_key | text | no | Provider message id; unique |
| created_at | timestamptz | no | |

**Indexes**

- `uq_inbox_messages_dedupe` unique on `(dedupe_key)`
- `ix_inbox_messages_conversation` on `(conversation_id, created_at)`

**Invariants**

- Provider retries dedupe on `dedupe_key`; duplicates never double-post
- Bot and staff messages share one stream so handover is chronological

## bot_rules

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | e.g. `Hours keyword`, `Panic phrases` |
| kind | text | no | `keyword` \| `panic` \| `menu_entry` \| `fallback` |
| keywords | text[] | no | Lowercased match set |
| priority | smallint | no | Higher wins |
| response_template | text | yes | Bot reply; must include clinic phone fallback line |
| action | text | no | `reply` \| `escalate` \| `menu_start` |
| active | boolean | no | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Invariants**

- `panic` rules always outrank ordinary keywords and raise `EmergencyDetected`
- Every template carries plain-language fallback (phone number) even when bot reply succeeds

## menu_flows

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | e.g. `New client pre-screen` |
| version | integer | no | Flows versioned; replies reference version |
| definition_json | jsonb | no | Numbered options tree |
| active | boolean | no | |
| created_at | timestamptz | no | |

## conversation_menu_state

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| conversation_id | uuid | no | FK → `conversations.id` |
| menu_flow_id | uuid | no | FK → `menu_flows.id` |
| menu_version | integer | no | |
| current_step | text | no | Step key in `definition_json` |
| answers_json | jsonb | no | Collected answers |
| started_at | timestamptz | no | |
| completed_at | timestamptz | yes | |
| abandoned_at | timestamptz | yes | |

**Invariants**

- At most one active menu state per conversation

## handover_sessions

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `hnd_…` |
| conversation_id | uuid | no | FK → `conversations.id` |
| requested_reason | text | no | `owner_request`, `panic`, `bot_unresolved`, `menu_complete` |
| requested_at | timestamptz | no | |
| claimed_by | uuid | yes | Cross-schema ref by id → staff user (inbox claim) |
| claimed_at | timestamptz | yes | |
| status | text | no | `requested` \| `claimed` \| `released` \| `resolved` |
| resolved_at | timestamptz | yes | |
| appointment_request_id | uuid | yes | Cross-schema ref by id when handover produced a booking intent |
| created_at | timestamptz | no | |

**Indexes**

- `ix_handover_open` on `(status, requested_at)` where `status in ('requested','claimed')`

**Invariants**

- Concurrent claim attempts: first claim wins (`claimed_by` set atomically)
- Bot pauses for the conversation while a session is open

## messaging_business_profiles

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | e.g. `Main line` |
| wa_phone_number | text | no | Unique |
| timezone | text | no | IANA |
| away_message | text | no | Out-of-hours auto-response |
| hours_json | jsonb | no | Weekly schedule mirroring content hours (operational copy) |
| active | boolean | no | |
| created_at | timestamptz | no | |

**Invariants**

- Away messages always include the emergency phone number
- Hours changes require sync from Clinic Content or an explicit staff edit here (documented source of truth: Clinic Content; this table is the messaging runtime copy)

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → Notifications | `EmergencyDetected`, `HandoverRequested` raise alerts |
| → Care Coordination | handover may create intent → appointment request via events |
| → Client & Patient | optional client correlation by phone |
| → Operations | inbox UI reads `conversations`/`inbox_messages`/`handover_sessions` via public module |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Care Messaging domain |
| `../domain-events.md` | `EmergencyDetected`, `HandoverRequested/Accepted` |
| `../api-specification.md` | WhatsApp webhooks and admin inbox APIs |
| `../prd.md` | WhatsApp triage bot with human handover |

---

# Acceptance Criteria

- Provider webhook retries never duplicate messages
- Panic keywords always set emergency state within the 2s bot latency budget
- Shared inbox supports concurrent agents with exclusive claim semantics
- Every bot path offers a phone fallback in the reply text

---

# Guiding Principle

> **The bot exists to get a frightened owner to a human faster. Any automation that delays that path has failed, no matter how clever it is.**
