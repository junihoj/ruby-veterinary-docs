# Telegram Setup Runbook

> **ruby-veterinary Documentation**
>
> **Document:** Telegram Handoff — Setup Runbook
>
> **Version:** 1.0.0
>
> **Status:** Living
>
> **Owner:** ruby-veterinary
>
> **Classification:** Operations

---

# Purpose

This runbook guides a staff member (no developer access needed) through setting up Telegram as the clinic's staff handoff channel. The setup creates a Telegram bot, a private supergroup with forum topics enabled, and connects it to the ruby-veterinary API via webhook.

Estimated time: **15 minutes** with a Telegram account and admin access to the server's environment variables.

---

# Prerequisites

- A Telegram account (personal or clinic phone number)
- Admin access to set environment variables on the API server
- A publicly accessible HTTPS domain for the API (webhook delivery requires HTTPS)

---

# Step 1: Create the Telegram Bot

1. Open Telegram and search for `@BotFather` (official bot creation bot).
2. Send the command `/newbot`.
3. Follow the prompts:
   - **Bot name:** `Ruby Veterinary Handoff` (display name, can be changed later)
   - **Bot username:** `ruby_vet_handoff_bot` (must end in `bot`, must be unique globally)
4. BotFather returns a **token** — a long string like `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`.
   - **Copy this token now.** It is shown only once. If lost, revoke and regenerate with `/revoke` then `/newbot`.

5. (Optional) Set bot profile details via BotFather:
   - `/setdescription` — "Ruby Veterinary Clinic — staff handoff assistant"
   - `/setabouttext` — "Internal use only. Connects web chat visitors to our veterinary team."
   - `/setuserpic` — Upload the clinic logo

---

# Step 2: Create the Handoff Supergroup

1. In Telegram, tap the pencil icon → **New Group**.
2. **Group name:** `Ruby Vet Handoff`
3. **Group type:** Private (not public — client conversations are confidential).
4. Add at least one other staff member as an initial member (Telegram requires ≥ 2 members to create a group; you can remove them after the bot joins).
5. After creating the group:
   - Tap the group name → **Edit** (pencil icon) → **Group Type**.
   - Enable **Topics** (this turns the group into a forum).
   - Each conversation from a web chat visitor will create a new topic in this forum.

---

# Step 3: Add the Bot to the Group

1. Open the group → tap group name → **Add Members**.
2. Search for your bot username (e.g., `@ruby_vet_handoff_bot`).
3. Add the bot.
4. **Promote the bot to admin** (required to manage forum topics):
   - Group info → **Administrators** → **Add Admin** → select the bot.
   - Enable these permissions only:
     - ✅ Manage Topics
     - ✅ Send Messages
     - ❌ Everything else off (delete messages, ban users, etc.)
   - Tap **Save**.

---

# Step 4: Get the Group ID + Webhook Secret

1. **Group ID:**
   - Send a message in the group (any text).
   - Forward that message to `@getidsbot` (a Telegram bot that reveals IDs).
   - Note the **chat ID** — it starts with `-100` (e.g., `-1001234567890`).
   - Remove `@getidsbot` from the group after.

2. **Webhook secret:**
   - Generate a strong random secret (32+ characters):
     ```bash
     openssl rand -hex 32
     ```
   - Save this; it goes in `TELEGRAM_WEBHOOK_SECRET`.

---

# Step 5: Set Environment Variables

On the API server, set these three environment variables:

```env
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
TELEGRAM_FORUM_GROUP_ID=-1001234567890
TELEGRAM_WEBHOOK_SECRET=<the output from openssl rand -hex 32>
```

Restart the API service after setting them. The API registers the webhook with
Telegram on startup (POST `https://api.telegram.org/bot<TOKEN>/setWebhook` with
`url=https://<your-domain>/api/v1/webhooks/telegram` and
`secret_token=<TELEGRAM_WEBHOOK_SECRET>`).

**Do not prefix any of these variables with `NEXT_PUBLIC_`.** These are
server-side secrets and must never be exposed to the browser.

---

# Step 6: Verify the Webhook

1. After the API starts, check the webhook status:
   ```bash
   curl https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/getWebhookInfo
   ```
2. Verify:
   - `"url"` is your API domain + `/api/v1/webhooks/telegram`
   - `"has_custom_certificate"` is `false` (you're using a standard HTTPS cert)
   - `"pending_update_count"` starts at `0`
   - No `"last_error_message"`

3. Send a test message to the bot (DM the bot, not the group):
   - The API logs should show a received update.
   - No forum topic should be created (DMs are not handoff conversations).

---

# Step 7: Staff Onboarding

1. Add all receptionists and veterinarians who handle chats to the `Ruby Vet Handoff` group.
2. Each staff member should:
   - Mute general notifications for the group (topics appear separately).
   - **Enable topic notifications** — new topics from web visitors appear as new items.
3. When a customer requests handoff in the web chat widget, a new forum topic appears in the group.
4. Staff commands (sent as messages in a topic):
   - `/claim` — assign the conversation to yourself (only one staff member can claim at a time).
   - `/close` — end the handoff, close the topic, notify the web chat visitor.
   - `/note <text>` — add an internal note (not visible to the visitor).

---

# Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Webhook shows `last_error_message` | Wrong URL or HTTPS cert issue | Verify domain, check Nginx/Caddy passes `/api/v1/webhooks/telegram` through |
| No topics created on handoff | Bot not admin in group | Re-promote; ensure "Manage Topics" is enabled |
| Bot doesn't respond to `/claim` | Webhook not receiving or update deduped | Check `getWebhookInfo` for pending updates; check API logs for `update_id` |
| Secret token mismatch | `TELEGRAM_WEBHOOK_SECRET` differs between env and setWebhook call | Regenerate secret, set in env, restart API (re-registers webhook) |
| Group ID wrong | Copied from group info instead of via `@getidsbot` | Use `@getidsbot`; ID must start with `-100` for supergroups |

---

# Off-boarding / Rotation

To rotate the bot token or webhook secret:

```bash
# 1. Revoke old token via BotFather
/revoke

# 2. Generate new token
/newbot  (or /token to just get a new token on the same bot)

# 3. Generate new webhook secret
openssl rand -hex 32

# 4. Update env vars + restart API
```

The API re-registers the webhook on startup with the new secret. Old webhook
URLs are overwritten.

---

# Security Notes

- The bot token is equivalent to a password — anyone with it controls the bot.
- The webhook secret prevents spoofed updates from non-Telegram sources.
- The supergroup is **private** — patient conversations must not be discoverable.
- Staff members added to the group should be limited to clinic personnel only.
- Remove staff members from the group when they leave the clinic.

---

# Related Documents

| Document | Relationship |
|---|---|
| `messaging-protocol.md` | WebSocket frame protocol for handoff states |
| `api-specification.md` | Webhook endpoint contract |
| `openapi/messaging.yaml` | Telegram webhook OpenAPI spec |
| `data-model/care-messaging.md` | `provider_threads` + `handover_sessions` tables |
| `architectural-decision-record.md` | ADR-0017 (Telegram as first provider) |