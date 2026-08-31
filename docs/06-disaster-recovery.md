# Disaster Recovery Drill: RDS Loss

The failure test (`docs/03-failure-test.md`) proved instance-level self-healing. This covers something the infrastructure can't self-heal from on its own: losing the RDS instance entirely.

## Scenario

The RDS instance (`meridean-db`, Single-AZ — see `adr/ADR-003-single-az-rds.md`) is lost or corrupted. There is no automatic failover in the current Single-AZ configuration, so recovery requires a deliberate restore.

## Recovery Process (walked through, not executed against the live instance)

1. **RDS Console → `meridean-db` → Actions → Restore to point in time.**
2. AWS restores from the automated backup (7-day retention enabled) to a **new** RDS instance — it does not overwrite or repair the original in place.
3. **The application tier's connection string must be manually repointed** to the new instance's endpoint, since the new instance has a different endpoint than the original. This is the step most likely to be forgotten under real incident pressure, and the clearest candidate for automation (e.g. via a Route 53 CNAME the app always points to, updated at restore time, rather than a hardcoded endpoint).

## Recovery Objectives

- **RPO (Recovery Point Objective):** up to 24 hours in the worst case — RDS automated backups are daily snapshots plus transaction logs, so actual data loss in a real incident would typically be much smaller (down to ~5 minutes) thanks to point-in-time recovery, but the worst-case bound is what should be quoted.
- **RTO (Recovery Time Objective):** realistically 10–20 minutes for the restore itself (comparable to the original Multi-AZ provisioning time observed during the build), plus manual time to repoint the connection string and verify the app is healthy again — call it 30 minutes end-to-end for a single engineer working the incident alone.

## Gap Identified

This RTO is **not** consistent with requirement #1 (survive failure with no manual intervention) — a Single-AZ database with a manual restore process is a meaningful gap against that requirement, and it's the direct consequence of the Multi-AZ limitation documented in ADR-003. Worth stating plainly rather than glossing over: **the data tier is the weakest link in this architecture's resilience story**, and the fix (enabling Multi-AZ) is a configuration change, not a redesign, once the account-level restriction is lifted.

## What Multi-AZ Would Change

With Multi-AZ enabled, RDS handles this scenario automatically: the standby is promoted, the endpoint DNS record updates transparently, and the application doesn't need any manual repointing — reducing this whole drill to an automatic failover measured in seconds, not a 30-minute manual process.
