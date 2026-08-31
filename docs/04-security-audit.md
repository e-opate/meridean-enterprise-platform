# Security Audit

A self-audit against the security requirements defined in `docs/00-business-requirements.md` (requirement #3: no shared credentials, encrypted data, no public access to compute/data by default).

## What I checked

### IAM & External Access
**Check:** AWS IAM Access Analyzer (resource analysis — external access), run against the full account.
**Result:** Zero findings. No resource in this account is shared or accessible from outside the account boundary.

*[Screenshot: Access Analyzer — no findings](../screenshots/access-analyzer-clean.png)*

### Network — Security Group Rules
**Check:** Manually reviewed inbound rules on all 3 security groups.
**Result:**
- `meridean-alb-sg` — HTTP/HTTPS open to `0.0.0.0/0` (expected — this is the public entry point)
- `meridean-app-sg` — inbound only from `meridean-alb-sg`, no direct public access
- `meridean-data-sg` — inbound only from `meridean-app-sg`, no direct public or app-bypassing access

No stray `0.0.0.0/0` rules found on the app or data tier security groups.

### Data — RDS
**Check:** RDS instance connectivity and encryption settings.
**Result:** Public access disabled. Encryption at rest enabled (default AWS KMS key). Matches requirement #3.

### Data — S3
**Check:** Account-level S3 Block Public Access settings.
**Result:** Not applicable — no S3 buckets were provisioned as part of this project's scope. Noted here explicitly so the absence reads as "considered, not applicable," rather than an oversight.

## Known Gaps

- **WAF Web ACL is untagged.** Encountered a console limitation applying tags to the WAFv2 resource via both its own management page and Tag Editor. Documented rather than left silent — in a Terraform-managed version of this project (planned next phase), this wouldn't be an issue, since tags are set declaratively at resource definition time.
- **RDS is Single-AZ, not Multi-AZ** — see `adr/ADR-003-single-az-rds.md`. An availability gap, not strictly a security one, but relevant to overall resilience posture.
- **No automated credential rotation** on the Secrets Manager secret — requires a Lambda function, not yet covered in this project's scope. Documented as a production enhancement.

## Summary

No unintended external exposure found. Network segmentation and data-tier isolation hold up under manual review. The gaps above are scoped and documented, not hidden — each has a clear path to resolution noted in its respective ADR or here.
