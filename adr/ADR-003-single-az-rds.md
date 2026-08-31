# ADR-003: Single-AZ RDS Deployment (Multi-AZ Deferred)

## Status
Accepted (account-level constraint, documented)

## Context
Requirement #1 calls for surviving AZ loss with no manual intervention, which for the data tier means Multi-AZ RDS (automatic failover to a standby in a second AZ). When provisioning RDS, the AWS account used for this project restricted Multi-AZ deployment behind a support-plan/quota upgrade not available in this environment.

## Decision
Deploy RDS as a **Single-AZ PostgreSQL instance** (`db.t3.micro`) rather than blocking the project on an account-level restriction outside engineering control. The limitation is documented here rather than silently worked around or hidden.

Additional decisions made alongside this:
- **Password authentication** (not IAM database authentication) — simpler to integrate for this project's scope; credentials are still never hardcoded, stored in AWS Secrets Manager instead.
- **`sslmode=require`** (not `verify-full`) for application/test connections — encrypts the connection without requiring the AWS RDS CA bundle to be present on every connecting client, appropriate for a lab environment.

## Alternatives Considered
**Request a quota/support-plan increase to unlock Multi-AZ** — Not pursued; out of scope for a portfolio project and adds cost without adding to the architecture story, since the reasoning for Multi-AZ is already fully documented in ADR-001 and this ADR.

**Use Aurora Serverless instead of standard RDS** — Rejected. Standard RDS PostgreSQL is what most junior/mid cloud roles actually test on, and Aurora introduces a different (and more complex) architecture than what this project set out to demonstrate.

## Consequences
- The data tier does **not** currently meet requirement #1's availability target — this is a known, explicitly flagged gap, not an oversight.
- If the primary AZ hosting the RDS instance fails, there is no automatic failover; recovery would require manual intervention (restore from automated backup, which is enabled with 7-day retention).

## Path to Production
1. Request an AWS service quota increase (or upgrade support plan) to unlock Multi-AZ deployment.
2. Modify the existing RDS instance to Multi-AZ (`aws rds modify-db-instance --multi-az`) — this can be done with minimal downtime, no rebuild required.
3. Switch client connections to `sslmode=verify-full` with the RDS CA bundle pinned.
4. Enable IAM database authentication as a further hardening step, removing the need for a long-lived database password entirely.
