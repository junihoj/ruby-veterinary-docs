# Care Messaging Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Care Messaging Data Model
>
> **Version:** 2.0.0
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

Defines the tables behind the multi-channel messaging front door: web chat widget, Telegram handoff, and (future) WhatsApp. Covers anonymous web sessions, AI bot routing with emergency/private-data classification, per-session sequentially ordered transcripts, handoff provider bridging via provider-swappable adapters, rate limits, and daily AI spend caps.

---

# Responsibilities

The Care Messaging context owns:

- Web chat sessions (anonymous and authenticated) with expiry and handoff lifecycle
- AI bot routing: deterministic emergency and private-data classification before LLM calls, RAG retrieval from the `ai_kb` schema, and capped daily usage
- Per-session seq-ordered transcript (inbox messages with monotonic sequence numbers)
- Handoff provider bridging: Telegram forum topics (and future WhatsApp) through a provider-swappable adapter
- Bot rules for keyword/panic/private-data classification (repurposed — no more numbered menu flows)
- Rate limits and daily AI spend caps enforced at the API boundary
- Provider thread tracking (one thread per conversation per provider)

It does **not** own appointment records (raises events to Care Coordination on handover intent), client pet data, or the AI knowledge base (that lives in `ai_kb`). Message bodies may contain clinical context; retention and access follow clinic policy and the audit requirements in Notifications/Operations.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Conversation | `conversations` | Messages, chat sessions, handover state, and provider threads move together |

---

# Entities

## messaging.conversations

Generalized from the original WhatsApp-only model. One row per contact identity per channel. Web chat sessions, Telegram users, and future WhatsApp contacts each get a conversation row keyed by `(channel, contact_identity)`.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `cnv_…` |
| channel | text | no | `web_chat` \| `telegram` \| `whatsapp` |
| contact_identity | text | no | Session id (web), Telegram user id, or phone (WhatsApp); together with `channel` forms the logical unique key |
| wa_phone_number | text | yes | **Legacy.** WhatsApp Business API number (E.164); nullable for non-WhatsApp channels |
| contact_phone | text | yes | **Legacy.** Owner's WhatsApp number (E.164); nullable for non-WhatsApp channels |
| contact_name | text | yes | Push name, Telegram display name, or web-provided name |
| client_id | uuid | yes | Cross-schema ref by id → `clients.id` when correlated |
| user_id | uuid | yes | Cross-schema ref by id → `identity.users.id` when signed-in correlation exists |
| direction | text | no | `inbound` \| `outbound` (thread direction semantics) |
| status | text | no | `bot_active` \| `awaiting_handover` \| `human_active` \| `closed` \| `blocked` |
| emergency_flag | boolean | no | DEFAULT false; true when panic keywords detected |
| last_inbound_at | timestamptz | no | |
| last_outbound_at | timestamptz | yes | |
| created_at | timestamptz | no | DEFAULT `now()` |
| updated_at | timestamptz | no | |

**Indexes**

- `ix_conversations_channel` on `(channel, status, last_inbound_at DESC)`
- `ix_conversations_emergency` on `(emergency_flag, status)`

**Invariants**

- Panic detection sets `emergency_flag` and priority queue ordering immediately.
- A conversation in `human_active` never auto-resumes bot replies until handover closes.
- `wa_phone_number` and `contact_phone` are nullable legacy columns retained for backward compatibility; new web chat and Telegram conversations leave them null.

## messaging.chat_sessions

A web or mobile chat session tied to a conversation. Carries an opaque token (hashed for storage), a client type, a monotonic sequence counter, and expiry metadata. Sessions are the unit of authentication for WebSocket connections.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| token_hash | text | no | SHA-256 of the opaque session token; UNIQUE |
| client_type | text | no | `web` \| `mobile` |
| conversation_id | uuid | no | FK → `conversations.id` |
| last_seq | bigint | no | DEFAULT 0; monotonically increasing message sequence number |
| status | text | no | `active` \| `handoff` \| `expired` \| `closed`; DEFAULT `active` |
| last_activity_at | timestamptz | no | DEFAULT `now()` |
| handoff_started_at | timestamptz | yes | Set when a handoff is requested from this session |
| expires_at | timestamptz | no | Computed from `last_activity_at` + TTL based on status |
| created_at | timestamptz | no | DEFAULT `now()` |

**Indexes**

- `ix_chat_sessions_token` unique on `(token_hash)`
- `ix_chat_sessions_expiry` on `(expires_at)` WHERE `status != 'closed'`

**Invariants**

- `seq` is allocated via `UPDATE ... SET last_seq = last_seq + 1 RETURNING last_seq` inside the message insert transaction; no gaps, no race conditions.
- AI-only sessions (status `active`, no handoff) expire `CHAT_SESSION_TTL_MINUTES` after `last_activity_at`.
- Handoff sessions expire `CHAT_HANDOFF_RETENTION_HOURS` after `last_activity_at`.
- Expired sessions emit an `expired` frame on reconnect and are swept by the `ChatSessionSweeper`.
- Reconnecting after TTL returns an `expired` frame with clinic contact info; the session is not revived.

## messaging.inbox_messages

Extended from the original WhatsApp-only model. Now includes session-scoped sequence numbers, frame type classification, and AI citations. One unified stream for bot, staff, and system messages so handover is chronologically coherent.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `msg_…` |
| conversation_id | uuid | no | FK → `conversations.id` |
| session_id | uuid | yes | FK → `chat_sessions.id`; null for provider-native messages (Telegram/WhatsApp with no web session) |
| seq | bigint | yes | Per-session monotonic sequence number; null when `session_id` is null |
| frame_type | text | yes | `token` \| `staff_message` \| `handoff` \| `system`; null for legacy rows |
| direction | text | no | `inbound` \| `outbound` |
| sender_type | text | no | `owner` \| `bot` \| `staff` |
| sender_user_id | uuid | yes | Cross-schema ref by id for staff sends |
| body_text | text | no | Plain text (rich media referenced separately) |
| media_object_key | text | yes | SeaweedFS key for media; never inline bytes |
| media_mime | text | yes | |
| citations | jsonb | yes | AI-generated citations: `[{chunk_id, document_title, excerpt}]` |
| client_type | text | yes | `web` \| `mobile` \| null for provider-native messages |
| status | text | no | `received` \| `sent` \| `delivered` \| `read` \| `failed` |
| dedupe_key | text | no | Provider message id or client-generated idempotency key; unique |
| request_id | text | yes | Correlates message to the originating HTTP/WS request |
| created_at | timestamptz | no | DEFAULT `now()` |

**Indexes**

- `uq_inbox_messages_dedupe` unique on `(dedupe_key)`
- `uq_inbox_messages_session_seq` unique on `(session_id, seq)` WHERE `session_id IS NOT NULL AND seq IS NOT NULL`
- `ix_inbox_messages_conversation` on `(conversation_id, created_at)`

**Invariants**

- `seq` is per-session monotonic; allocated atomically with the session's `last_seq`.
- Replay queries: `WHERE session_id = $1 AND seq > $last_seen_seq AND created_at > now() - interval $ttl`.
- Expired session messages are deleted by the `ChatSessionSweeper` along with the session.
- Provider retries dedupe on `dedupe_key`; duplicates never double-post.
- Bot and staff messages share one stream so handover is chronological.

## messaging.provider_threads

Maps a conversation to an external provider thread (e.g., Telegram forum topic). One thread per conversation per provider. Created when a handoff is requested.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| conversation_id | uuid | no | FK → `conversations.id` |
| provider | text | no | `telegram` \| `whatsapp` |
| provider_thread_id | text | no | Telegram forum topic id or WhatsApp conversation id |
| metadata | jsonb | yes | Provider-specific state (Telegram: chat_id, message_thread_id, forum_topic_created_at) |
| created_at | timestamptz | no | DEFAULT `now()` |

**Indexes**

- `uq_provider_threads_conv_provider` unique on `(conversation_id, provider)`
- `uq_provider_threads_provider_id` unique on `(provider, provider_thread_id)`

**Invariants**

- One thread per conversation per provider.
- Created on handoff request; the `MessagingProviderPort` adapter creates the actual provider resource and stores the thread id here.
- `metadata` carries Telegram-specific state (chat_id, message_thread_id) so the adapter does not need to poll Telegram for context.

## messaging.chat_usage_daily

Tracks AI usage per calendar day for rate limiting and cost control. Atomic upsert on every AI request.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| day | date | no | PK; UTC calendar day |
| requests | integer | no | DEFAULT 0 |
| prompt_tokens | integer | no | DEFAULT 0 |
| completion_tokens | integer | no | DEFAULT 0 |
| cost_usd | numeric(10,6) | no | DEFAULT 0; cumulative cost for the day |

**Indexes**

- PK on `(day)` — single row per day

**Invariants**

- Atomic upsert on every AI request:
  ```sql
  INSERT INTO messaging.chat_usage_daily (day, requests, prompt_tokens, completion_tokens, cost_usd)
  VALUES ($day, 1, $prompt_tokens, $completion_tokens, $cost_usd)
  ON CONFLICT (day) DO UPDATE SET
    requests = chat_usage_daily.requests + 1,
    prompt_tokens = chat_usage_daily.prompt_tokens + $prompt_tokens,
    completion_tokens = chat_usage_daily.completion_tokens + $completion_tokens,
    cost_usd = chat_usage_daily.cost_usd + $cost_usd;
  ```
- When `requests >= CHAT_DAILY_REQUEST_CAP` or `completion_tokens >= CHAT_DAILY_TOKEN_CAP`, the API returns a `CAP_REACHED` error and no further AI calls are made for the day.
- Cap values are environment-configured (`CHAT_DAILY_REQUEST_CAP`, `CHAT_DAILY_TOKEN_CAP`).

## messaging.handover_sessions

Extended from the original model to include the provider and provider thread. Tracks the lifecycle of a handoff from request through claim to resolution.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `hnd_…` |
| conversation_id | uuid | no | FK → `conversations.id` |
| provider | text | no | `telegram` \| `whatsapp`; the provider handling this handoff |
| provider_thread_id | text | yes | FK by value → `provider_threads.provider_thread_id` where provider matches |
| requested_reason | text | no | `owner_request` \| `panic` \| `bot_unresolved` \| `menu_complete` |
| requested_at | timestamptz | no | |
| claimed_by | uuid | yes | Cross-schema ref by id → staff user (inbox claim) |
| claimed_at | timestamptz | yes | |
| status | text | no | `requested` \| `claimed` \| `released` \| `resolved` |
| resolved_at | timestamptz | yes | |
| closed_at | timestamptz | yes | |
| closed_reason | text | yes | |
| appointment_request_id | uuid | yes | Cross-schema ref by id when handover produced a booking intent |
| created_at | timestamptz | no | DEFAULT `now()` |

**Indexes**

- `ix_handover_open` on `(status, requested_at)` WHERE `status IN ('requested', 'claimed')`

**Invariants**

- One active handover per conversation at a time.
- Concurrent claim attempts: first claim wins (`claimed_by` set atomically via `UPDATE ... WHERE claimed_by IS NULL`).
- Bot pauses for the conversation while a handover session is open.
- `provider` and `provider_thread_id` are set when the handover is created; the `MessagingProviderPort` adapter creates the external resource.

## messaging.bot_rules

Repurposed from the original numbered-menu bot. Now used exclusively for deterministic emergency and private-data keyword classifiers. The `menu_type` column is unused by the AI bot.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | e.g. `Panic phrases`, `PII email pattern`, `Hours keyword` |
| kind | text | no | `keyword` \| `panic` \| `fallback`; `menu_entry` is deprecated |
| keywords | text[] | no | Lowercased match set |
| priority | smallint | no | Higher wins |
| response_template | text | yes | Deterministic fallback reply; must include clinic phone fallback line |
| action | text | no | `reply` \| `escalate`; `menu_start` is deprecated |
| active | boolean | no | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Invariants**

- `panic` rules always outrank ordinary keywords and raise `EmergencyDetected`.
- Emergency and PII-classified messages never reach the LLM; they receive a deterministic template response.
- Every template carries plain-language fallback (phone number) even when the bot reply succeeds.

---

# Deprecated Tables

## messaging.menu_flows

**Deprecated.** The AI bot does not use numbered menus. Table kept for backward compatibility with existing data; no new rows are created. All new bot interactions use natural-language AI responses or deterministic keyword templates from `bot_rules`.

## messaging.conversation_menu_state

**Deprecated.** Session-scoped menu state is replaced by the AI bot's conversational context and the `chat_sessions` model. Table kept for backward compatibility; no new rows are created.

## messaging.messaging_business_profiles

**Legacy.** WhatsApp-specific business profile data (phone numbers, timezone, away messages, hours). Not used by web chat or Telegram channels. Retained for backward compatibility with any WhatsApp data that may exist. New channel configurations live in environment variables and the `provider_threads` table.

---

# Chat Session Lifecycle

```
                   ┌─────────────────┐
                   │  Anonymous web  │
                   │  visitor opens  │
                   │  chat widget    │
                   └───────┬─────────┘
                           │
                   ┌───────▼─────────┐
                   │  Session created│
                   │  status: active │
                   │  last_seq: 0    │
                   └───────┬─────────┘
                           │
                   ┌───────▼─────────┐
                   │  bot_active     │
                   │  (AI responds)  │
                   └───┬─────────┬───┘
                       │         │
              visitor requests  panic/emergency
              handoff           detected
                       │         │
               ┌───────▼──┐  ┌──▼───────────┐
               │ handoff  │  │ emergency_flag│
               │ requested│  │ set; handoff  │
               │ status:  │  │ auto-created  │
               │ handoff  │  └───────────────┘
               └───────┬──┘
                       │
               ┌───────▼──────────┐
               │ staff claims     │
               │ handover session │
               │ status: claimed  │
               └───────┬──────────┘
                       │
               ┌───────▼──────────┐
               │ human_active     │
               │ (conversation)   │
               └───────┬──────────┘
                       │
              ┌────────┼────────┐
              │                 │
      staff resolves     visitor disconnects
      handover           or session expires
              │                 │
      ┌───────▼──┐      ┌──────▼──────────┐
      │ closed   │      │ expired         │
      │ (conv)   │      │ (session)       │
      └──────────┘      └──────┬──────────┘
                               │
                     reconnect after TTL
                               │
                      ┌────────▼──────────┐
                      │ expired frame     │
                      │ + clinic contact  │
                      │ session NOT       │
                      │ revived           │
                      └───────────────────┘
```

- **Sweeper interval**: every 5 minutes (`ChatSessionSweeper` cron tick).
- **AI-only sessions**: expire `CHAT_SESSION_TTL_MINUTES` after `last_activity_at` (default: 30 min).
- **Handoff sessions**: expire `CHAT_HANDOFF_RETENTION_HOURS` after `last_activity_at` (default: 72 hours).
- **Reconnect after TTL**: the server sends an `expired` WebSocket frame with clinic phone number and hours; the session is not revived. The client must start a new session.
- **Expired session cleanup**: the sweeper deletes `chat_sessions` rows where `expires_at < now()` and cascades to associated `inbox_messages` rows (via application-level cleanup, as FK is nullable).

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → AI Assistant | AI bot retrieves from `ai_kb` when a message passes emergency/PII classification; the Messaging context calls the `ai-assistant` module via its public facade |
| → Notifications | `EmergencyDetected`, `HandoverRequested` raise alerts |
| → Care Coordination | handover may create intent → appointment request via events |
| → Client & Patient | optional client correlation by phone or signed-in user id |
| → Operations | inbox UI reads `conversations` / `inbox_messages` / `handover_sessions` via public module |
| → Identity | `user_id` cross-reference for authenticated sessions; `ChatSessionGuard` validates both cookie tokens and JWT bearers |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `ai-assistant.md` | AI knowledge base schema invoked by the bot |
| `../domain-model.md` | Care Messaging domain |
| `../domain-events.md` | `EmergencyDetected`, `HandoverRequested/Accepted`, `ChatSessionExpired` |
| `../api-specification.md` | WebSocket handshake, chat REST endpoints, admin inbox APIs |
| `../architectural-decision-record.md` | ADR-0017 hybrid messaging + AI assistant decision; ADR-0008 amended |
| `../prd.md` | M6 messaging requirements |

---

# Acceptance Criteria

- Web chat widget connects via Socket.IO, receives a session token in the handshake response, and resumes with `last_seq` on reconnect within TTL
- AI bot responds within the latency budget; emergency/PII-classified messages never reach the LLM
- Provider webhook retries never duplicate messages (dedupe on `dedupe_key` across all channels)
- Panic keywords always set `emergency_flag` within the 2s bot latency budget
- Shared inbox supports concurrent agents with exclusive claim semantics
- Daily AI usage cap is enforced atomically; `CAP_REACHED` response is returned when cap is hit
- Expired sessions emit `expired` frame on reconnect and are swept within 5 minutes of their expiry time
- Every bot path offers a phone fallback in the reply text

---

# Guiding Principle

> **The bot exists to get a frightened owner to a human faster. Any automation that delays that path has failed, no matter how clever it is.**