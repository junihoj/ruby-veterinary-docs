# Telegram Setup Runbook

> **ruby-veterinary Documentation**
>
> **Document:** Telegram Handoff — Setup Runbook
>
> **Version:** 1.1.0
>
> **Status:** Living
>
> **Owner:** ruby-veterinary
>
> **Classification:** Operations

---

# Purpose

This runbook guides setup of Telegram as the clinic's staff handoff channel, in
two environments:

- **Development** — a developer exposes the local API through a named Cloudflare
  tunnel and connects a dedicated dev bot.
- **Production** — a staff member creates the clinic-facing bot and private
  forum group, then connects it to the deployed API.

Both environments use the same model: one Telegram bot, one private supergroup
with forum topics enabled, and a webhook to
`/api/v1/webhooks/telegram`.

Estimated time: **30 minutes for development**, **15 minutes for production**
once the API is deployed.

---

# Environment Matrix

| | Development | Production |
|---|---|---|
| Bot username | `ruby_vet_dev_bot` (fallback: `ruby_vet_handoff_dev_bot`) | `ruby_vet_handoff_bot` |
| Bot display name | `Ruby Vet Handoff (Dev)` | `Ruby Veterinary Handoff` |
| Group | `Ruby Vet Handoff (Dev)` | `Ruby Vet Handoff` |
| Public webhook host | `https://dev-api.ruby-veterinary.com` (named tunnel) | `https://<prod-api-domain>` |
| Webhook path | `/api/v1/webhooks/telegram` | `/api/v1/webhooks/telegram` |
| Env file | `ruby-veterinary-api/.env` (gitignored) | VPS `.env` (outside the repo) |
| Webhook secret | Generated once for dev | Different secret from dev |

> **Never share one bot between environments.** A bot can have exactly one
> webhook URL. Pointing the dev webhook at a shared bot silently redirects
> production updates to the developer machine (or breaks production entirely).

---

# Prerequisites

## Development

- A Telegram account
- A Cloudflare-managed DNS zone (the named tunnel needs a hostname)
- `cloudflared` installed
- The API running locally on `http://localhost:3000`
- Write access to `ruby-veterinary-api/.env`

## Production

- A Telegram account (personal or clinic phone number)
- Admin access to set environment variables on the API server
- A publicly accessible HTTPS domain for the API (webhook delivery requires HTTPS)

---

# Part A — Development Setup

Use a dedicated dev bot and dev group. They are disposable: you can revoke the
bot and delete the group without touching production.

## A1. Create the dev bot

1. Open Telegram and search for `@BotFather`.
2. Send `/newbot`.
3. Follow the prompts:
   - **Bot name:** `Ruby Vet Handoff (Dev)`
   - **Bot username:** `ruby_vet_dev_bot` (must end in `bot`, no hyphens, unique
     globally; use `ruby_vet_handoff_dev_bot` or `rubyvet_dev_bot` if taken)
4. BotFather returns a **token** like `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`.
   - Store it in the team password manager. It is shown only once.
5. Optional: `/setdescription`, `/setabouttext`, `/setuserpic` for the dev bot.

## A2. Create the dev forum group

1. Telegram → **New Group**.
2. **Group name:** `Ruby Vet Handoff (Dev)`.
3. **Group type:** Private.
4. Add a second test/staff account (Telegram requires ≥ 2 members).
5. Group info → **Edit** → **Group Type** → enable **Topics**.
   - Enabling Topics converts the group to a supergroup/forum. Every handoff
     conversation becomes its own topic.

## A3. Add the bot and grant permissions

1. Group info → **Add Members** → search `@ruby_vet_dev_bot` → add.
2. **Promote to admin** (required to create and manage topics):
   - **Administrators** → **Add Admin** → select the bot.
   - Enable only:
     - ✅ Manage Topics
     - ✅ Send Messages
     - ❌ everything else off
   - **Save**.

## A4. Capture the group ID and webhook secret

1. Post any message in the group.
2. Forward it to `@getidsbot` and note the **chat ID** — it starts with `-100`
   (e.g., `-1001234567890`).
3. Remove `@getidsbot` from the group.
4. Generate a dev webhook secret (32+ characters):

   ```bash
   openssl rand -hex 32
   ```

   PowerShell alternative:

   ```powershell
   -join ((1..32) | ForEach-Object { '{0:x2}' -f (Get-Random -Maximum 256) })
   ```

5. Save the secret. It goes in `TELEGRAM_WEBHOOK_SECRET`.

## A5. Set local environment variables

Add these to `ruby-veterinary-api/.env` (gitignored — never commit them):

```env
TELEGRAM_BOT_TOKEN=<dev bot token>
TELEGRAM_FORUM_GROUP_ID=-1001234567890
TELEGRAM_WEBHOOK_SECRET=<dev secret from A4>
```

**Do not prefix any of these variables with `NEXT_PUBLIC_`.** These are
server-side secrets and must never be exposed to the browser.

> **Implementation status:** the API does not read `TELEGRAM_*` yet. The
> variables are inert until the M6b Telegram handoff code lands. Do not expect
> the API to register the webhook automatically before then — register it
> manually per A8.

## A6. Expose the local API with a named Cloudflare tunnel

Telegram cannot deliver webhooks to `localhost`; the URL must be public HTTPS.
A **named** tunnel keeps the same hostname across restarts, so the webhook does
not need re-registering every time.

1. Install `cloudflared`:
   - Windows: `winget install --id Cloudflare.cloudflared`
   - macOS: `brew install cloudflared`
   - Linux: follow the Cloudflare package instructions
2. Authenticate and pick the Cloudflare zone:

   ```bash
   cloudflared tunnel login
   ```

3. Create the tunnel:

   ```bash
   cloudflared tunnel create ruby-vet-dev
   ```

   Note the tunnel UUID printed by the command.

4. Route DNS for the dev hostname:

   ```bash
   cloudflared tunnel route dns ruby-vet-dev dev-api.ruby-veterinary.com
   ```

5. Create `~/.cloudflared/config.yml` (on Windows:
   `C:\Users\<you>\.cloudflared\config.yml`):

   ```yaml
   tunnel: <tunnel-uuid>
   credentials-file: C:\Users\<you>\.cloudflared\<tunnel-uuid>.json
   ingress:
     - hostname: dev-api.ruby-veterinary.com
       service: http://localhost:3000
     - service: http_status:404
   ```

6. Run the tunnel:

   ```bash
   cloudflared tunnel run ruby-vet-dev
   ```

7. Verify the API is reachable through the tunnel before touching Telegram:

   ```bash
   curl https://dev-api.ruby-veterinary.com/health
   ```

   If `/health` fails, fix the tunnel or local API first. Telegram errors will
   be misleading otherwise.

## A7. Verify Telegram wiring

These checks work even before the webhook controller exists:

```bash
# Token is valid
curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getMe"

# Group is a forum and the bot is a member
curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getChat?chat_id=$TELEGRAM_FORUM_GROUP_ID"
```

Expect `"is_forum": true` in the `getChat` response.

Confirm the bot can actually manage topics:

```bash
curl -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/createForumTopic" \
  -d "chat_id=$TELEGRAM_FORUM_GROUP_ID" \
  -d "name=setup test"
```

Delete the test topic with the returned `message_thread_id`:

```bash
curl -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/deleteForumTopic" \
  -d "chat_id=$TELEGRAM_FORUM_GROUP_ID" \
  -d "message_thread_id=<message_thread_id>"
```

If `createForumTopic` returns 403, the bot is not an admin or lacks
**Manage Topics**.

## A8. Register the dev webhook

Register only after `/api/v1/webhooks/telegram` exists (M6b). Registering
earlier makes Telegram retry into a 404 and clutters `getWebhookInfo`.

```bash
curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/setWebhook" \
  -d "url=https://dev-api.ruby-veterinary.com/api/v1/webhooks/telegram" \
  -d "secret_token=$TELEGRAM_WEBHOOK_SECRET"
```

Verify:

```bash
curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getWebhookInfo"
```

Expect:

- `"url"` matches the dev hostname + `/api/v1/webhooks/telegram`
- `"pending_update_count"` is `0`
- no `"last_error_message"`

While the tunnel is not running, Telegram keeps retrying and the pending count
grows. Stop the webhook when you finish for the day:

```bash
curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/deleteWebhook"
```

---

# Part B — Production Setup

Run this only after the API is deployed at a public HTTPS domain. Step 5 refers
to the VPS `.env`; never store production secrets in the repository.

## B1. Create the production bot

1. Open Telegram and search for `@BotFather`.
2. Send `/newbot`.
3. Follow the prompts:
   - **Bot name:** `Ruby Veterinary Handoff` (display name, can be changed later)
   - **Bot username:** `ruby_vet_handoff_bot` (must end in `bot`, must be unique globally)
4. BotFather returns a **token** — a long string like `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`.
   - **Copy this token now.** It is shown only once. If lost, revoke and regenerate with `/revoke` then `/newbot`.

5. (Optional) Set bot profile details via BotFather:
   - `/setdescription` — "Ruby Veterinary Clinic — staff handoff assistant"
   - `/setabouttext` — "Internal use only. Connects web chat visitors to our veterinary team."
   - `/setuserpic` — Upload the clinic logo

## B2. Create the handoff supergroup

1. In Telegram, tap the pencil icon → **New Group**.
2. **Group name:** `Ruby Vet Handoff`
3. **Group type:** Private (not public — client conversations are confidential).
4. Add at least one other staff member as an initial member (Telegram requires ≥ 2 members to create a group; you can remove them after the bot joins).
5. After creating the group:
   - Tap the group name → **Edit** (pencil icon) → **Group Type**.
   - Enable **Topics** (this turns the group into a forum).
   - Each conversation from a web chat visitor will create a new topic in this forum.

## B3. Add the bot to the group

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

## B4. Get the group ID and webhook secret

1. **Group ID:**
   - Send a message in the group (any text).
   - Forward that message to `@getidsbot` (a Telegram bot that reveals IDs).
   - Note the **chat ID** — it starts with `-100` (e.g., `-1001234567890`).
   - Remove `@getidsbot` from the group after.

2. **Webhook secret:**
   - Generate a strong random secret (32+ characters), different from dev:
     ```bash
     openssl rand -hex 32
     ```
   - Save this; it goes in `TELEGRAM_WEBHOOK_SECRET`.

## B5. Set environment variables

On the API server, set these three environment variables in the VPS `.env`:

```env
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
TELEGRAM_FORUM_GROUP_ID=-1001234567890
TELEGRAM_WEBHOOK_SECRET=<the output from openssl rand -hex 32>
```

Restart the API service after setting them.

Once M6b is deployed, the API registers the webhook with Telegram on startup
(POST `https://api.telegram.org/bot<TOKEN>/setWebhook` with
`url=https://<your-domain>/api/v1/webhooks/telegram` and
`secret_token=<TELEGRAM_WEBHOOK_SECRET>`). Before that, register it manually
with the command in B6.

**Do not prefix any of these variables with `NEXT_PUBLIC_`.** These are
server-side secrets and must never be exposed to the browser.

## B6. Register and verify the webhook

If the API does not register the webhook automatically, run:

```bash
curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/setWebhook" \
  -d "url=https://<prod-api-domain>/api/v1/webhooks/telegram" \
  -d "secret_token=$TELEGRAM_WEBHOOK_SECRET"
```

Then check the status:

```bash
curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getWebhookInfo"
```

Verify:

- `"url"` is your API domain + `/api/v1/webhooks/telegram`
- `"has_custom_certificate"` is `false` (you're using a standard HTTPS cert)
- `"pending_update_count"` starts at `0`
- No `"last_error_message"`

Send a test message to the bot (DM the bot, not the group):

- The API logs should show a received update.
- No forum topic should be created (DMs are not handoff conversations).

## B7. Staff onboarding

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
| Webhook returns 404 / retries grow | Webhook controller not deployed yet (M6b) | Defer `setWebhook`, or run `deleteWebhook` until the endpoint exists |
| No topics created on handoff | Bot not admin in group | Re-promote; ensure "Manage Topics" is enabled |
| `createForumTopic` returns 403 | Bot lacks Manage Topics | Re-check admin permissions in A3/B3 |
| `getChat` shows `"is_forum": false` | Topics not enabled on the group | Group info → Edit → Group Type → enable Topics |
| Bot doesn't respond to `/claim` | Webhook not receiving or update deduped | Check `getWebhookInfo` for pending updates; check API logs for `update_id` |
| Secret token mismatch | `TELEGRAM_WEBHOOK_SECRET` differs between env and setWebhook call | Regenerate secret, set in env, restart API (re-registers webhook) |
| Group ID wrong | Copied from group info instead of via `@getidsbot` | Use `@getidsbot`; ID must start with `-100` for supergroups |
| Dev tunnel returns 502/1033 | Tunnel not running or `config.yml` ingress wrong | Start `cloudflared tunnel run ruby-vet-dev`; check the service port is 3000 |
| Dev tunnel hostname unreachable | DNS route missing | Re-run `cloudflared tunnel route dns ruby-vet-dev dev-api.ruby-veterinary.com` |
| Dev updates appear in production (or vice versa) | Both environments share one bot | Create separate bots per environment (A1 / B1); one webhook per bot |

---

# Off-boarding / Rotation

Rotate each environment separately; rotating dev must not touch production.

```bash
# 1. Revoke old token via BotFather
/revoke

# 2. Generate new token
/newbot  (or /token to just get a new token on the same bot)

# 3. Generate new webhook secret
openssl rand -hex 32

# 4. Update env vars + restart API
```

Once M6b is deployed, the API re-registers the webhook on startup with the new
secret and old webhook URLs are overwritten. Before then, re-run `setWebhook`
manually with the new secret. For development, also re-run `setWebhook`, or
`deleteWebhook` when the tunnel is not in use.

---

# Security Notes

- The bot token is equivalent to a password — anyone with it controls the bot.
- The webhook secret prevents spoofed updates from non-Telegram sources.
- Dev and production must use **separate bots and separate secrets**.
- The supergroup is **private** — patient conversations must not be discoverable.
- Staff members added to the group should be limited to clinic personnel only.
- Remove staff members from the group when they leave the clinic.
- Secrets live in `.env` files only: the local gitignored file for development,
  the VPS `.env` for production. Never commit them or expose them to the browser.

---

# Related Documents

| Document | Relationship |
|---|---|
| `messaging-protocol.md` | WebSocket frame protocol for handoff states |
| `api-specification.md` | Webhook endpoint contract |
| `openapi/messaging.yaml` | Telegram webhook OpenAPI spec |
| `data-model/care-messaging.md` | `provider_threads` + `handover_sessions` tables |
| `architectural-decision-record.md` | ADR-0017 (Telegram as first provider) |
| `implementations/api-plan.md` | M6b — Telegram handoff implementation scope |
