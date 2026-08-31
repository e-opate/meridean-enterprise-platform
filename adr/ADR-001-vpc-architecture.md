# ADR-001: VPC & Network Architecture

## Status
Accepted

## Context
MerideanCorp's platform needs a network foundation that meets the availability requirement (survive AZ loss, no manual intervention) and the security requirement (no public access to compute or data layers by default), while anticipating future multi-team use.

## Decision
Build a multi-AZ, three-tier VPC:

- **VPC CIDR:** `10.0.0.0/16`
- **3 Availability Zones** (not 2)
- **Public subnets** (one per AZ, `10.0.0.0/24`–`10.0.2.0/24`): ALB, NAT Gateways
- **Private app subnets** (one per AZ, `10.0.10.0/24`–`10.0.12.0/24`): EC2 instances
- **Private data subnets** (one per AZ, `10.0.20.0/24`–`10.0.22.0/24`): RDS
- **NAT Gateway per AZ** (not a single shared NAT)
- **Data-tier route tables carry no NAT/internet route at all** — isolation enforced at the routing layer, independent of security group rules

CIDR ranges are grouped by tens (0–9 public, 10–19 app, 20–29 data) so a subnet's tier is readable directly from its CIDR block, without checking tags.

## Alternatives Considered

**2-AZ design** — Rejected. Cheaper, but doesn't demonstrate the availability requirement as robustly, and a 3-AZ design is standard for enterprise-grade availability targets.

**Single shared NAT Gateway** — Rejected. Cheaper (~1/3 the NAT cost), but creates a cross-AZ dependency: if the AZ hosting the shared NAT goes down, private subnets in the other AZs lose outbound internet access too — directly undermines the "survive AZ loss" requirement.

**Two-tier design (app and data sharing a subnet/route table)** — Rejected. Initially built this way, then corrected: sharing a route table between app and data subnets meant data subnets technically had an outbound internet path via NAT, contradicting the "no public access to data layer" requirement. Split into separate route tables so data subnets have zero internet route — only local VPC traffic (needed for the app tier to reach RDS).

## Consequences
- Higher cost than a minimal design (3 NAT Gateways running continuously is the single largest fixed cost in this architecture)
- Stronger, more defensible security posture: data isolation doesn't depend on someone remembering to configure a security group correctly — it's structurally impossible to route out
- Slightly more operational complexity (4 route tables instead of 2) in exchange for that isolation guarantee
