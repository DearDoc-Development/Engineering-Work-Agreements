# DearDoc — Organization-wide Claude Instructions

This file is shared across every DearDoc product repository. It captures the engineering agreements, methodology, and review standards that apply regardless of which repo or service is being edited. Repo-specific context lives in each repo's own `CLAUDE.md` and extends — never contradicts — these rules.

## How to consume this file

Each DearDoc repo should import this file via the `@` syntax at the top of its `CLAUDE.md`. Setup instructions are in [README.md](./README.md#claude-code-setup).

## Methodology: Cronos

We develop in one-week cycles using the Cronos framework. The full methodology is in [cronos.md](./cronos.md). The short version:

- **Monday — Initialization & Scaffolding.** Translate the Epic into an AI-ready plan. Scaffold structure, schemas, and core logic. The developer is **Mission Control**: review the plan before any code lands.
- **Tuesday–Wednesday — Vibe Coding.** Conversational iteration with mid-loop verification. Keep a mental model of the system — the AI does not.
- **Thursday — Validation & Automated QA.** Unit, integration, E2E, visual, and security scans. Generation stops; rigorous verification takes over.
- **Friday — Documentation, Polish, Peer Review.** Half-day review by someone other than the author. Release gated on the review outcome.

### Working implications for Claude

- Before any non-trivial implementation, present a plan and wait for explicit approval (the "Mission Control" gate).
- Prefer editing existing files; scaffold only when the cycle is in Initialization.
- Never mark a cycle task complete with failing tests, partial implementation, or unresolved errors.
- At the end of a cycle, update inline docs, READMEs, and API specs — documentation is a cycle deliverable, not an afterthought.
- Surface risks (auth, state, hallucinated APIs, edge cases) proactively, using the peer-review checklist in cronos.md.

## Source Control

- **Platform:** GitHub under [DearDoc-Development](https://github.com/DearDoc-Development).
- **Flow:** [GitHub Flow](https://githubflow.github.io/). `main` is always deployable. Feature branches cut from `main`, PR back into `main`.
- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Types: `feat`, `fix`, `docs`, `chore`, and `BREAKING CHANGE` footers for major releases.
- **Granularity:** commit often, in small chunks. Keep history readable.

## Project Management

- **Monday.com** is the source of truth for tasks, ownership, and status. Link PRs to Monday items where applicable.
- **Slack** for day-to-day communication; **Google Drive** for documents and reports.

## Testing & QA

See [Testing.md](./Testing.md). In every Cronos cycle, Thursday is reserved for verification — plan accordingly.

## Project Documentation

New projects follow [TechnicalDocumentation.md](./TechnicalDocumentation.md) as the README template. AI-assisted docs are expected to be generated as part of the Friday finalization step.

## Security & Data Handling

- Never hardcode credentials. Use environment variables and secret managers.
- TLS 1.2+ enforced on all endpoints.
- For PCI/PHI data: zero-persistence. Raw card data, CVVs, and expiry dates must never be stored in databases, logs, caches, or backups. Sensitive fields must be sanitized from logs.
- Follow the peer-review security checklist in `cronos.md` on every PR.

## Cloud & Infrastructure

We are bound by HIPAA and SOC 2. Full agreements in [CloudBestPractices.md](./CloudBestPractices.md). The rules that most often affect what Claude is asked to do:

- **Two tiers of platform.** AWS and GCP only for production and anything holding real patient (PHI) or customer data. Other platforms (Vercel, Railway, Supabase, Cloudflare) are prototypes only — never with real data, never with production credentials, and never calling production APIs.
- **Terraform, not the console.** Propose production infrastructure as code. Never suggest a console change to production outside an emergency.
- **Compliant by construction.** When adding a resource, the same change must include: encryption at rest and in transit, a dedicated least-privilege role, private-by-default networking, a `VantaOwner` tag, alarms wired to an existing notification channel, and explicit 365-day log retention. Use the per-resource baselines in CloudBestPractices.md — don't invent thresholds.
- **Never deploy without telemetry.** Do not treat a resource as done if it has no alarms.
- **Ticket first.** Infrastructure and access changes require an approved Monday.com item. Surface this rather than proceeding silently.

### Working implications for Claude

- Treat any new production resource, IAM change, or network-perimeter change as non-trivial: present the plan and wait for approval (Mission Control gate).
- Never write PHI, PII, card data, or secrets into logs, prompts, fixtures, or test data — synthetic data only.
- Never suggest disabling encryption, opening `0.0.0.0/0`, granting `allUsers`/`allAuthenticatedUsers`, attaching wildcard IAM policies, or using Owner/Editor roles on service accounts.
- If a compliance rule blocks the task, say so and propose the ticket-and-exception path instead of a workaround.

## Modifications

These agreements are a living document. Open a PR in [Engineering-Work-Agreements](https://github.com/DearDoc-Development/Engineering-Work-Agreements) to propose changes.
