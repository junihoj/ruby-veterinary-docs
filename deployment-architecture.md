# deployment-architecture.md

> **ruby-veterinary Product Requirements Specification (PRS)**
>
> **Document:** Deployment Architecture
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Architecture Standard

---

# Purpose

This document defines how ruby-veterinary is deployed and operated: the single-VPS topology, containerisation, networking, TLS, CI/CD, secrets, monitoring, and how the 99.9% uptime commitment is met without a managed platform.

It records the operational consequence of ADR-0014. Database-level recovery detail lives in `database/backup-and-recovery.md`.

---

# Deployment Philosophy

- **One VPS, one stack** - the entire product runs on a single Ubuntu server under Docker Compose; simplicity is the reliability strategy for a single clinic
- **The internet never reaches the VPS directly** - Cloudflare terminates edge TLS, caches public content, and proxies only 80/443 to the origin
- **Everything restarts itself** - containers use `restart: unless-stopped` with healthchecks; a crash is an alert, not a manual intervention
- **Off-site or it did not back up** - copies that stay on the VPS are not backups
- **Deploys are boring** - CI builds, ships, and health-checks; rollback is redeploying the previous image tag

---

# Architecture

```
                    Internet
                       |
                 Cloudflare
            DNS + CDN + edge TLS
            cache + DDoS filter
                       |  (443, origin certificate)
        +--------------v----------------------------------+
        | VPS (Ubuntu LTS, 8 GB / 4 vCPU recommended)      |
        |  UFW: 22 (SSH), 80, 443 only                     |
        |  nginx reverse proxy + static cache              |
        |                                                  |
        |  +------------------+   +--------------------+   |
        |  | Next.js 16       |   | NestJS 11 API      |   |
        |  | :3001 (internal) |   | :3000 (internal)   |   |
        |  +------------------+   +---------+----------+   |
        |                                |                |
        |  +------------------------------v-------------+  |
        |  | PostgreSQL 16 (internal only, :5432)       |  |
        |  +--------------------------------------------+  |
        |  +------------------------------+---------------+|
        |  | MinIO (S3 API, signed URLs)  | Docker volumes||
        |  +------------------------------+---------------+|
        |  swap 2 GB | log rotation | healthchecks         |
        +--------------------------------------------------+
                       |
        daily encrypted backup job (pg_dump + MinIO volume)
                       |
              Offsite S3-compatible bucket
           (separate provider/account from the VPS)
```

Only nginx's ports 80/443 are reachable from the internet (plus SSH restricted to known IPs). PostgreSQL, MinIO, the API, and the frontend bind to the internal Docker network or `127.0.0.1`.

---

# Environment Strategy

| Environment | Where | Purpose |
|-------------|-------|---------|
| Local development | Developer machine, `docker-compose.dev.yml` | Feature work, tests |
| Production | Single VPS | Everything public |

No staging environment at this scale. Changes that touch payments, prescriptions, or emergency paths are verified against sandbox providers first (per `engineering-guidelines.md` Release Management).

# Infrastructure Components

| Component | Choice | Reference |
|-----------|--------|-----------|
| Server | Ubuntu LTS on a VPS provider (8 GB RAM / 4 vCPU recommended; provider = open question in `prd.md` §14) | ADR-0014 |
| Container runtime | Docker + Docker Compose plugin | ADR-0014 |
| Reverse proxy | nginx on the host, TLS to Cloudflare origin | - |
| DNS / CDN / edge TLS | Cloudflare (free plan sufficient) | ADR-0005 |
| Frontend | `ruby-veterinary-web-frontend` container, port 3001 | ADR-0005 |
| API | `ruby-veterinary-api` container, port 3000 | ADR-0002 |
| Database | PostgreSQL 16 container, volume-mounted, internal only | ADR-0003 |
| Object storage | MinIO container (S3-compatible), private buckets, signed URLs | ADR-0009 |
| Firewall | UFW: 22, 80, 443 | - |
| Swap | 2 GB file (protects against OOM during deploys and traffic spikes) | - |

# Containerisation

Every service runs in Docker Compose with:

- **Healthchecks** - `pg_isready` for PostgreSQL, HTTP health endpoint for API and frontend; compose `depends_on` with `condition: service_healthy`
- **Restart policy** - `restart: unless-stopped` on all services
- **Resource limits** - memory/CPU caps so one runaway container cannot starve the others
- **Internal network** - application services on a private bridge network; only nginx reaches the published ports
- **Volumes** - named volumes for PostgreSQL data and MinIO data; `.env` mounted read-only

`Dockerfile`s for web and API and the compose files are created when Phase 1 implementation begins; they live in the parent repository alongside this documentation set (a future `infrastructure/` folder), following the pattern proven in the organisation's first deployment.

# Networking

| Rule | Detail |
|------|--------|
| Inbound from internet | 80/443 to nginx (Cloudflare origin), SSH (22) restricted by firewall/allowlist |
| Origin pull | Cloudflare proxies to the VPS; origin certificate installed on nginx; "Always Use HTTPS" at the edge |
| Internal | API-to-Postgres and API-to-MinIO traffic never leaves the Docker network |
| Outbound | API reaches payment gateway, WhatsApp Business API, and email provider over HTTPS only |

# CI/CD Pipeline

| Stage | Gate |
|-------|------|
| Lint & type check | ESLint + `tsc --noEmit` |
| Tests | Jest (unit/integration), OpenAPI drift check |
| Build | `next build` / `nest build`, Docker image build |
| Ship | CI connects to the VPS over SSH, pulls the release, `docker compose up -d` with healthcheck wait |
| Verify | Health endpoint, homepage fetch through Cloudflare, form submission probe |
| Rollback | Redeploy the previous image tag - one command, data unaffected |

Deploys are trunk-based and continuous. Database migrations run as a distinct step before the new code goes healthy (see `database/migrations.md`); a failed migration halts the rollout and keeps the previous version serving.

The emergency path (static contact pages served from cache) must survive a deploy in progress.

# Secrets Management

- `.env` lives on the VPS outside any repository; never committed, never printed in logs
- CI holds only the SSH credential needed to deploy
- Provider keys (payments, WhatsApp, email) are stored in the `.env` with least-privilege scopes
- Rotating a secret is a VPS file edit plus a container restart; rotation cadence is reviewed quarterly

# SSL / TLS

1. Cloudflare edge certificate for every public hostname (automatic renewal)
2. Origin certificate on nginx (Cloudflare origin CA or Let's Encrypt via Certbot if the origin must also serve direct traffic)
3. HTTP to HTTPS redirect enforced at the edge; HSTS enabled
4. TLS renewal failure alerts - an expired origin certificate would break the origin pull

# Monitoring & Observability

| Signal | How | Alert |
|--------|-----|-------|
| Uptime | External probe checks https:// every minute (cloud probe, not the VPS itself) | Page on 2 consecutive failures |
| Container health | Docker healthchecks; unhealthy container alerts | Email to clinic ops address |
| Resource pressure | CPU/RAM/disk usage; disk > 80% warns | Email + dashboard |
| TLS expiry | Certificate validity probe | Email 14/7/1 days before |
| Application errors | Structured logs with `requestId`, aggregated per service | Error-rate threshold |
| Backups | Job success + backup age (RPO breach warning) | Email |

Logs are written to files with rotation (`max-size`, `max-file`) so the disk cannot be exhausted by a noisy service.

## How 99.9% Is Met on a Single VPS

The NFR commitment in `non-functional-requirements.md` is unchanged. On one server it is earned operationally:

1. **Provider network SLA** covering the VPS itself (the only component whose uptime we buy)
2. **Self-healing containers** - `restart: unless-stopped` plus healthchecks recover crashed processes without human action
3. **Swap + resource limits** - a memory spike degrades instead of killing services
4. **Cloudflare cache** - public pages stay served from edge cache during short origin interruptions; static emergency content survives a restart
5. **Off-site backups with tested restore** - the recovery path for the worst case (`database/backup-and-recovery.md`)
6. **External monitoring** - outages are detected and measured, which is how 99.9% (~43 minutes/month) is tracked rather than assumed

Known single-VPS limits are accepted in ADR-0014: planned maintenance counts against uptime, and a full host failure means restore-to-new-VPS rather than automatic failover.

# Backups

Executed by scheduled jobs on the VPS, shipped off-site the same day:

- Nightly `pg_dump` + WAL archiving for PostgreSQL
- MinIO data volume export (uploads and CMS media)
- Encrypted, then pushed to the offsite S3-compatible bucket on a **separate provider/account**
- Weekly automated restore verification, quarterly DR drill

Full scope, retention, and procedures: `database/backup-and-recovery.md`.

# Security

- UFW allows only 22/80/443; SSH key-only, password auth disabled, fail2ban for brute force
- Unattended security updates for the OS; container images rebuilt on a regular cadence
- PostgreSQL and MinIO never published to the internet
- Root access avoided in day-to-day operations; application user in the `docker` group
- Weekly dependency audit in CI (per `engineering-guidelines.md`)

# Scaling Strategy

| Stage | Trigger | Action |
|-------|---------|--------|
| 1. Now | Baseline | Vertical resize of the VPS (minutes, no architecture change) |
| 2. Read pressure | Catalog/article queries breach budget | Tune indexes; move public reads to cached routes; consider read replica on a second node |
| 3. Write or worker contention | Subscription billing or bot traffic contends with web | Extract workers/background jobs to their own process on the same VPS, then to a second VPS |
| 4. Beyond one box | Sustained capacity need | Split API or database onto dedicated hosts; requires a new ADR |

# Related Documents

| Document | Relationship |
|----------|-------------|
| `architectural-decision-record.md` | ADR-0014 (hosting), ADR-0005 (frontend), ADR-0009 (storage) |
| `database/backup-and-recovery.md` | Backup and restore procedures this topology relies on |
| `engineering-guidelines.md` | CI/CD gates and release management |
| `non-functional-requirements.md` | The 99.9% uptime commitment met by the mechanisms above |
| `prd.md` | Hosting dependency and the open VPS provider question |

# Acceptance Criteria

- The topology described here is reproducible from a fresh VPS using only this document and the compose files
- Only ports 80/443 (and restricted SSH) are reachable from the internet
- Every container restarts and healthchecks automatically; crash alerts fire
- Backups leave the VPS daily and a restore has been proven
- Uptime is measured externally, and the 99.9% tracking is in place before launch

# Guiding Principle

> **One box, no surprises. Everything on it restarts itself, everything important leaves it daily, and every deploy can be undone by one command.**
