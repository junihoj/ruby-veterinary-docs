# backup-and-recovery.md

> **ruby-veterinary Documentation**
>
> **Document:** Backup and Recovery
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

This document defines what is backed up, how often, where copies live, how long they are kept, how a restore is performed, and how the whole process is proven to work.

Business continuity for a clinic depends on proven recovery, not on the existence of backup files.

---

# Recovery Objectives

| Objective | Target | Rationale |
|-----------|--------|-----------|
| **RPO** (data loss tolerance) | ≤ 15 minutes | At most one billing cycle or a handful of form submissions lost |
| **RTO** (time to restore) | ≤ 2 hours for the database; ≤ 4 hours full service | Emergency contact pages must return first, within minutes |

The first recovery priority is always the emergency surface (hours, phone, address, static pages) - it can be restored from CDN cache alone while the database is still recovering.

# Backup Scope

| Data | Method | Location |
|------|--------|----------|
| PostgreSQL (all schemas) | Continuous archiving (WAL) + nightly full logical backup | Encrypted, pushed daily to an offsite S3-compatible bucket on a **separate provider/account from the VPS** |
| Object storage: MinIO volume (medical-history uploads, CMS media) | Daily export of the MinIO data volume | Same offsite bucket, encrypted |
| Application secrets and configuration | Encrypted copy of the VPS `.env` on each change | Password manager / secure offline storage - never on the VPS alone |
| Application code and images | Rebuilt from git history by CI | Git hosting (and image tags retained on rollback) |
| CDN-cached public pages | Not backed up | Regenerated on redeploy; Cloudflare serves stale cache during recovery |

# Backup Schedule

| Backup | Frequency | Retention |
|--------|-----------|-----------|
| WAL / continuous | Every segment (near-continuous) | 7 days |
| Full database | Nightly | 30 days |
| Weekly full | Sunday | 12 weeks |
| Monthly full | 1st of month | 12 months |
| MinIO volume export | Nightly | 90 days for uploads; per CMS policy for media |
| Pre-deploy snapshot | Automatically before every migration batch | 7 days |

Every job encrypts before upload and runs from the VPS to the offsite bucket; a job that fails to leave the VPS is treated as a failed backup.

# Security of Backups

- Encrypted at rest with keys separate from the primary database's keys
- Encrypted in transit
- Access restricted to the operational break-glass role; every access alerting
- Backup storage credentials rotated and never present in the application

# Retention and Deletion

Retention honours both operational needs and privacy obligations:

- Client medical-history uploads are deleted on verified client deletion request; backups containing them age out through the retention schedule above, and the deletion request is logged
- Audit rows are retained beyond business data where required by the clinic's policy
- Retention expiry is automated, not manual

# Restore Procedures

## Database Restore (point in time)

1. Declare the incident and stop deploys
2. Identify target point (never "latest" by default - choose the last known good)
3. Provision a restore instance in an isolated environment
4. Restore base backup, replay WAL to the target time
5. Verify integrity: row counts on critical tables (orders in range, prescription decisions), last audit entry, spot-check referential counts
6. Verify application behaviour against the restored instance
7. Promote: switch the application to the restored database (or copy forward writes made after the target point, if feasible)
8. Verify production: health check, form submission probe, checkout smoke test
9. Post-incident review within 48 hours; file follow-ups

## Object Storage Restore

Repoint or copy objects from version history; verify signed URL access for a sample of recent uploads.

## Partial Failure

If only a service misbehaves (bad deploy, corrupted rows), prefer: rollback application -> forward-fix migration -> restore affected tables from the same-day backup, in that order, before any full restore.

# Verification and Drills

| Check | Frequency | Owner |
|-------|-----------|-------|
| Automated restore to a scratch instance with row-count assertions | Weekly | Pipeline |
| Full DR exercise restoring to a clean environment and running smoke tests | Quarterly | Engineering |
| Backup alert on any missed or failed job | Per job | Alerting |
| Review of RPO/RTO actuals after any real restore | Per incident | Engineering + practice manager |

A backup that has never been restored is treated as unverified.

# Monitoring and Alerting

- Job success/failure per backup type, with alert on failure or silence
- Backup age alert (RPO breach warning)
- Storage capacity trend for backup buckets
- WAL archiving lag

Failures surface in the admin alert dashboard alongside other operational signals (`../functional-requirements.md`, Notifications & Management).

# Disaster Recovery

| Scenario | Response |
|----------|----------|
| Single table or schema corruption | Point-in-time restore of the affected schema into a new database; replay writes; swap |
| Full primary database loss | Restore from the offsite bucket onto the same VPS (or a fresh one), verify, restart the stack |
| Total VPS loss (provider failure, destruction) | Provision a replacement VPS, rebuild the stack from compose files, restore PostgreSQL and MinIO from the offsite bucket, repoint Cloudflare DNS; static emergency content is served from edge cache throughout |
| Ransomware or malicious deletion | Break-glass restore from the offsite bucket's immutable/object-lock copies; rotate all credentials first |

Because backups live on a separate provider from the VPS, the total-loss path never depends on the failed machine.

# Acceptance Criteria

- RPO and RTO targets are defined and measured, not aspirational
- Backups are automated, encrypted, and stored off the VPS on a separate provider/account
- Weekly automated restore verification passes
- Quarterly DR exercise is completed and documented, including at least one restore onto a clean machine
- Restore runbook has been executed by someone other than its author

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `database-architecture.md` | What is being protected |
| `migrations.md` | Migration failures handled before full restore |
| `../deployment-architecture.md` | VPS topology these jobs run on and the offsite bucket destination |
| `../architectural-decision-record.md` | Object storage and hosting decisions |
| `../non-functional-requirements.md` | Uptime and graceful degradation commitments |

# Guiding Principle

> **Business continuity depends on proven recovery, not the existence of backup files. Every backup must be automated, encrypted, independently stored, routinely verified, and regularly restored in controlled exercises. The system is resilient only when critical services can be recovered within the stated objectives with data integrity preserved.**
