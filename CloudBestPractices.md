# Cloud Best Practices

The purpose of this document is to provide a set of guidelines and best practices for creating and changing cloud resources at [DearDoc](https://getdeardoc.com/). It applies to **anyone** who provisions or modifies infrastructure — engineers and manager-builders alike.

This document is a living document and will be updated as the team grows and the company evolves.

DearDoc serves healthcare practices and is bound by **HIPAA** and **SOC 2**. That shapes the one idea everything here flows from:

> **Real patient and customer data only ever lives on platforms we control, monitor, and have compliance agreements with.**

The per-resource baselines in [Per-resource baselines](#per-resource-baselines) are derived from our Vanta test set, so that every new resource is **compliant by construction** — it passes Vanta on day one instead of generating findings to clean up later.

## Table of Contents

- [Cloud Best Practices](#cloud-best-practices)
  - [Table of Contents](#table-of-contents)
  - [How this fits our other agreements](#how-this-fits-our-other-agreements)
  - [Approved platforms: the two-tier rule](#approved-platforms-the-two-tier-rule)
    - [Tier 1 — Production and anything with real customer data](#tier-1--production-and-anything-with-real-customer-data)
    - [Tier 2 — Prototypes, proofs of concept, and internal tools](#tier-2--prototypes-proofs-of-concept-and-internal-tools)
  - [Data classification quick reference](#data-classification-quick-reference)
  - [Creating a resource: the checklist](#creating-a-resource-the-checklist)
  - [Notification channels: reuse, don't multiply](#notification-channels-reuse-dont-multiply)
  - [Per-resource baselines](#per-resource-baselines)
    - [AWS](#aws)
    - [GCP](#gcp)
    - [GitHub / SDLC](#github--sdlc)
  - [Security baseline](#security-baseline)
  - [Modifications](#modifications)

## How this fits our other agreements

Infrastructure work is not a separate track — it runs through the same gates as application code:

- **Source control.** Terraform changes follow [GitHub Flow and the PR mandatory steps](./README.md#source-control) like any other change. Infrastructure is a `feat`/`fix`/`chore` commit, reviewed by a human peer, never merged by bypassing branch protection.
- **Cronos cycles.** Provisioning belongs in Monday's initialization phase, so alarms and IAM land in the same cycle as the resource. Thursday's validation includes confirming the resource is green in Vanta. See [`cronos/`](./cronos/).
- **Mission Control gate.** New production resources, IAM changes, and anything touching the network perimeter are non-trivial by definition: present the plan and get explicit approval before applying.
- **Documentation.** A new resource is documented in the owning repo's README per [TechnicalDocumentation.md](./TechnicalDocumentation.md) as part of the Friday finalization step.
- **Compliance is not optional.** Every rule below maps to a HIPAA or SOC 2 control we are audited against. If a rule blocks you, the answer is a ticket and a documented exception — never a silent workaround.

## Approved platforms: the two-tier rule

### Tier 1 — Production and anything with real customer data

**AWS and Google Cloud Platform only.** These are the only environments covered by our Business Associate Agreements, our intrusion detection (GuardDuty, Cloud IDS), our centralized logging, and our Vanta compliance monitoring. Any system that stores, processes, or transmits real patient data (PHI) or real customer/practice data lives here. No exceptions, no "just temporarily."

### Tier 2 — Prototypes, proofs of concept, and internal tools

Other platforms (Vercel, Railway, Supabase, Cloudflare, etc.) are fine for experimentation and internal tooling, under these hard conditions:

- **No real patient or customer data. Ever.** Use invented/synthetic data only. This includes exports, screenshots of real records, and chat transcripts.
- **No production credentials.** A prototype never holds keys, tokens, or service accounts that reach production systems — and it doesn't call production APIs, even read-only. Point it at sandbox/staging or mock data.
- **Label it.** Name and describe it as non-production so nobody mistakes it for a supported system.
- **Graduation is a migration, not a promotion.** If a prototype proves useful, it moves into AWS/GCP and goes through the full [resource-creation checklist](#creating-a-resource-the-checklist) *before* it touches real data. There is no path where a Tier 2 system starts handling PHI in place.

If you're unsure which tier something belongs to, the deciding question is: *could real patient or customer data ever flow through this?* If yes — or if you're not sure — it's Tier 1.

## Data classification quick reference

| Data type | Examples | Where it may live |
|---|---|---|
| **PHI** | Patient names, appointment requests, AI Smartchat transcripts, anything patient-identifiable | Tier 1 only. Encrypted at rest and in transit. Never in logs, prompts, spreadsheets, or screenshots. |
| **Customer business data** | Practice configurations, billing, contracts, leads | Tier 1 only. |
| **Internal company data** | Our docs, internal metrics, non-customer content | Tier 1 preferred; Tier 2 with judgment. |
| **Synthetic / public data** | Invented sample records, public docs | Anywhere. |

## Creating a resource: the checklist

Every new production resource (database, function, bucket, service, queue) goes through all of these. Vanta continuously checks much of this — the goal is that a new resource comes up green on day one, not that we fix findings later.

1. **Ticket first.** The change is documented and approved in [Monday.com](https://monday.com/) before it happens (this is our SOC 2 change-management control).
2. **Infrastructure as Code.** Production resources are created via Terraform, not hand-built in the console. Console changes to production are for emergencies only, documented after the fact, and codified immediately.
3. **Owner, tags, and inventory.** Every resource has a `VantaOwner` tag set to an active employee's email, a description, and — if it holds customer data — is marked as containing user data in the Vanta inventory. Devices and resources assigned to leavers get reassigned during offboarding.
4. **Encryption on.** At rest (managed KMS is fine) and in transit (TLS 1.2+). This is the default on both clouds — the rule is: never turn it off. Note: at-rest encryption on RDS and DocumentDB can only be set **at creation**; retrofitting means a snapshot → encrypted copy → restore migration. Get it right on day one.
5. **Least-privilege IAM.** A dedicated service account or role per workload, scoped to exactly what it needs. No wildcard policies, no `allUsers`/`allAuthenticatedUsers` grants, no Owner/Editor roles on GCP service accounts, no IAM policies attached directly to users (groups/roles only), no shared human accounts, no long-lived user keys where a role or workload identity works.
6. **Private by default.** No public buckets, no databases or SSH open to `0.0.0.0/0` or `::/0`, everything inside the VPC perimeter. The only ports open to the world are 80/443 on the public entry point (load balancer). Public exposure beyond that only when the ticket explicitly says so and says why.
7. **Telemetry before production.** Alarms are created **in the same Terraform change** as the resource, following the [per-resource baselines](#per-resource-baselines). *Never deploy without telemetry* — a resource with no alarms is not production-ready and will fail Vanta.
8. **Channel parity.** Every alarm attaches to an existing notification topic/channel per the [reuse map](#notification-channels-reuse-dont-multiply). An alert nobody receives is non-compliant.
9. **Explicit log retention.** Log groups are set to the **365-day** baseline explicitly — never left on defaults. Any deviation needs documented business approval.
10. **Verify in Vanta.** After creation, confirm the resource appears in inventory with owner and description and shows no failing tests. Fix or formally document any finding before moving on.

## Notification channels: reuse, don't multiply

We route by domain into **existing** channels. Do not create new SNS topics or GCP notification channels without a ticket explaining why the map below doesn't fit.

| Domain | Sink |
|---|---|
| AWS relational + DocumentDB (memory, CPU, storage, IO, reads/writes) | `deardoc-rds-alerts` (SNS) |
| AWS DynamoDB capacity | `dynamodb-capacity-alerts` (SNS) |
| AWS serving path (Lambda errors, EC2 CPU, load balancers, SQS queue age) | `lambda-error-alerts` (SNS) |
| AWS security (GuardDuty findings) | `deardoc-security-alerts` (SNS) — the one sanctioned new topic, mirroring the GCP IDS channel, so threats never drown in capacity noise |
| All GCP alert policies (Firestore, Pub/Sub, IDS) | Standard GCP channel — same recipients as the Cloud IDS channel (engineering + on-call owners) |

**GCP note:** notification channels are **project-scoped**. The standing rule is *reuse the endpoint* (same recipients), which means each GCP project that carries alert policies gets one channel object pointing at the standard recipients, created via Terraform alongside the project's first alert.

## Per-resource baselines

When you create one of these, the listed items ship in the same change. This is what "done" means for infrastructure.

### AWS

**RDS / PostgreSQL instance**

- `storage_encrypted = true` (creation-time only)
- Automated daily backups, retention ≥ 7 days
- **pgaudit enabled**: custom parameter group with `pgaudit` in `shared_preload_libraries`, `pgaudit.log = write,ddl`, `CREATE EXTENSION pgaudit` per database (requires a reboot — schedule it as part of provisioning, before the instance carries traffic)
- Security group: app subnets only, never public
- Alarms → `deardoc-rds-alerts`: `FreeableMemory` ≤ 1 GB · `FreeStorageSpace` ≤ 10 GB · `DiskQueueDepth` > 20 · `CPUUtilization` > 80% (5-min avg, 2 periods)

**DocumentDB cluster**

- `storage_encrypted = true` (creation-time only)
- Daily backups, retention set explicitly
- Alarms → `deardoc-rds-alerts`: `FreeableMemory` ≤ 1 GB · `CPUUtilization` > 80% · `ReadThroughput` and `WriteThroughput` above a baseline you set (dimension: `DBClusterIdentifier`)

**DynamoDB table**

- Encryption: default (verify not overridden)
- `point_in_time_recovery { enabled = true }`
- Capacity alarms → `dynamodb-capacity-alerts`: consumed read/write capacity > 5,000 units (2 × 5 min, sum)

**Lambda function**

- Error alarm per function → `lambda-error-alerts`: `Errors` > 0 (5-min sum). Covered automatically by the `for_each` over `data.aws_lambda_functions` — confirm the new function appears after apply
- Log group retention set to 365 days explicitly

**ECS / Fargate service**

- Images built by CI from a version-controlled repo and deployed **by digest (SHA-256)** or from an ECR repo with tag immutability
- Security groups: explicit ports only (no "All traffic"), nothing public except 80/443 through the LB, no public SSH
- Task role scoped per service

**Load balancer (ALB)**

- TLS 1.2+ security policy, valid ACM certificate (expired certs removed)
- Alarms → `lambda-error-alerts`: `HTTPCode_ELB_5XX_Count` · `TargetResponseTime` · `UnHealthyHostCount` > 0

**SQS queue**

- Alarm → `lambda-error-alerts`: `ApproximateAgeOfOldestMessage` above a threshold that makes sense for the queue's SLA

**S3 bucket**

- Block Public Access on (account level is enabled; never carve out exceptions without a ticket)
- Default encryption (verify), bucket policy denying `aws:SecureTransport = false`
- User-data buckets: versioning on + server access logging to the dedicated log bucket
- `VantaOwner` tag

**EC2 instance (if any)**

- CrowdStrike Falcon installed, SSM for access (no public SSH), CPU alarm → `lambda-error-alerts`, security group rules per checklist items 5–6

**VPC / networking**

- Flow logs enabled on every VPC
- GuardDuty enabled in every active region, findings ≥ high routed via EventBridge → `deardoc-security-alerts`
- CloudTrail: all-region trail with log file validation, log bucket access restricted

### GCP

**New GCP project (any)**

- Connected to Vanta before it carries workloads
- One standard notification channel object ([section recipients](#notification-channels-reuse-dont-multiply))
- Log bucket retention set to 365 days
- Service accounts per workload; no universal grants; no Owner/Editor on service accounts

**Firestore**

- Encryption: default (nothing to do, don't work around it)
- Alert policies → standard channel: `document/read_count` and `document/write_count` (frequency) plus read/write **sizes** (volume) where the project handles material traffic. Distribution metrics always use explicit `ALIGN_SUM`/`ALIGN_RATE` aligners and `REDUCE_SUM` reducers

**Pub/Sub subscription**

- Alert policy → standard channel: `oldest_unacked_message_age` above the subscription's SLA

**Cloud Storage bucket**

- Versioning or a retention policy on user-data buckets; no `allUsers`/`allAuthenticatedUsers`

**Networking**

- Cloud IDS endpoint attached to every production network, high/critical threats routed to the standard channel
- VPC flow logs enabled; default firewall rules removed or restricted (default-deny posture)
- Log sinks: destinations tracked by Vanta, no public access grants

### GitHub / SDLC

Applies to every repo. Complements the branching and PR rules in [README.md](./README.md#source-control).

- Repos private; branch protection on `main` **including administrators**; author ≠ reviewer enforced
- Dependabot vulnerability scanning on; remediation SLAs by severity (critical/high fast, medium/low scheduled) with a named owner
- Org invitations cancelled if older than 365 days

## Security baseline

- **Intrusion detection stays on.** GuardDuty (AWS) and Cloud IDS (GCP) cover the production perimeter; high/critical findings route per the [channel map](#notification-channels-reuse-dont-multiply) with no muting without a ticket.
- **Secrets live in secret managers** (GCP Secret Manager / AWS Secrets Manager), never in code, `.env` files in repos, Terraform state comments, or logs.
- **Logs exclude PHI, PII, card data, and secrets.** No exceptions, including "temporary" debug logging. Sentry error reports must be scrubbed the same way, and Sentry must keep at least one active alert rule.
- **Centralize logging.** Application and infrastructure logs flow to Cloud Logging / CloudWatch — not to ad-hoc files or third-party sinks outside Tier 1.
- **MFA everywhere**: infrastructure providers, root accounts, and GitHub. Root accounts have no access keys and are break-glass only.
- **Access changes are tickets.** Granting, broadening, or transferring access goes through Monday.com; offboarding removes AWS, GCP, GitHub, and Sentry access and reassigns owned resources.

## Modifications

This document and every other file in this repository are living documents and will be updated as the team grows and the company evolves.

If you have any suggestions or want to propose a change, please open a pull request and submit it.
