# AI Assistant Data Model

> **ruby-veterinary Documentation**
>
> **Document:** AI Assistant Data Model
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** AI Assistant Bounded Context
>
> **Classification:** Supporting Domain
>
> **Schema:** `ai_kb`

---

# Purpose

Defines the tables behind the RAG-powered AI bot: knowledge-base documents, their chunked embeddings, ingestion job tracking, and PII scan findings. This schema lives outside the Prisma-managed set and is accessed through a dedicated `pg.Pool` with a least-privilege role (`ai_kb_app`), ensuring the AI module can never touch clinical or identity schemas.

---

# Responsibilities

The AI Assistant context owns:

- Knowledge-base documents (policies, services, hours, pricing, catalog, FAQ) with versioned, idempotent ingestion
- Document chunking and vector embeddings via `text-embedding-3-small` (1536 dimensions)
- Ingestion job queue (in-process worker, DB-backed per ADR-0015 rule 7) for embedding, reindexing, archiving, and deletion
- PII scan findings as advisory signals for the human approval gate
- Retrieval queries: cosine similarity search over active document chunks, filtered by category when the caller provides context

It does **not** own chat conversations, bot rules, or handover state (those live in `messaging`). The AI bot is invoked by the Messaging context when a message is not flagged as emergency or containing private data; this schema stores the knowledge the bot retrieves, not the conversation it participates in.

---

# Aggregate Roots

| Aggregate | Root | Notes |
|-----------|------|-------|
| Document | `documents` | Chunks, ingestion jobs, and findings cascade from it; one document → many chunks, one active version at a time |

---

# Entities

## ai_kb.documents

A knowledge-base document ingested from an uploaded file or URL. Versioned: re-ingesting the same content (matched by SHA-256 hash) bumps the version and replaces old chunks atomically.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; DEFAULT `gen_random_uuid()` |
| title | text | no | Human-readable label shown in admin review |
| category | text | no | `policy` \| `services` \| `hours` \| `pricing` \| `catalog` \| `faq` \| `other` |
| source | text | yes | Original filename or URL for traceability |
| content_hash | text | no | SHA-256 of raw ingestion content; used for idempotency |
| status | text | no | `pending_review` \| `active` \| `archived`; DEFAULT `pending_review` |
| effective_date | date | yes | When the content takes effect (policies, price changes) |
| version | integer | no | DEFAULT 1; incremented on re-ingestion of same content_hash |
| created_at | timestamptz | no | DEFAULT `now()` |
| updated_at | timestamptz | no | Updated on status change or version bump |

**Indexes**

- `uq_documents_hash_version` unique on `(content_hash, version)`
- `ix_documents_status` on `(status)`

**Invariants**

- A document in `pending_review` cannot be retrieved by the bot; only `active` documents are retrievable.
- `archived` documents are excluded from retrieval but retained for audit.
- Re-ingesting a document with the same `content_hash` bumps `version` and replaces old chunks atomically (delete old chunks → insert new chunks → update document version).
- Archive/delete cascade to chunks (FK `ON DELETE CASCADE`).
- Human approval is required before a document transitions from `pending_review` to `active`. Approval is performed through the admin dashboard or CLI, not through automated rules.
- `effective_date` is advisory; retrieval filters on `status = 'active'`, not date.

## ai_kb.chunks

A text segment of a document, stored with its vector embedding for similarity search. Created during ingestion with `embedding = NULL`, then batch-embedded by the worker.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; DEFAULT `gen_random_uuid()` |
| document_id | uuid | no | FK → `documents.id` ON DELETE CASCADE |
| chunk_index | integer | no | Ordinal within the document (0-based) |
| content | text | no | The chunk text; sized to fit within the embedding model's token limit |
| embedding | vector(1536) | yes | `text-embedding-3-small` embedding; NULL until the worker processes it |
| token_count | integer | yes | Token count from the embedding API response |
| metadata | jsonb | yes | Arbitrary metadata (page number, section heading, chunking strategy) |
| created_at | timestamptz | no | DEFAULT `now()` |

**Indexes**

- `uq_chunks_doc_index` unique on `(document_id, chunk_index)`
- `ix_chunks_embedding_ivfflat` ivfflat index on `embedding vector_cosine_ops` with `lists = 100` (created after first population; not created in schema migration)

**Invariants**

- Chunks are created during ingestion with `embedding = NULL`, then batch-embedded by the worker polling `ingestion_jobs`.
- Reindexing a document deletes all its chunks and re-creates them from the stored content.
- Retrieval query pattern:
  ```sql
  SELECT c.content, c.metadata, d.title, d.category
  FROM ai_kb.chunks c
  JOIN ai_kb.documents d ON d.id = c.document_id
  WHERE d.status = 'active'
  ORDER BY c.embedding <=> $query_embedding
  LIMIT k;
  ```
- The ivfflat index requires at least `lists` rows before it becomes effective; the worker creates the index after the first bulk ingestion, not in the schema DDL.

## ai_kb.ingestion_jobs

A work queue for the in-process embedding worker. Jobs are claimed atomically, processed off the request path, and never retried automatically on failure (manual re-queue via admin CLI).

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; DEFAULT `gen_random_uuid()` |
| document_id | uuid | yes | FK → `documents.id`; nullable for housekeeping jobs |
| kind | text | no | `ingest` \| `reindex` \| `archive` \| `delete` |
| status | text | no | `queued` \| `running` \| `done` \| `failed`; DEFAULT `queued` |
| error | text | yes | Error message or stack trace on failure |
| created_at | timestamptz | no | DEFAULT `now()` |
| started_at | timestamptz | yes | Set when the worker claims the job |
| finished_at | timestamptz | yes | Set on completion or failure |

**Indexes**

- `ix_ingestion_jobs_queue` on `(status, created_at)` — the worker's claim query:
  ```sql
  UPDATE ai_kb.ingestion_jobs
  SET status = 'running', started_at = now()
  WHERE id IN (
    SELECT id FROM ai_kb.ingestion_jobs
    WHERE status = 'queued'
    ORDER BY created_at
    LIMIT $batch_size
    FOR UPDATE SKIP LOCKED
  )
  RETURNING *;
  ```

**Invariants**

- Embedding happens off the request path; the upload endpoint inserts chunks and enqueues a job, then returns immediately.
- The worker claims jobs with `FOR UPDATE SKIP LOCKED` for concurrent safety.
- Failed jobs are not automatically retried; an admin re-queues via CLI or dashboard.
- Deleting a document enqueues a `delete` job that removes chunks and sets `document.status = 'archived'`.

## ai_kb.ingestion_findings

Advisory PII scan results attached to a document. Findings inform the human approval gate but do not block ingestion automatically.

| Attribute | Type | Null | Notes |
|-----------|------|------|-------|
| id | uuid | no | PK; DEFAULT `gen_random_uuid()` |
| document_id | uuid | no | FK → `documents.id` ON DELETE CASCADE |
| rule | text | no | `pii_email` \| `pii_phone` \| `pii_address` \| `pii_patient_id` \| `pii_lab_value` |
| excerpt | text | yes | The matched snippet (truncated for display) |
| severity | text | no | `low` \| `medium` \| `high`; DEFAULT `low` |
| created_at | timestamptz | no | DEFAULT `now()` |

**Invariants**

- Findings are advisory only; the human approval gate is authoritative.
- A document may have zero or many findings.
- Findings are regenerated on re-ingestion; old findings are not retained across versions.

---

# Extension

The `vector` extension must be present in the PostgreSQL instance:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

This is provided by the `pgvector/pgvector:pg16` Docker Compose image, which replaces the `postgres:16-alpine` image (see ADR-0017). The extension is created in a migration that runs before any `ai_kb` table DDL.

---

# Role

The `ai_kb_app` database role enforces least-privilege isolation between the AI module and the rest of the database:

- **Password** stored in the environment as `AI_KB_DATABASE_URL` (a full connection string including the role and password); never in the repository.
- **USAGE** on schema `ai_kb` only — no access to `identity`, `clients`, `pharmacy`, `messaging`, or any other schema.
- **GRANT SELECT, INSERT, UPDATE, DELETE** on all tables in `ai_kb`.
- **No USAGE on public schema** (revoked).
- The NestJS `AiKnowledgeBaseModule` creates a dedicated `pg.Pool` from `AI_KB_DATABASE_URL`; Prisma does not connect to this schema.

```sql
-- Representative DDL (run in a migration, not hand-applied):
CREATE ROLE ai_kb_app WITH LOGIN PASSWORD '<from-env>';
GRANT USAGE ON SCHEMA ai_kb TO ai_kb_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA ai_kb TO ai_kb_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA ai_kb
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO ai_kb_app;
REVOKE USAGE ON SCHEMA public FROM ai_kb_app;
```

---

# Cross-Context References

| Direction | Reference |
|-----------|-----------|
| → Messaging | AI bot retrieves from `ai_kb` when a chat message is not flagged as emergency/private-data; the Messaging context calls the `ai-assistant` module via its public facade |
| → Operations | Admin dashboard reads `ai_kb.documents`, `ingestion_jobs`, and `ingestion_findings` for the document review workflow; ingestion CLI is an Operations concern |

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `care-messaging.md` | The Messaging data model that invokes the AI bot |
| `../architectural-decision-record.md` | ADR-0017 hybrid messaging + AI assistant decision |
| `../api-specification.md` | Document ingestion and knowledge-base search API endpoints |
| `../bounded-context.md` | `ai_kb` schema placement and context boundaries |
| `../database/database-architecture.md` | pgvector extension and `ai_kb` schema placement |
| `../prd.md` | M6 messaging requirements |

---

# Acceptance Criteria

- Document upload → chunk → embed pipeline completes without blocking the HTTP request
- A document in `pending_review` returns zero chunks from retrieval queries
- Re-ingestion of identical content (same `content_hash`) bumps version and replaces chunks atomically
- The `ai_kb_app` role cannot SELECT from `identity.users`, `clients.pets`, or any non-`ai_kb` table
- PII findings are regenerated on re-ingestion and displayed in the admin approval view
- ivfflat index is created after first population, not in the schema migration

---

# Guiding Principle

> **The knowledge base is a clinical safety boundary. No document enters the bot's retrievable set without a human saying it is safe. No embedding query runs against a schema that can see patient data. Boring isolation is the whole point.**