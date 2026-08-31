# MerideanCorp Enterprise Cloud Platform

I built this as a personal project to practice enterprise AWS architecture. So I made up a client — a fintech doing small-business lending — and gave myself their real problem: one on-prem server, no redundancy, shared admin logins, zero visibility into cost, onto a properly secured AWS environment.

Built manually through the Console first, on purpose — I wanted to understand every piece before Terraform.

## What it looks like

```
Internet
   │
   ▼
[ ALB + WAF ]  ──spans 3 AZs──
   │
   ▼
[ Auto Scaling Group ]  (private subnets, no public IPs)
   │
   ▼
[ RDS PostgreSQL ]  (fully isolated — zero internet route, in or out)
```

*(Full diagram: [`docs/architecture.svg`](docs/architecture.svg))*

## What I'd want a reviewer to notice

- **No SSH, anywhere.** Access is IAM + Session Manager only. No keys to leak, no port 22 open.
- **The data tier can't reach the internet even if it wanted to.** Not a firewall rule — the route table itself has no path out. Isolation you can't misconfigure your way around.
- **Nothing's hardcoded.** DB credentials live in Secrets Manager, pulled at runtime through a scoped IAM role.
- **I actually broke it on purpose.** Killed a running instance mid-session and recorded the Auto Scaling Group replacing it with zero downtime. Writeup: `docs/failure-test.md`.

## Stack

VPC (3 AZ, 3-tier) · EC2 + ASG · ALB · RDS PostgreSQL · IAM · Secrets Manager · CloudWatch · WAF · Route 53

## The honest parts

I'm not going to pretend this is flawless — here's what's cut for scope, and what I'd change first in a real production account:

| What | Why it's this way | What changes in prod |
|---|---|---|
| HTTP only | No real domain to get a TLS cert for | ACM cert + HTTPS |
| RDS is Single-AZ | Account hit a quota wall on Multi-AZ | Multi-AZ, one settings change |
| No DNS failover | Same — needs a real domain | Route 53 failover policy |
| Built by hand, no Terraform yet | Wanted the concepts solid first | Rebuilding as IaC — next phase |

## Read more

- [`docs/requirements.md`](docs/requirements.md) — the brief I wrote for myself
- [`adr/`](adr/) — every real decision, with what I rejected and why
- [`docs/network-verification.md`](docs/network-verification.md) — proof it actually works
- [`docs/failure-test.md`](docs/failure-test.md) — proof it actually recovers
