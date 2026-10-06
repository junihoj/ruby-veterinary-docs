# Notifications Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Notifications Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Notifications Bounded Context
>
> **Classification:** Supporting Domain
>
> **Schema:** `notifications`

---

# Purpose

Defines the tables behind the delivery layer: notifications raised by any context, delivery channels and attempts, and alert subscriptions for the admin dashboard. Notifications is a leaf schema — nothing depends on its availability for core flows.

---

# Responsibilities

The Notifications context owns:

- Notification rows (what was meant to be told to whom)
- Delivery attempts across channels (email, WhatsApp template, in-app)
- Alert subscriptions for staff (form submissions, bot escalations)

It does **not** own the business objects it reports on (orders, prescriptions, submissions). Templates reference those objects by id and type.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Notification | `notifications` | One intent; attempts are children |

---

# Entities

## notifications

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `ntf_…` |
| type | text | no | e.g. `appointment_received`, `prescription_approved`, `order_paid`, `newsletter`, `alert_form`, `alert_escalation` |
| subject | text | no | |
| body_text | text | no | Plain-language body; fallback copy is mandatory |
| audience | text | no | `owner` \| `staff` \| `public_subscriber` |
| recipient_user_id | uuid | yes | Cross-schema ref by id |
| recipient_email | citext | yes | For email channel |
| recipient_phone | text | yes | For WhatsApp template channel |
| source_type | text | no | Owning context type |
| source_id | uuid | no | Id of originating business object |
| priority | text | no | `normal` \| `high` \| `critical` |
| status | text | no | `pending` \| `partially_delivered` \| `delivered` \| `failed` \| `suppressed` |
| created_at | timestamptz | no | |
| delivered_at | timestamptz | yes | |

**Indexes**

- `ix_notifications_source` on `(source_type, source_id)`
- `ix_notifications_pending` on `(status, created_at)` where `status = 'pending'`
- `ix_notifications_recipient` on `(recipient_user_id, created_at desc)`

**Invariants**

- Owner-facing notifications never contain another client's data
- `critical` priority alerts always attempt at least dashboard + one external channel
- Suppression (newsletter unsubscribe) sets `suppressed` and does not retry

## delivery_channels

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| code | text | no | Unique: `email`, `whatsapp_template`, `in_app_dashboard` |
| adapter | text | no | Outbound adapter binding name |
| active | boolean | no | |
| config_json | jsonb | yes | Non-secret config only (from address, template ids) |

**Invariants**

- Secrets (SMTP creds, API tokens) live in the VPS environment, never in `config_json`

## delivery_attempts

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| notification_id | uuid | no | FK → `notifications.id` |
| channel_id | uuid | no | FK → `delivery_channels.id` |
| status | text | no | `queued` \| `sent` \| `delivered` \| `failed` \| `bounced` |
| provider_ref | text | yes | Message id from provider |
| error_code | text | yes | |
| error_message | text | yes | Truncated |
| attempt_number | smallint | no | Starts at 1 |
| created_at | timestamptz | no | |
| completed_at | timestamptz | yes | |

**Indexes**

- `ix_delivery_attempts_notification` on `(notification_id, created_at)`

**Invariants**

- Retry policy is bounded; permanent failures surface in the admin alert dashboard
- Provider webhook status callbacks update attempts by `provider_ref`

## alert_subscriptions

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| staff_user_id | uuid | no | Cross-schema ref by id |
| alert_type | text | no | `form_submission`, `bot_escalation`, `rx_queue`, `payment_failure`, `backup_failure` |
| channel | text | no | `dashboard`, `email`, `whatsapp` |
| active | boolean | no | |
| created_at | timestamptz | no | |

**Indexes**

- `uq_alert_subs` unique on `(staff_user_id, alert_type, channel)`

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| ← All contexts | Raise notifications via events; never call Notifications inside a critical transaction |
| → Operations | Dashboard consumes notification/alert stream for the admin alert view |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Notifications domain |
| `../domain-events.md` | `AlertRaised` and event subscribers |
| `../api-specification.md` | Admin alerts API |
| `../engineering-guidelines.md` | Outbound adapter patterns |

---

# Acceptance Criteria

- Owner notifications and staff alerts are distinguishable by `audience`
- Failed deliveries raise dashboard alerts without failing the originating user request
- Newsletter suppression is immediate
- No secrets appear in notification or attempt rows

---

# Guiding Principle

> **Tell the right person the right thing, then get out of the way. If delivery fails, visibility of the failure matters more than another blind retry.**
