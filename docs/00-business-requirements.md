# Business Requirements

## Scenario

MerideanCorp is a fintech company offering small-business lending. Their customer-facing loan application platform currently runs on a handful of on-prem servers in a single data center. Growth has outpaced that setup: no redundancy, ad-hoc access control (shared admin logins), no formal monitoring, and finance has no visibility into infrastructure cost. This is their first cloud migration.

## Requirements

**1. Availability**
The platform must survive the loss of a single AWS Availability Zone with no manual intervention. Target: 99.9% uptime.

**2. Scalability**
Must handle unpredictable traffic spikes (marketing campaigns, month-end loan processing) without manual resizing. Scale-out and scale-in should be automatic.

**3. Security & Compliance Posture**
No shared credentials. All access must be individually attributable (fintech regulators expect to know who did what). Data at rest and in transit must be encrypted. Public internet access to compute and data layers is prohibited by default.

**4. Observability**
Infrastructure and application health must be visible without SSHing into a server to check. Alerts must fire before customers notice a problem, not after.

**5. Cost Accountability**
Finance must be able to see spend broken down by environment/team without asking engineering. No untagged resources.

**6. Access Control / Multi-Team**
Engineering and (eventually) a compliance/audit function need different access levels. IAM design must anticipate more than one team using this environment.

**7. Documented Decision-Making**
Every non-trivial infrastructure decision must be recorded with the alternatives considered and why they were rejected (ADR format).

## How This Project Maps to Each Requirement

| Requirement | Where it's addressed |
|---|---|
| Availability | 3-AZ VPC, ASG, ALB, RDS (ADR-001, ADR-003) |
| Scalability | Auto Scaling Group, target-tracking (Phase 2) |
| Security | IAM least-privilege, SSM-only access, Secrets Manager, WAF, routing-layer data isolation |
| Observability | CloudWatch alarms + dashboard, Route 53 health check |
| Cost accountability | Phase 3 — tagging strategy, Cost Explorer analysis |
| Access control | IAM role scoped to EC2, expandable per-team in Phase 3 |
| Documented decisions | `adr/` directory, one ADR per major decision |
