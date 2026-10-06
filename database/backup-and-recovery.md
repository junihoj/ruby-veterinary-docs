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
| PostgreSQL (all schemas) | Continuous archiving (WAL) + nightly full logical/base backup | Encrypted object storage, different region from primary |
| Object storage: medical-history uploads and CMS media | Provider-side versioning/replication with lifecycle rules | Same provider, versioned bucket |
| Application secrets and configuration | Platform secret store export, encrypted | Secure offline/manager storage |
| CDN-cached public pages | Not backed up | Regenerated on redeploy |

# Backup Schedule

| Backup | Frequency | Retention |
|--------|-----------|-----------|
| WAL / continuous | Every segment (near-continuous) | 7 days |
| Full database | Nightly | 30 days |
| Weekly full | Sunday | 12 weeks |
| Monthly full | 1st of month | 12 months |
| Object storage versions | Continuous | 90 days for uploads; per CMS policy for media |
| Pre-deploy snapshot | Automatically before every migration batch | 7 days |

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
| Full primary database loss | Provision new instance, restore to target time, replay, switch DNS/connection config |
| Region/provider outage | Restore in the secondary region from cross-region backups; CDN serves static emergency content meanwhile |
| Ransomware or malicious deletion | Break-glass restore from offline/immutable backup copies; rotate all credentials first |

Immutable (object-lock) copies are retained for the monthly backups so that deletion inside the account cannot destroy them.

# Acceptance Criteria

- RPO and RTO targets are defined and measured, not aspirational
- Backups are automated, encrypted, and stored outside the primary account/region
- Weekly automated restore verification passes
- Quarterly DR exercise is completed and documented
- Restore runbook has been executed by someone other than its author

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `database-architecture.md` | What is being protected |
| `migrations.md` | Migration failures handled before full restore |
| `../architectural-decision-record.md` | Object storage and hosting decisions |
| `../non-functional-requirements.md` | Uptime and graceful degradation commitments |

# Guiding Principle

> **Business continuity depends on proven recovery, not the existence of backup files. Every backup must be automated, encrypted, independently stored, routinely verified, and regularly restored in controlled exercises. The system is resilient only when critical services can be recovered within the stated objectives with data integrity preserved.**
