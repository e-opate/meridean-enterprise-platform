# ADR-002: HTTP-Only Load Balancer, No DNS Failover Routing

## Status
Accepted (lab environment limitation, documented)

## Context
A production fintech application would terminate TLS at the ALB using an AWS Certificate Manager (ACM) certificate, and would use a Route 53 hosted zone with a failover routing policy for DNS-level resilience. Both require a registered domain name. This project uses AWS-generated endpoints only (ALB DNS name), with no domain registered.

## Decision
- The ALB listener is configured for HTTP:80 only. No HTTPS listener, no ACM certificate.
- A Route 53 health check monitors the ALB's HTTP endpoint directly, demonstrating the health-check mechanism, but without a hosted zone or failover routing policy layered on top (since there's no domain to route).

## Alternatives Considered
**Register a real domain for this project** — Rejected for now. Adds ongoing cost and scope beyond demonstrating the architecture pattern; the mechanism (health checks, ALB routing, target health) is fully demonstrable without it.

## Consequences
- Traffic to the ALB is unencrypted in this lab environment — acceptable here since no real customer or financial data is involved, unacceptable in production.
- DNS-level failover (routing away from a fully degraded region/endpoint) isn't demonstrated, only endpoint-level health checking.

## Path to Production
1. Register a domain, create a Route 53 hosted zone.
2. Request an ACM certificate for the domain, validate via DNS.
3. Add an HTTPS:443 listener to the ALB referencing the ACM cert; redirect HTTP:80 → HTTPS:443.
4. Configure a failover (or latency-based, for multi-region) routing policy in Route 53 pointing at the ALB, backed by the existing health check.
