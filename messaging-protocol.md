# Messaging Protocol

> **ruby-veterinary Documentation**
>
> **Document:** Messaging Protocol — WebSocket + Auth
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** API Protocol

---

# Purpose

This document defines the real-time messaging contract between
`ruby-veterinary-web-frontend` and `ruby-veterinary-api` over Socket.IO
WebSocket transport, the session authentication model (web + mobile), the
frame protocol, reconnection replay semantics, and error codes. REST
resources are defined in `openapi/messaging.yaml` and described in
`api-specification.md` §Care Messaging.

---

# Transport

- **Library:** `socket.io` (server) / `socket.io-client` (browser + mobile)
- **Namespace:** `/chat`
- **Path:** `/api/v1/chat/socket.io`
- **Transport:** `websocket` (no long-polling fallback)
- **Protocol:** Socket.IO v4 with default `msgpack` parser

---

# Authentication

## Web (browser widget)

1. Client calls `POST /api/v1/chat/session` (anonymous, `Origin` header validated).
2. Server returns `Set-Cookie: rv_chat_session=<opaque token>; HttpOnly; Secure; SameSite=Lax; Path=/api` and body `{ "sessionId": "...", "seq": 0 }`.
3. Client connects to `/chat` with `withCredentials: true`. The HttpOnly cookie is sent automatically on the WebSocket upgrade request.
4. Server resolves `ChatSessionContext` from the cookie in `handleConnection`.
5. On disconnect, the cookie persists; reconnection carries the same session.

## Mobile (native app)

1. Client calls `POST /api/v1/chat/session/mobile` (anonymous, `Origin` header validated).
2. Server returns `{ "sessionId": "...", "sessionToken": "...", "accessToken": "...", "refreshToken": "..." }`.
   - `accessToken`: short-lived JWT (30 min), `sub` claim = `sessionId`.
   - `refreshToken`: opaque, single-use, rotated on each refresh call.
3. Client connects to `/chat` with `auth: { token: "<accessToken>" }`.
4. Server extracts `sessionId` from the JWT in `handleConnection`.
5. When the access token nears expiry, client calls `POST /api/v1/chat/token/refresh` with `Authorization: Bearer <refreshToken>` to get a new `accessToken`. The old refresh token is invalidated (rotation).
6. On reconnect, client refreshes the token if expired and reconnects.

## ChatSessionGuard

One guard (`ChatSessionGuard`) handles both flows:
- Incoming HTTP request with `rv_chat_session` cookie → resolve session by hashed token.
- Incoming WS handshake with `auth.token` → decode JWT, verify `sub`, resolve session.
- Missing or expired → `401 UNAUTHORIZED` (HTTP) or `error` frame + disconnect (WS).

---

# Session Lifecycle

```
anonymous POST /session ───► active (AI-only, 45 min TTL)
                                   │
                          visitor requests handoff
                                   │
                                   ▼
                              handoff (24 h TTL)
                                   │
                          staff accepts / closes
                                   │
                                   ▼
                              expired / closed
```

| State | TTL | Reconnect behaviour |
|---|---|---|
| `active` (AI-only) | 45 min after `last_activity_at` | Replay AI messages; if expiry has passed → `expired` frame + disconnect |
| `handoff` | 24 h after `last_activity_at` | Replay all messages (AI + staff); if expiry has passed → `expired` frame + disconnect |
| `expired` | — | Server emits `expired` frame with clinic contact info and disconnects |
| `closed` | — | No replay possible; session is terminal |

The `ChatSessionSweeper` runs every 5 minutes via `@Interval(5000 * 60)`: deletes stale AI-only chats (`status = 'active' AND expires_at < now()`), marks handoff as expired (`status = 'handoff' AND expires_at < now()`). Expired sessions cannot be reconnected.

---

# Frame Protocol

Every frame the server emits carries a per-session monotonic `seq` (bigint).
The client uses `seq` to sort, deduplicate, and track the replay watermark
(`last_seen_seq`).

## Frame types

### `token` — streaming text chunk

```json
{
  "seq": 42,
  "type": "token",
  "role": "assistant|user",
  "text": "Our clinic operates from",
  "final": false
}
```

A `token` frame with `final: true` is the last chunk of a message. The client
appends `text` to the in-progress message; on `final: true`, the message is
complete. User messages are echoed as a single `token` frame with
`final: true`.

### `citation` — source attribution

```json
{
  "seq": 42,
  "type": "citation",
  "sources": [
    {
      "title": "Company Policies",
      "category": "policy",
      "effectiveDate": "2026-10-01"
    }
  ]
}
```

Emitted after the assistant's `token` stream and before `done`. Replay does not
re-emit `citation` frames — citations are reconstructed from the stored
`inbox_messages.citations` JSONB column.

### `done` — message complete

```json
{
  "seq": 42,
  "type": "done",
  "messageSeq": 42
}
```

Signals the end of a complete message (all `token` + optional `citation`
frames). `messageSeq` matches the `seq` of the message's first `token` frame.

### `error` — non-fatal error on last message

```json
{
  "seq": 43,
  "type": "error",
  "code": "RATE_LIMITED",
  "retryAfterMs": 5000
}
```

The session remains open. The client may retry after waiting.

### `handoff` — staff handover state change

```json
{ "seq": 44, "type": "handoff", "state": "requested" }
{ "seq": 45, "type": "handoff", "state": "accepted", "staffName": "Dr. Ada" }
{ "seq": 46, "type": "handoff", "state": "closed" }
```

### `staff_message` — staff reply bridged from Telegram

```json
{
  "seq": 47,
  "type": "staff_message",
  "text": "Hi! I'm looking at your question about ...",
  "senderName": "Dr. Ada"
}
```

### `expired` — session terminated

```json
{
  "seq": 48,
  "type": "expired",
  "contact": {
    "phone": "+2348139640640",
    "formUrl": "https://ruby-veterinary.com",
    "bookingUrl": "https://calendly.com/ezeonyekachukwu97/30min"
  }
}
```

Emitted on connect/reconnect after session TTL has elapsed, or when the
sweeper terminates a session. After this frame, the server disconnects.

### `notice` — system notice (new in M6a RSV)

```json
{
  "seq": 1,
  "type": "notice",
  "text": "Please don't share personal or pet-specific details in this chat. If your pet is having an emergency, call the clinic now."
}
```

Emitted once per session, immediately after connection, as `seq: 1`.

---

# Message Flow (inbound)

```
Client:               Server:
emit('msg', { text }) ──►
                        ├─ Rate check (IP + session)
                        ├─ Echo user message → token frame (final: true)
                        ├─ Record in inbox_messages
                        ├─ Classify: emergency? private-data? normal?
                        ├─ if emergency: fixed safe response (no LLM)
                        ├─ if private-data: handoff reply (no retrieval)
                        ├─ if normal: RAG retrieve → prompt → LLM stream
                        │   ├─ token frames (streaming)
                        │   ├─ citation frame
                        │   └─ done frame
                        └─ Update usage_daily
```

The client emits `msg` events; the server emits `token`/`citation`/`done`/`error` frames.

---

# Reconnection + Replay

## Client reconnect flow

```ts
// On reconnect_attempt, update the auth payload
socket.io.on('reconnect_attempt', () => {
  socket.auth = { last_seen_seq: state.lastSeenSeq };
});

// On reconnect, deduplicate frames already received
socket.on('token', (frame) => {
  if (frame.seq <= state.lastSeenSeq) return; // already seen
  dispatch(frame);
});
```

## Server replay

On connection, the `auth.last_seen_seq` value (or `0` for new sessions) triggers
a replay query:

```sql
SELECT * FROM messaging.inbox_messages
WHERE session_id = $1
  AND seq > $2           -- last_seen_seq
  AND created_at > $3    -- now() - session TTL
ORDER BY seq
```

The server emits one `token` frame (with `final: true`, full stored text, not streaming deltas)
per missed message, then a `done` frame for each. On replay, `citation` frames are not
re-emitted — the stored `citations` JSONB column is reconstructed on the client from the
message metadata.

If the session has expired (TTL elapsed), the server emits `expired` and disconnects
instead of replay.

---

# Heartbeat

Socket.IO ping/pong is configured at the library level:

```ts
// Server
const io = new Server(httpServer, {
  pingInterval: 25_000,
  pingTimeout: 60_000,
  // ...
});
```

No application-layer heartbeat is needed. Socket.IO handles dead connections.
On server-side timeout, `handleDisconnect` is called and `last_activity_at` is
updated (no immediate sweep — sweeper handles expiry).

---

# Client-Side Visibility / Online

The frontend widget listens to `visibilitychange` and `online`/`offline` events:

```ts
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'visible') reconnect();
});

window.addEventListener('online', () => reconnect());
window.addEventListener('offline', () => { /* show offline indicator */ });
```

This is a client implementation detail, not part of the wire protocol.

---

# Error Codes

Stable string codes; clients branch on `code`, never on `message`.

| Code | Meaning | Client action |
|---|---|---|
| `RATE_LIMITED` | Per-IP or per-session rate exceeded | Wait `retryAfterMs`, back off |
| `CAP_REACHED` | Daily request or token cap exhausted | Show clinic contact; no more messages accepted for the day |
| `UNAUTHORIZED` | Session invalid or expired | Recreate session (web) or refresh token (mobile) |
| `MESSAGE_TOO_LONG` | Input exceeds `CHAT_MAX_MESSAGE_CHARS` | Trim and resend |
| `SESSION_FULL` | Per-session message cap reached | Handoff suggestion |
| `TURNSTILE_REQUIRED` | Suspicious activity detected | Require Turnstile solve before next session create |
| `PROVIDER_ERROR` | Telegram API error (M6b) | Offline indicator; retry after backoff |
| `INTERNAL` | Unexpected server error | Retry once, then show clinic contact |

---

# Rate Limits on WS

| Limit | Scope | Enforced |
|---|---|---|
| Per-IP messages | `CHAT_RATE_PER_IP_PER_MIN` / 60 000 ms token bucket | On every `msg` emit |
| Per-session messages | `CHAT_RATE_PER_SESSION_PER_MIN` / 60 000 ms | On every `msg` emit |
| Max messages per session | `CHAT_MAX_MESSAGES_PER_SESSION` (60) | Counter in session |
| Max message length | `CHAT_MAX_MESSAGE_CHARS` (2000) | Before classification |
| Daily request cap | `CHAT_DAILY_REQUEST_CAP` (global) | `usage_daily` upsert |
| Daily token cap | `CHAT_DAILY_TOKEN_CAP` (global) | `usage_daily.completion_tokens` |

Rate limits are in-memory token buckets (per-IP, per-session) and DB-backed
(daily cap). On deploy restart, in-memory buckets reset — daily cap survives.

---

# Seq Allocation

`seq` is allocated atomically inside the message insert transaction:

```sql
-- Within the same transaction:
UPDATE messaging.chat_sessions
SET last_seq = last_seq + 1
WHERE id = $session_id
RETURNING last_seq;

INSERT INTO messaging.inbox_messages (session_id, seq, ...)
VALUES ($session_id, $returned_seq, ...);
```

The `UPDATE` row lock guarantees total ordering under concurrent access.
Skipped seq values are impossible — the `UPDATE` happens before `INSERT`
in the same locked transaction.

---

# Transport Restrictions

- No polling fallback (`transports: ['websocket']` only).
- No binary frames (all JSON).
- No room switching for a session (one session = one room).
- No cross-session messaging (a session's frames are private to that session).
- No reconnection after `expired` frame (socket disconnects permanently).

---

# Related Documents

| Document | Relationship |
|---|---|
| `openapi/messaging.yaml` | REST endpoints (session, mobile, refresh, admin AI, webhook) |
| `api-specification.md` | §Care Messaging — REST contract |
| `data-model/care-messaging.md` | DB tables behind sessions + messages |
| `data-model/ai-assistant.md` | AI KB schema + retrieval |
| `architectural-decision-record.md` | ADR-0017 (hybrid messaging + AI) |

---

# Guiding Principle

> **Frames are facts. Every frame is committed to the message table before it
> hits the wire, and every frame has a `seq` that proves it.**