# Publishing Data Model

> **ruby-veterinary Documentation**
>
> **Document:** Publishing Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** Publishing Bounded Context
>
> **Classification:** Core Domain
>
> **Schema:** `publishing`

---

# Purpose

Defines the tables behind the clinic's educational blog: articles and rich text bodies, categories and tags, author references to clinic staff, embedded media, SEO metadata, newsletter subscribers, and related-article links. Content is written by clinicians and managed through the staff back office without code changes.

---

# Responsibilities

The Publishing context owns:

- Articles, drafts, and publish state
- Categories and tags
- Article authorship (references to staff profiles by id)
- Media assets attached to articles
- Per-article SEO metadata
- Newsletter subscriber list and subscribe endpoint state
- Related-article associations

It does **not** own staff profile records (Clinic Content) or email delivery mechanics (Notifications). Publish raises `ArticlePublished`; cache invalidation and newsletter dispatch hang off that event.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Article | `articles` | Body, SEO metadata, category links, and publish state move together |
| Taxonomy | `categories` / `tags` | Small, staff-managed vocabularies |

---

# Entities

## articles

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; serialised `art_…` |
| slug | text | no | Unique; changes after publish require redirect care |
| title | text | no | |
| excerpt | text | yes | Card/listing summary |
| body_md | text | no | Markdown with embedded media refs |
| cover_object_key | text | yes | MinIO key |
| status | text | no | `draft` \| `published` \| `archived` |
| category_id | uuid | yes | FK → `categories.id` (same schema) |
| author_profile_id | uuid | yes | Cross-schema ref by id → `content.staff_profiles.id` |
| published_at | timestamptz | yes | Set on first publish |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |
| seo_metadata_id | uuid | yes | FK → `seo_metadata.id` |

**Indexes**

- `uq_articles_slug` unique on `(slug)`
- `ix_articles_status_published` on `(status, published_at desc)`
- `ix_articles_category` on `(category_id, status)`

**Invariants**

- Only `published` articles appear in public reads
- `published_at` is never null for published rows and never moves on edits (cache and sitemap stability)
- Author points at an existing published staff profile; broken bylines are a review failure

## categories

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | e.g. `Dog Care` |
| slug | text | no | Unique |
| description | text | yes | |
| sort_order | integer | no | |
| created_at | timestamptz | no | |

**Indexes**

- `uq_categories_slug` unique on `(slug)`

## tags

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| name | text | no | e.g. `Nutrition` |
| slug | text | no | Unique |
| created_at | timestamptz | no | |

**Indexes**

- `uq_tags_slug` unique on `(slug)`

## article_tags

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| article_id | uuid | no | FK → `articles.id` |
| tag_id | uuid | no | FK → `tags.id` |

**Indexes**

- PK `(article_id, tag_id)`; `ix_article_tags_tag` on `(tag_id)`

## seo_metadata

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| meta_title | text | yes | ≤ 60 chars preferred; stored as given |
| meta_description | text | yes | ≤ 160 chars preferred |
| canonical_path | text | yes | Relative path override |
| og_image_object_key | text | yes | Social share image |
| no_index | boolean | no | Default false |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

## media_assets

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| object_key | text | no | MinIO key (ADR-0009) |
| mime_type | text | no | `image/jpeg`, `image/png`, `image/webp` |
| byte_size | integer | no | |
| alt_text | text | no | Accessibility: required for every embedded image |
| width_px | integer | yes | |
| height_px | integer | yes | |
| uploaded_by | uuid | yes | Cross-schema ref by id → staff user |
| created_at | timestamptz | no | |

**Invariants**

- Public serving uses WebP (or the original where conversion failed) through CDN/object URLs
- `alt_text` non-empty is enforced on upload into article bodies

## article_relations

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| article_id | uuid | no | FK → `articles.id` |
| related_article_id | uuid | no | FK → `articles.id` |
| weight | smallint | no | Ordering for "related" block |

**Invariants**

- No self-links; both sides must be published when surfaced

## newsletter_subscribers

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK |
| email | citext | no | Unique |
| status | text | no | `pending` \| `active` \| `unsubscribed` |
| consented_at | timestamptz | no | Double opt-in timestamp |
| confirmed_at | timestamptz | yes | |
| source | text | yes | e.g. `footer`, `article` |
| created_at | timestamptz | no | |

**Indexes**

- `uq_newsletter_subscribers_email` unique on `(email)`

**Invariants**

- Subscribe endpoint is rate-limited; duplicates return success without new rows (idempotent UX)
- Unsubscribe is one-click and immediate; suppressions are honoured by the delivery adapter

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → Clinic Content | `author_profile_id` → staff profile; read-only |
| → Notifications | `ArticlePublished` and subscribe confirmations consume via events |
| → Operations | Staff back office writes through this context only |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../domain-model.md` | Publishing domain |
| `../domain-events.md` | `ArticlePublished` and related events |
| `../api-specification.md` | Publishing API endpoints |
| `../prd.md` | Blog/CMS and newsletter requirements |

---

# Acceptance Criteria

- Draft articles are invisible to public and CDN until published
- Publishing is idempotent under staff retries and raises exactly one `ArticlePublished`
- Every image in an article body has stored alt text
- Newsletter consent state is explicit; unsubscribed addresses never receive dispatch

---

# Guiding Principle

> **Clinician-authored content is the trust engine. Ship it fast, keep it findable, and never let a draft leak into the public web.**
