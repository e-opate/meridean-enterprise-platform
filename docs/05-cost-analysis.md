# Cost Analysis

## What Actually Cost Money

| Resource | Rate | Notes |
|---|---|---|
| 3× NAT Gateway | ~$0.045/hr each (~$97/mo if left running continuously) | Largest fixed cost in this architecture, by far |
| ALB | ~$0.0225/hr + minor LCU charges | Small, steady |
| WAF Web ACL | ~$5/mo base + ~$1/mo per rule group | 2 rule groups attached |
| RDS (Single-AZ, `db.t3.micro`) | Free-tier eligible | $0 within free-tier hours |
| EC2 (2× t3.micro) | Free-tier eligible (750 hrs/mo combined) | $0 within free-tier hours |

*[Screenshot: Cost Explorer breakdown by Tier tag](../screenshots/cost-explorer.png)*

## Cost Governance Applied

- All resources tagged (`Project`, `Environment`, `Tier`, `Owner`, `ManagedBy`) with one documented exception (WAF — see `docs/04-security-audit.md`), enabling the breakdown above.
- Expensive resources (NAT Gateways, RDS, ALB) were deliberately built, verified, and torn down between work sessions rather than left running — treated as a working discipline, not just a one-time cost-saving step.

## What I'd Optimize in a Real Production Account

1. **NAT Gateway count is the biggest lever.** One-per-AZ was the right call for the stated availability requirement (survive AZ loss with no manual intervention), but it's roughly 3x the cost of a single shared NAT. If the business tolerated brief manual intervention during an AZ event, a single NAT Gateway (or NAT instances, cheaper but higher-maintenance) would cut this cost significantly — a real trade-off between spend and RTO, not a free win either way.
2. **3 AZs vs 2.** Same trade-off pattern — 3 AZs gives better resilience but proportionally higher NAT/subnet cost. For a workload with MerideanCorp's actual traffic (currently: one on-prem server), 2 AZs might be the more honest starting point, scaling to 3 as real load justifies it.
3. **Right-sizing after real traffic data exists.** Everything here is provisioned at minimum viable size (`t3.micro`, `db.t3.micro`) because there's no real production load yet. The next cost review should happen after actual usage patterns are known, not before.

## Summary

The architecture's cost profile is dominated by availability decisions (NAT redundancy, AZ count), not by compute or storage — which is the right shape for an early-stage but security/availability-conscious platform. The main lever for future savings is deciding how much availability the business actually needs to pay for, not trimming compute.
