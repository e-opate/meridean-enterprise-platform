Network Subnets

Subnet Name,Tier,Availability Zone,IPv4 CIDR,Auto-Assign Public IP
eazycat-prod-public-a,Public,us-east-1a,10.0.1.0/24
eazycat-prod-private-b,Public/Edge,us-east-1b,10.0.2.0/24
eazycat-prod-private-app-a,App (Private),us-east-1a,10.0.11.0/24
eazycat-prod-private-app-b,App (Private),us-east-1b,10.0.12.0/24
eazycat-prod-private-db-a,DB (Private),us-east-1a,10.0.21.0/24
eazycat-prod-private-db-b,DB (Private),us-east-1b,10.0.22.0/24

Routetabel## Network Addressing & Subnet Allocation

| Subnet Name | Tier | Availability Zone | IPv4 CIDR | Auto-Assign Public IP |
| :--- | :--- | :--- | :--- | :--- |
| `eazycat-prod-public-a` | Public | `us-east-1a` | `10.0.1.0/24` | Enabled |
| `eazycat-prod-private-b` | Public/Edge | `us-east-1b` | `10.0.2.0/24` | Disabled |
| `eazycat-prod-private-app-a` | App (Private) | `us-east-1a` | `10.0.11.0/24` | Disabled |
| `eazycat-prod-private-app-b` | App (Private) | `us-east-1b` | `10.0.12.0/24` | Disabled |
| `eazycat-prod-private-db-a` | DB (Private) | `us-east-1a` | `10.0.21.0/24` | Disabled |
| `eazycat-prod-private-db-b` | DB (Private) | `us-east-1b` | `10.0.22.0/24` | Disabled |

---

## Route Table Configurations

| Route Table Name | Target Gateway | Destination | Associated Subnets | Security Rationale |
| :--- | :--- | :--- | :--- | :--- |
| `eazycat-prod-public-rt` | `eazycat-prod-igw` | `0.0.0.0/0` | `public-a`, `private-b` | Direct internet ingress/egress for edge components. |
| `eazycat-prod-private-app-rt` | `eazycat-prod-nat` | `0.0.0.0/0` | `app-a`, `app-b` | Secure outbound internet access for patches without accepting inbound traffic. |
| `eazycat-prod-db-rt` | `Local` | `10.0.0.0/16` | `db-a`, `db-b` | Complete air-gap from external networks to prevent data exfiltration. |

---

## Firewall & Security Group Rules (Least-Privilege Chaining)

| Security Group | Inbound Rules | Source | Outbound Rules | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| `eazycat-prod-web-sg` | HTTP (80)<br>HTTPS (443) | `0.0.0.0/0` | All (`0.0.0.0/0`) | Entry point for public web clients. |
| `eazycat-prod-app-sg` | Custom TCP (8080) | `eazycat-prod-web-sg` | All (`0.0.0.0/0`) | Restricts backend access strictly to the web tier. |
| `eazycat-prod-db-sg` | MySQL/Aurora (3306) | `eazycat-prod-app-sg` | Local Only | Restricts database access strictly to application instances. |

---

## Architectural Decisions & Trade-Offs

* **Air-Gapped Persistence Tier:** Eliminating default routes (`0.0.0.0/0`) in `eazycat-prod-db-rt` ensures that even if database instances are compromised, automated exfiltration or command-and-control communication with the internet is impossible at the route layer.
* **Security Group Chaining over IP CIDRs:** By using parent Security Group IDs as sources in rule definitions, future Auto Scaling Groups (ASGs) can launch EC2 instances dynamically without requiring manual security group rule updates.
* **Single NAT Gateway (Cost vs. Redundancy):** A single Elastic IP NAT Gateway was placed in `us-east-1a` to minimize AWS hourly idle charges during initial staging, with route table structure ready to scale to Multi-AZ NAT Gateways for production fault tolerance.