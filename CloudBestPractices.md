# Cloud Best Practices

The purpose of this document is to provide a set of guidelines and best practices for creating and changing cloud resources at [DearDoc](https://getdeardoc.com/). It applies to **anyone** who provisions or modifies infrastructure — engineers and manager-builders alike.

This document is a living document and will be updated as the team grows and the company evolves.

DearDoc serves healthcare practices and is bound by **HIPAA** and **SOC 2**. That shapes the one idea everything here flows from:

> **Real patient and customer data only ever lives on platforms we control, monitor, and have compliance agreements with.**

The per-resource baselines in [Per-resource baselines](#per-resource-baselines) are derived from our Vanta test set, so that every new resource is **compliant by construction** — it passes Vanta on day one instead of generating findings to clean up later. Version 1.2 adds what the October–November 2026 remediation taught us: GCP projects and VMs, Cloud SQL, IAM custom roles, organization policies, alert hygiene, and a formal exception process.

## Table of Contents

- [Cloud Best Practices](#cloud-best-practices)
  - [Table of Contents](#table-of-contents)
  - [How this fits our other agreements](#how-this-fits-our-other-agreements)
  - [Approved platforms: the two-tier rule](#approved-platforms-the-two-tier-rule)
    - [Tier 1 — Production and anything with real customer data](#tier-1--production-and-anything-with-real-customer-data)
    - [Tier 2 — Prototypes, proofs of concept, and internal tools](#tier-2--prototypes-proofs-of-concept-and-internal-tools)
  - [Data classification quick reference](#data-classification-quick-reference)
  - [Creating a resource: the checklist](#creating-a-resource-the-checklist)
  - [Notification channels and alert hygiene](#notification-channels-and-alert-hygiene)
  - [Identity: custom roles, never predefined critical roles](#identity-custom-roles-never-predefined-critical-roles)
  - [Organization policies](#organization-policies)
  - [Per-resource baselines](#per-resource-baselines)
    - [AWS](#aws)
    - [GCP](#gcp)
    - [GitHub / SDLC](#github--sdlc)
  - [Security baseline](#security-baseline)
  - [Exceptions](#exceptions)
  - [Vanta test map](#vanta-test-map)
  - [Modifications](#modifications)

## How this fits our other agreements

Infrastructure work is not a separate track — it runs through the same gates as application code:

- **Source control.** Terraform changes follow [GitHub Flow and the PR mandatory steps](./README.md#source-control) like any other change. Infrastructure is a `feat`/`fix`/`chore` commit, reviewed by a human peer, never merged by bypassing branch protection.
- **Cronos cycles.** Provisioning belongs in Monday's initialization phase, so alarms and IAM land in the same cycle as the resource. Thursday's validation includes confirming the resource is green in Vanta. See [`cronos/`](./cronos).
- **Mission Control gate.** New production resources, IAM changes, and anything touching the network perimeter are non-trivial by definition: present the plan and get explicit approval before applying.
- **Documentation.** A new resource is documented in the owning repo's README per [TechnicalDocumentation.md](./TechnicalDocumentation.md) as part of the Friday finalization step.
- **Compliance is not optional.** Every rule below maps to a HIPAA or SOC 2 control we are audited against. If a rule blocks you, the answer is a ticket and a documented [exception](#exceptions) — never a silent workaround.
- **CLI fixes get codified.** A `gcloud`/`aws` command is fine to close a finding today; the same change lands in Terraform before the ticket closes, or the next `apply` reverts it and the finding returns.

## Approved platforms: the two-tier rule

### Tier 1 — Production and anything with real customer data

**AWS and Google Cloud Platform only.** These are the only environments covered by our Business Associate Agreements, our intrusion detection (GuardDuty, Cloud IDS), our centralized logging, and our Vanta compliance monitoring. Any system that stores, processes, or transmits real patient data (PHI) or real customer/practice data lives here. No exceptions, no "just temporarily."

### Tier 2 — Prototypes, proofs of concept, and internal tools

Other platforms (Vercel, Railway, Supabase, Cloudflare, etc.) are fine for experimentation and internal tooling, under these hard conditions:

- **No real patient or customer data. Ever.** Use invented/synthetic data only. This includes exports, screenshots of real records, and chat transcripts.
- **No production credentials.** A prototype never holds keys, tokens, or service accounts that reach production systems — and it doesn't call production APIs, even read-only. Point it at sandbox/staging or mock data.
- **Label it.** Name and describe it as non-production so nobody mistakes it for a supported system.
- **Graduation is a migration, not a promotion.** If a prototype proves useful, it moves into AWS/GCP and goes through the full [resource-creation checklist](#creating-a-resource-the-checklist) *before* it touches real data. There is no path where a Tier 2 system starts handling PHI in place.

**Tier 2 inside our clouds.** A test project in GCP or AWS is still inside the Vanta boundary: Vanta does not know it is "just a test." A test VM gets CrowdStrike, labels, and CPU monitoring like any other; a test project gets an owner, `environment=test`, a review date, and is deleted when the test ends. What a test project *may* skip — with a documented [exception](#exceptions) — is cost-disproportionate controls such as a Cloud IDS endpoint.

If you're unsure which tier something belongs to, the deciding question is: *could real patient or customer data ever flow through this?* If yes — or if you're not sure — it's Tier 1.

## Data classification quick reference

| Data type                   | Examples                                                                                     | Where it may live                                                                                    | Label value      |
|-----------------------------|----------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|------------------|
| **PHI**                     | Patient names, appointment requests, AI Smartchat transcripts, anything patient-identifiable | Tier 1 only. Encrypted at rest and in transit. Never in logs, prompts, spreadsheets, or screenshots. | `confidential`   |
| **Customer business data**  | Practice configurations, billing, contracts, leads                                           | Tier 1 only.                                                                                         | `confidential`   |
| **Internal company data**   | Our docs, internal metrics, non-customer content                                             | Tier 1 preferred; Tier 2 with judgment.                                                              | `internal`       |
| **Synthetic / public data** | Invented sample records, public docs                                                         | Anywhere.                                                                                            | `synthetic`      |

The last column is the value for the `dataclassification` label on GCP instances (see [GCP baselines](#gcp)).

## Creating a resource: the checklist

Every new production resource (database, function, bucket, service, queue) goes through all of these. Vanta continuously checks much of this — the goal is that a new resource comes up green on day one, not that we fix findings later.

01. **Ticket first.** The change is documented and approved in [Monday.com](https://monday.com/) before it happens (this is our SOC 2 change-management control).
02. **Infrastructure as Code.** Production resources are created via Terraform, not hand-built in the console. Console or CLI changes to production are for emergencies and compliance fixes only, documented after the fact, and codified before the ticket closes.
03. **Owner, tags, and inventory.** Every resource has a `VantaOwner` tag (AWS) or `vantaowner` label (GCP) set to an active employee's email, a description, and — if it holds customer data — is marked as containing user data in the Vanta inventory. Resources owned by leavers get reassigned during offboarding.
04. **Encryption on.** At rest (managed KMS is fine) and in transit (TLS 1.2+). This is the default on both clouds — the rule is: never turn it off. Note: at-rest encryption on RDS and DocumentDB can only be set **at creation**; retrofitting means a snapshot → encrypted copy → restore migration. Get it right on day one.
05. **Least-privilege IAM.** A dedicated service account or role per workload, scoped to exactly what it needs, using our [custom roles](#identity-custom-roles-never-predefined-critical-roles). No wildcard policies, no `allUsers`/`allAuthenticatedUsers` grants, no Owner/Editor on service accounts, no IAM policies attached directly to users (groups/roles only), no shared human accounts, no long-lived user keys where a role or workload identity works.
06. **Private by default.** No public buckets, no databases or SSH open to `0.0.0.0/0` or `::/0`, everything inside the VPC perimeter. The only ports open to the world are 80/443 on the public entry point (load balancer). Public exposure beyond that only when the ticket explicitly says so and says why.
07. **Telemetry before production.** Alarms are created **in the same Terraform change** as the resource, following the [per-resource baselines](#per-resource-baselines). *Never deploy without telemetry* — a resource with no alarms is not production-ready and will fail Vanta.
08. **Channel parity.** Every alarm attaches to an existing notification topic/channel per the [channel map](#notification-channels-and-alert-hygiene). An alert nobody receives is non-compliant — and we have found topics with zero subscribers, so verify, don't assume.
09. **Explicit log retention.** Log groups and log buckets are set to the **365-day** baseline explicitly — never left on defaults. Any deviation needs documented business approval.
10. **Verify in Vanta.** After creation, confirm the resource appears in inventory with owner and description and shows no failing tests. Fix or formally document any finding before moving on.

## Notification channels and alert hygiene

We route by domain into **existing** channels. Do not create new SNS topics or GCP notification channels without a ticket explaining why the map below doesn't fit.

| Domain                                                                   | Sink                                                                                             |
|--------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| AWS relational + DocumentDB (memory, CPU, storage, IO, reads/writes)     | `deardoc-rds-alerts` (SNS)                                                                       |
| AWS DynamoDB capacity                                                    | `dynamodb-capacity-alerts` (SNS)                                                                 |
| AWS serverless and queues (Lambda errors, SQS queue age)                 | `lambda-error-alerts` (SNS)                                                                      |
| AWS load balancers (5XX, latency, healthy/unhealthy hosts)               | `deardoc-alb-errors` (SNS)                                                                       |
| AWS security (GuardDuty findings)                                        | `deardoc-security-alerts` (SNS) — mirrors the GCP IDS channel so threats never drown in capacity noise |
| All GCP alert policies                                                   | The project's standard channels: `engineering@` email + the `#engineering_alerts` Slack channel  |

**Recipients.** Alerts go to the team, not to people. The standard recipient set is the engineering distribution list and the Slack channel; a project may add its operator (the person who owns a test VM, say) as an extra channel. Personal mailboxes of leadership are **not** on alert policies — that is how alert fatigue starts, and alert fatigue is how real alerts get missed.

**GCP note:** notification channels are **project-scoped**. The standing rule is *reuse the endpoint* (same recipients), which means each GCP project that carries alert policies gets its standard channel objects, created via Terraform alongside the project's first alert. The Slack channel type needs a one-time OAuth per project in the console; the Slack channel's inbound email address works without it.

**AWS note:** alarms send on `ALARM` only. `OKActions` stay empty — recovery is visible on the dashboard and doubling every notification trains people to ignore the topic.

**Thresholds and durations.** Vanta checks that a policy *exists*; we choose what it says. The standard is "sustained, not spiky": every threshold has a duration, and the number sits well above normal peaks.

| Alert                                   | Threshold                    | Sustained |
|-----------------------------------------|------------------------------|-----------|
| CPU utilization (Cloud SQL, GCE, RDS)   | > 90% (RDS keeps 80%)        | 15 min    |
| Memory / disk utilization (Cloud SQL)   | > 90%                        | 15 min    |
| Cloud SQL disk I/O                      | > 5,000 ops/s per instance   | 15 min    |
| Cloud Run / Cloud Functions errors      | > 100 per function           | 10 min    |
| Firestore document reads / writes       | > 500k / > 50k per 5 min     | 5 min     |
| Load balancer 5XX                       | > 10 per 5 min               | 5 min     |
| Unhealthy hosts                         | ≥ 1                          | 10 min    |

Tune a threshold from data, not from annoyance. When a function trips the error alert repeatedly, the fix is the function, not a bigger number.

Example — the standard GCP channels and a policy, in Terraform:

```hcl
resource "google_monitoring_notification_channel" "engineering" {
  display_name = "Engineering alerts"
  type         = "email"
  labels       = { email_address = "engineering@getdeardoc.com" }
}

resource "google_monitoring_alert_policy" "fn_errors" {
  display_name = "Cloud Run / Cloud Functions execution errors"
  combiner     = "OR"
  conditions {
    display_name = "Cloud Run 5xx responses (per service, 10 min)"
    condition_threshold {
      filter          = "metric.type=\"run.googleapis.com/request_count\" AND resource.type=\"cloud_run_revision\" AND metric.labels.response_code_class=\"5xx\""
      comparison      = "COMPARISON_GT"
      threshold_value = 100
      duration        = "600s"
      aggregations {
        alignment_period     = "300s"
        per_series_aligner   = "ALIGN_SUM"
        cross_series_reducer = "REDUCE_SUM"
        group_by_fields      = ["resource.labels.service_name"]
      }
    }
  }
  notification_channels = [google_monitoring_notification_channel.engineering.id]
  documentation {
    content   = "A Cloud Run service is failing. Check its logs and Sentry; if it started after a deploy, roll back. Owner: DearDoc Engineering."
    mime_type = "text/markdown"
  }
}
```

## Identity: custom roles, never predefined critical roles

Vanta flags the predefined roles `roles/owner`, `roles/editor`, `roles/iam.serviceAccountUser`, `roles/iam.serviceAccountAdmin`, and `roles/iam.serviceAccountTokenCreator` on any user-managed service account. Google also grants `roles/editor` to every project's default compute and App Engine service accounts automatically, which is how most of our findings appeared. The standard:

- **Deployers and serverless runtimes** (CI deploy accounts, Cloud Functions / Cloud Run identities) hold the organization custom role **`serverlessStackAdmin`**. It contains the permissions those workloads actually use (Functions, Run, Firebase, Firestore, Pub/Sub, Scheduler, Firebase Auth, logging, monitoring) and deliberately **excludes** token-creation (`iam.serviceAccounts.getAccessToken`, `signBlob`, `signJwt`, `implicitDelegation`), service-account and key creation, and any `setIamPolicy`. When a workload needs a permission the role lacks, add that permission to the role — never fall back to Editor.
- **Compute Engine VMs** run as a **dedicated service account** per VM holding a small project-level custom role (`vmRuntime`: log and metric writing) plus resource-level grants for whatever the workload touches (a specific secret, a specific bucket). Default compute service accounts hold nothing.
- **Default service accounts** (`<project>@appspot`, `<number>-compute@developer`) keep no predefined roles. Existing projects were migrated to the custom role; new projects never receive the grant, by [organization policy](#organization-policies).
- **Role hygiene.** A deploy role with token-creation permissions at project level lets a leaked CI key impersonate any service account in the project — the escalation Vanta's test exists to catch. Review custom roles for `iam.*`, `*.setIamPolicy`, and `secretmanager.versions.access` before granting them widely.

Migration order, always: add the custom role → run a deploy / exercise the workload → remove the predefined role → watch logs for `PERMISSION_DENIED` for a day. Back up each project's IAM policy first (`gcloud projects get-iam-policy P --format=json`); it is the one-command rollback.

## Organization policies

Two constraints stop the most common findings from recurring in every new project:

| Constraint                                              | Why                                                                                       |
|---------------------------------------------------------|-------------------------------------------------------------------------------------------|
| `iam.automaticIamGrantsForDefaultServiceAccounts`       | New projects no longer grant Editor to default service accounts                           |
| `compute.skipDefaultNetworkCreation`                    | New projects no longer get an auto-mode `default` VPC with 40+ subnets nobody uses         |

Both are enforced at the organization level and managed in Terraform (`google_org_policy_policy`). A new project that genuinely needs a network creates a custom-mode one, named for its purpose, with flow logs from day one.

## Per-resource baselines

When you create one of these, the listed items ship in the same change. This is what "done" means for infrastructure.

### AWS

**RDS / PostgreSQL instance**

- `storage_encrypted = true` (creation-time only)
- Automated daily backups, retention ≥ 7 days
- **Custom parameter group** (never `default.*`) with **pgaudit**: `pgaudit` appended to `shared_preload_libraries` (keep `pg_stat_statements`), `pgaudit.log = ddl,role,write`, `CREATE EXTENSION pgaudit` in each database. `shared_preload_libraries` is static — the change takes effect on reboot. For a new instance that is part of provisioning; for an existing one it is a scheduled window, ideally combined with any pending `system-update` maintenance so downtime is paid once.
- PostgreSQL log export to CloudWatch (`enabled_cloudwatch_logs_exports = ["postgresql"]`) with the log group at 365 days
- Security group: app subnets only, never public
- Alarms → `deardoc-rds-alerts`: `FreeableMemory` ≤ 1 GB · `FreeStorageSpace` ≤ 10 GB · `DiskQueueDepth` > 20 · `CPUUtilization` > 80% (5-min avg, 2 periods)

**Aurora PostgreSQL cluster** — as RDS, with the parameters on a custom **cluster** parameter group.

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

- Images come from an ECR repository with `image_tag_mutability = "IMMUTABLE"` and are tagged with the **git commit SHA**; nothing pushes `:latest`. Third-party sidecars use semantic version tags.
- The order matters when converting an existing service: pipeline tags with the SHA → task definition references the tag (`var.image_tag`) → one real deploy proves it → flip the repository to `IMMUTABLE` → codify. Flipping first breaks deploys; digest-pinning without changing the pipeline makes deploys silently stop. `IMMUTABLE_WITH_EXCLUSION` is not accepted.
- Security groups: explicit ports only, nothing public except 80/443 through the LB, no public SSH
- Task role scoped per service
- Deployment runbook for teams owning a Fargate service: *Remediation guide — version-controlled images* (Engineering wiki)

**Load balancer (ALB)**

- TLS 1.2+ security policy, valid ACM certificate (expired certs removed)
- Alarms → `deardoc-alb-errors`, named `ALB-<Metric>-<lb-name>`: `HTTPCode_Target_5XX_Count` > 10 / 5 min · `TargetResponseTime` > 1 s avg (2 periods) · `UnHealthyHostCount` ≥ 1 · `HealthyHostCount` < 1 (missing data = breaching)
- When a target group is recreated, re-point the host-count alarms at the new target group id — a stale dimension leaves the alarm permanently in `ALARM`

**SQS queue**

- Alarm → `lambda-error-alerts`: `ApproximateAgeOfOldestMessage` above a threshold that makes sense for the queue's SLA

**S3 bucket**

- Block Public Access on (account level is enabled; never carve out exceptions without a ticket)
- Default encryption (verify)
- **HTTPS-only** bucket policy — merged into any existing policy, never replacing it:

  ```json
  {
    "Sid": "DenyInsecureTransport",
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": ["arn:aws:s3:::BUCKET", "arn:aws:s3:::BUCKET/*"],
    "Condition": { "Bool": { "aws:SecureTransport": "false" } }
  }
  ```

  Buckets that serve web content directly by URL need their consumers on `https://` first; confirm from the server access logs (the `TLSVersion` field is `-` on plain-HTTP requests) before applying.
- User-data buckets: versioning on + server access logging to the central log bucket
- `VantaOwner` tag

**EC2 instance (if any)**

- CrowdStrike Falcon installed, SSM for access (no public SSH), CPU alarm → `lambda-error-alerts`, security group rules per checklist items 5–6

**VPC / networking**

- Flow logs enabled on every VPC
- GuardDuty enabled in every active region, findings ≥ high routed via EventBridge → `deardoc-security-alerts`
- CloudTrail: all-region trail with log file validation, log bucket access restricted

### GCP

**New GCP project (any)** — the project baseline; one Terraform module, applied on creation

- Connected to Vanta before it carries workloads; labeled `vantaowner`, `environment`
- Standard notification channels (engineering email + Slack); operator channels as needed
- Log bucket retention set to 365 days (`_Default` bucket)
- No `default` network (org policy); if a network is needed, custom-mode, flow logs on
- Default service accounts hold no predefined roles (org policy); workloads use custom roles
- **Cloud IDS notifications**: a logs-based counter metric `cloud_ids_high_critical_threats` on `ids.googleapis.com/threat` logs with `alert_severity = HIGH or CRITICAL`, plus an alert policy on it. Required by Vanta whether or not an IDS endpoint exists, and free.
- **Cloud Run / Cloud Functions error policy** (project-wide; see the example above) if the project will run functions or services
- **Firestore read and write frequency policies** if the project uses Firestore
- Projects named `sys-*` are Apps Script system projects and are excluded from inventory loops

**Compute Engine instance**

- Terraform-managed; `goog-terraform-provisioned` and `managed_by` labels present
- Five **required labels**, lowercase: `environment` (`production`/`staging`/`test`), `owner` (`first_getdeardoc_com`), `project`, `dataclassification` (`synthetic`/`internal`/`confidential`), `application`
- **CrowdStrike Falcon** installed via Terraform startup script and reporting; Vanta matches hosts on the GCP instance ID, so the sensor must register with it. Installer from a private bucket, CID from Secret Manager — never in the repo.
- Dedicated service account with the `vmRuntime` custom role; `scopes = ["cloud-platform"]`; `allow_stopping_for_update = true`
- CPU utilization policy (> 90%, 15 min) → standard channels
- Its network: flow logs on every subnet, firewall rule logging on, no `0.0.0.0/0` except 443, and either a Cloud IDS endpoint (production) or a documented [exception](#exceptions) (test)

**Cloud SQL instance**

- Automated backups daily, 7 retained, **point-in-time recovery** on (`backup_configuration { enabled = true, point_in_time_recovery_enabled = true }`); binary logging on MySQL
- **Automatic storage increase** on — the real fix for the free-storage problem
- Private IP where the workload allows; if private, the VPC it peers with is in scope for flow logs
- Four alert policies (project-wide on `cloudsql_database`): CPU > 90%, memory > 90%, disk > 90%, disk read/write ops above baseline — all 15-minute durations
- `gcloud sql` calls consume the API through the gcloud *quota project*: if a command fails with "Cloud SQL Admin API not enabled" on a project you did not name, unset `billing/quota_project`

**Firestore**

- Encryption: default (nothing to do, don't work around it)
- Alert policies → standard channels: `document/read_count` and `document/write_count` (frequency), filter restricted to `resource.type=one_of("firestore.googleapis.com/Database","firestore_instance")` — the API rejects a filter without a resource type

**Pub/Sub subscription**

- Alert policy → standard channels: `oldest_unacked_message_age` above the subscription's SLA

**Cloud Storage bucket**

- Versioning or a retention policy on user-data buckets; no `allUsers`/`allAuthenticatedUsers`

**Cloud Run service / Cloud Function**

- Covered by the project-wide error policy; runs as a service account holding `serverlessStackAdmin` (or narrower), never Editor
- Pub/Sub, Scheduler, Tasks, and Firebase Auth permissions the code needs are in the custom role; a `PERMISSION_DENIED` after a role change names the permission to add

**External HTTPS load balancer**

- SSL policy with `min_tls_version = "TLS_1_2"`, `profile = "MODERN"` attached to every target HTTPS proxy — with no policy attached, Google's default still accepts TLS 1.0

**Networking**

- Production networks: Cloud IDS endpoint attached, with a packet-mirroring policy on the subnets it inspects (an endpoint alone inspects nothing); high/critical threats routed via the logs-based metric above
- All networks: VPC flow logs (5-minute aggregation, 50% sampling, all metadata) on every subnet; firewall rule logging on; default-deny posture, no public port 22
- Log sinks: destinations tracked by Vanta, no public access grants
- Unused `default` networks are deleted, not monitored

### GitHub / SDLC

Applies to every repo. Complements the branching and PR rules in [README.md](./README.md#source-control).

- Repos private; branch protection on the default branch **including administrators** (`enforce_admins = true`); PR required, one approval, stale approvals dismissed, no force-push, no deletion; author ≠ reviewer enforced
- Dependabot vulnerability scanning on; remediation SLAs by severity (critical: 7 days, high: 30, medium: 90, low: next scheduled maintenance) with a named owner per repo
- Org invitations cancelled if older than 365 days
- Repositories outside the SOC 2 system boundary (internal tooling, CRM configuration, finance automations) are **deactivated in Vanta with a scope justification**, not left failing; dead repositories are **archived**, which removes them from scope automatically

## Security baseline

- **Intrusion detection stays on.** GuardDuty (AWS) and Cloud IDS (GCP) cover the production perimeter; high/critical findings route per the [channel map](#notification-channels-and-alert-hygiene) with no muting without a ticket. Host-level detection (CrowdStrike Falcon) runs on every VM.
- **Secrets live in secret managers** (GCP Secret Manager / AWS Secrets Manager), never in code, `.env` files in repos, Terraform state comments, chat threads, or logs.
- **Logs exclude PHI, PII, card data, and secrets.** No exceptions, including "temporary" debug logging. Sentry error reports must be scrubbed the same way, and Sentry must keep at least one active alert rule.
- **Centralize logging.** Application and infrastructure logs flow to Cloud Logging / CloudWatch — not to ad-hoc files or third-party sinks outside Tier 1.
- **MFA everywhere**: infrastructure providers, root accounts, and GitHub. Root accounts have no access keys and are break-glass only.
- **Access changes are tickets.** Granting, broadening, or transferring access goes through Monday.com; offboarding removes AWS, GCP, GitHub, and Sentry access and reassigns owned resources.
- **Error Reporting is not alerting.** Google Cloud Error Reporting notifies on *new* error groups; it does not see volume and does not satisfy the monitoring tests. Keep it — and keep the threshold policies beside it.

## Exceptions

Sometimes a control is disproportionate — a $1,000/month Cloud IDS endpoint for a single test VM. The answer is never to leave the finding failing or to fake the control; it is a **documented, time-boxed exception** with compensating controls, recorded as a deactivation in Vanta. The note must contain:

1. The resource, project, and owner
2. Why the environment is lower risk (non-production, synthetic data, no PHI, no production traffic)
3. Why the control is disproportionate (cost, impact)
4. The compensating controls that are **actually in place** — write the note after they are, not before (e.g. CrowdStrike reporting on the instance, flow logs and firewall logging on, restricted firewall rules, project monitoring)
5. A review date, and what ends the exception (project deletion, or graduation to Tier 1 which triggers the full control)
6. Who approved it

Exceptions are reviewed on their date. One that is still "temporary" at the second review becomes a ticket to implement the control or retire the resource.

## Vanta test map

Where to look when a test fails. Tests not listed are covered by the [checklist](#creating-a-resource-the-checklist) or are documents maintained outside this standard.

| Vanta test                                                       | Baseline item                                                           |
|------------------------------------------------------------------|-------------------------------------------------------------------------|
| Fargate deploys version-controlled images                        | ECS / Fargate — ECR `IMMUTABLE` + git-SHA tags                          |
| Load balancer latency / server errors / unhealthy host count     | ALB alarms → `deardoc-alb-errors`                                       |
| Serverless function error rate monitored (AWS)                   | Lambda error alarm (`for_each`)                                         |
| SQL database CPU / memory / storage / IO monitored (AWS)         | RDS alarms → `deardoc-rds-alerts`                                       |
| DocumentDB cluster memory / reads / writes monitored             | DocumentDB alarms                                                       |
| RDS for PostgreSQL / Aurora pgaudit enabled                      | RDS — custom parameter group with pgaudit                               |
| S3 buckets allow only HTTPS traffic                              | S3 — `DenyInsecureTransport` policy                                     |
| Server logs retained for 365 days (AWS)                          | Checklist item 9                                                        |
| Instance CPU monitored (GCP)                                     | Compute Engine — CPU policy                                             |
| GCP Compute Engine instances have required labels                | Compute Engine — five labels                                            |
| CrowdStrike is installed on all cloud instances                  | Compute Engine / EC2 — Falcon via Terraform                             |
| Critical IAM roles not granted to service accounts (GCP)         | [Identity](#identity-custom-roles-never-predefined-critical-roles)      |
| Intrusion detection system notifications configured (GCP)        | New project — IDS logs-based metric + policy                            |
| Intrusion detection system enabled (GCP)                         | Networking — IDS endpoint, or [exception](#exceptions)                  |
| VPC Flow Logs enabled (GCP)                                      | Networking — flow logs on every subnet, or delete the unused network    |
| NoSQL database read / write frequency monitored (GCP)            | Firestore — frequency policies                                          |
| SQL database CPU / memory / storage / I/O monitored (GCP)        | Cloud SQL — four policies                                               |
| Daily SQL database backups enabled (GCP)                         | Cloud SQL — backups + PITR                                              |
| Cloud Run functions have error monitoring configured (GCP)       | New project — error policy                                              |
| Load balancers enforce TLS 1.2 or higher (GCP)                   | External HTTPS LB — SSL policy                                          |
| Ensure branch protection rules are enforced for administrators   | GitHub — `enforce_admins`                                               |
| Critical / High / Medium / Low vulnerabilities addressed         | GitHub — Dependabot SLAs                                                |
| Messaging queue message age monitored                            | SQS / Pub/Sub age alarms                                                |

## Modifications

This document and every other file in this repository are living documents and will be updated as the team grows and the company evolves.

If you have any suggestions or want to propose a change, please open a pull request and submit it.
