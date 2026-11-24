# Engineering Work Agreements

The purpose of this repository is to provide a set of guidelines and best practices for the development of software projects at [DearDoc](https://getdeardoc.com/).

This document is a living document and will be updated as the team grows and the company evolves.

**The [release protocol](#release-protocol) is the rule for tickets, branching, review, testing, release and rollback. It replaces any older rule in this repository.**

## Table of Contents

- [Engineering Work Agreements](#engineering-work-agreements)
  - [Table of Contents](#table-of-contents)
  - [Project Management](#project-management)
    - [Agile](#agile)
    - [Cronos](#cronos)
    - [Tools](#tools)
  - [People](#people)
  - [Release protocol](#release-protocol)
    - [The five rules](#the-five-rules)
    - [Roles](#roles)
    - [The route of a change](#the-route-of-a-change)
    - [Restricted areas](#restricted-areas)
    - [Full documents](#full-documents)
  - [Source Control](#source-control)
    - [Branching](#branching)
    - [Commits](#commits)
    - [PR and Deploy Mandatory Steps](#pr-and-deploy-mandatory-steps)
      - [Team commitments](#team-commitments)
      - [Versioning](#versioning)
      - [Mandatory steps](#mandatory-steps)
      - [Rollback, urgent fix and break-glass](#rollback-urgent-fix-and-break-glass)
  - [Cloud and Infrastructure](#cloud-and-infrastructure)
  - [Claude Code Setup](#claude-code-setup)
  - [Project Documentation](#project-documentation)
  - [Modifications](#modifications)

## Project Management

This section outlines key practices for effective project management at DearDoc.

### Agile

We predominantly adopt [Agile methodologies](https://agilemanifesto.org/) to manage our work, adapting our approach based on team preferences and project demands.

At DearDoc, we primarily utilize two Agile methodologies: [Scrum](https://www.scrum.org/) and [Kanban](https://www.atlassian.com/agile/kanban). The choice between Scrum and Kanban depends on the specific requirements of the project as well as team dynamics.

**Key Differences**:

- **Scrum:** Operates on fixed-duration cycles known as sprints, with defined roles (Product Owner, Scrum Master, Development Team) and ceremonies (Sprint Planning, Daily Standup, Sprint Review, Sprint Retrospective). Scrum focuses on regular cadences and defined deliverables.
- **Kanban:** Focuses on continuous workflow with no set time frames for iterations. It emphasizes ongoing delivery without predefined roles or formal ceremonies.

So, while Scrum is more structured and prescriptive, Kanban is more flexible and adaptive, useful for a continuous task such as maintenance or projects developed by individual teams. The choice between the two depends on the project requirements and team dynamics.

### Cronos

For feature development we use one-week **Cronos cycles** — a structured methodology for AI-augmented engineering. Cronos is the build cadence: it organises the work. Code ships through the [daily release](#release-protocol), not at the end of the cycle. The full framework documentation lives in [`cronos/`](./cronos/).

**Quick reference:**

| Day | Phase | Focus |
|-----|-------|-------|
| Monday | Initialization | Tickets, file-level plans with rollback, architecture decisions |
| Tue–Wed | Vibe Coding | Iterative AI-assisted implementation |
| Thursday | Validation | Automated testing, security scans, QA on staging |
| Friday | Finalization | Documentation, polish, cycle review |

**Key rules:**

- Every change has a file-level plan, with its rollback, in the Monday ticket. A `#fix` needs three lines: the change, the expected result, the rollback.
- Approval before code is needed only for [restricted areas](#restricted-areas), and the approver is a Code Owner.
- Friday is a cycle review, not a ship or hold decision.
- A team may ask someone to validate risky work. This is optional and is not a release gate.
- Cycles can span multiple repos. Coordinate via a shared Epic in Monday.com. Each PR maps to one ticket.

See [`cronos/README.md`](./cronos/README.md) for the full methodology, roles, and requirements.

### Tools

- **[Slack](https://slack.com/):** Our primary tool for team communication.
- **[Monday.com](https://monday.com/):** Utilized for managing projects and facilitating collaboration. It allows teams to create task boards, assign responsibilities, track progress, and integrate with other tools, effectively supporting both Scrum and Kanban methodologies.
- **[Google Drive](https://drive.google.com/):** Used for storing and sharing project documentation, reports, and other files.

Monday.com is integral to project management at our company, providing a customizable platform suitable for diverse project needs. So, everything regarding the project management should be handled through Monday.com.

## People

Agile methodologies guide us in defining roles and responsibilities within project teams, emphasizing clarity but not hierarchy.

Regarding people management, every team member should have a touching point with another team member in the company, usually a team leader or a project manager. Some of the shared responsibilities are:

- **Role Clarity:** Understanding individual roles within the team.
- **Alignment with Company Vision:** Awareness of the company’s vision and mission.
- **Responsibilities:** Clear definition of what is expected from each team member.
- **Career Growth:** Opportunities for professional development including training and career paths.
- **Goal-Oriented Performance:** Focus on achieving specific, measurable goals.
- **Feedback and Evaluation:** Regular coaching and feedback sessions to foster growth and improvement.
- **Holidays and Time Off:** Clear guidelines on holidays, time off, and work-life balance.

**Development is fostered through**:

- **Regular 1-on-1s:** Scheduled weekly and quarterly, these sessions are dedicated to personal development and are distinct from regular project management activities.

**Tools**:

Our tool for handling personal development is [Ninety](https://www.ninety.io/). Using the [EOS](https://www.eosworldwide.com/) methodology, we run meetings with a clear agenda and goals. Manage Rocks and To-Dos, and have a clear view of the company's vision and mission. This tool is also helpful for evaluations, with tools such as [People Analyzer](https://www.eosworldwide.com/people).

## Release protocol

Owner: Project Manager and QA Lead (named on the [release protocol page](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol.md)). Last reviewed: 2026-11-24.

One track, two checkpoints. It applies to every repo that deploys to customers. Where it differs from an older rule in this repository, the protocol wins. Outside contributors should start with the [Contributor Guide](./ContributorGuide.md).

**The route in short:** ticket → plan → branch from `staging` → PR #1 to `staging` with the template, under 2,000 changed lines → CI and the Copilot verdict (Checkpoint 1) → staging check → release PR from `staging` to `main`, opened by the Release Captain, checked by an engineer with no change in it (Checkpoint 2) → daily release → message and verify within 30 minutes → rollback if it fails.

### The five rules

1. **No ticket, no pull request.**
2. **Nobody pushes straight to `staging` or `main`,** and nobody deploys from a laptop. One interim manual step exists today: the backend rules deploy (a backend's Firestore and Storage rules), run by the Release Captain as part of the release. See [In force today](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol.md#in-force-today).
3. **Nobody ships their own change alone.** AI reviews the code; a second person authorizes the release.
4. **No real patient data outside production.** Not in staging, tickets, PRs, screenshots, test files, or AI tools.
5. **Data changes can't be rolled back.** Add first, remove in a later release.

### Roles

| Role | Can |
|---|---|
| **Contributor** | Open PRs, merge to `staging` once approved. No production access |
| **Core Engineer and Code Owner** | Everything a contributor can, plus release checks and approvals in restricted areas |
| **Sponsor** | A core engineer who reads a contributor's code and checks their change in production with them |
| **Release Captain** | Runs the daily release, owns rollback calls, does the weekly sample |
| **Tech Lead** | Reviews contributor access each quarter and, with Product, sets the sprint objective |

### The route of a change

1. **Ticket.** A Monday ticket with requirements, expected output, one owner, and a target date. A contributor's ticket also names the sponsor. Tag it `#fix` only for a fix. A `#fix` is high priority: it must be in production within 8 hours, or by the end of the day.
2. **Plan.** Before any code, write a file-level plan with its rollback in the ticket. An engineer approves it first only when the change touches a [restricted area](#restricted-areas); the approver is then a Code Owner. A `#fix` needs three lines: the change, the expected result, the rollback.
3. **Build.** In repos with `staging`, branch from `staging`. Other repos follow their own README or CLAUDE.md. Name the branch `<type>/<ticket-id>-<slug>` (for example `fix/1234567890-phone-validation`). Open **PR #1 to `staging`** using the template, with every section filled in.
4. **Checkpoint 1: AI code review.** CI runs (typecheck, lint, tests, build, secret scan) and Copilot reviews with the DearDoc instructions.

   | Copilot's verdict | What happens |
   |---|---|
   | Clean or Low | Merge to `staging` |
   | Medium | Fix it, or answer it in the PR with a reason. Then merge |
   | High | Fix it. Copilot re-reviews, and a teammate must approve |
   | Restricted area | A Code Owner must approve |

   If an outside contributor's change touches logic (anything beyond copy, styling, layout, tests, or docs), the sponsor reads the code and approves before it merges. A `#fix` has one human review, by the engineer who merges the release to `main`, and that engineer reads the code.
5. **Staging.** Merging deploys to staging, and E2E tests run after every merge. Until the testing agent exists, the author (the sponsor, for a contributor) checks by hand against the ticket and records what they saw in the PR. If it passes, the change waits on `staging` for the next release. If it fails, it is reverted from `staging` straight away, and the author fixes it on a new branch from `staging` with a new PR.
6. **Checkpoint 2: release check.** The Release Captain opens a release PR from `staging` to `main`. An engineer with no change of their own in the release checks that:
   - every change matches its ticket, and nothing else is in it;
   - each change was approved by Copilot or a person in its place, no High finding is open, and restricted areas have Code Owner approval;
   - E2E passed on the exact staging commit being released;
   - each change has its rollback written, and data changes add first, remove later;
   - for a contributor's logic change, a core engineer has read the code: the sponsor, or for a `#fix`, you.

   Then they approve and merge. If every core engineer has a change in the release, two of them approve.
7. **Deploy.** The daily release runs **Monday to Thursday at 16:00 Madrid**. There are no Friday deploys except fixes and rollbacks. Each release is tagged `vYYYY.MM.DD.N` (version tags are not live yet; they are being set up).
8. **Message and verify.** After every deploy, each committer and sponsor with a change in it gets a message with the version tag and tickets. Within 30 minutes each committer confirms the change in production and records it on the ticket. Contributors verify through the UI, and their sponsor confirms.

**If it fails, roll back.** See [Rollback, urgent fix and break-glass](#rollback-urgent-fix-and-break-glass).

### Restricted areas

These always need an approved plan, Code Owner approval on both PRs, and the Release Captain present at deploy. Copilot does not approve them.

- Login and permissions, `firestore.rules`, `storage.rules`
- Secrets handling
- Where patient data is sent
- Payments and billing
- Database migrations
- Deploy and CI configuration: `.github/`, `firebase.json`, `.firebaserc`, Terraform, IAM
- AI review and agent instructions: `CLAUDE.md`, `AGENTS.md`, `REVIEW.md`, `*.instructions.md`
- New third-party services, SDKs, AI tools, or runtime packages
- The AI chat's prompts, model, tools, or what it says to patients
- Telephony (Twilio, Bandwidth)

### Full documents

- [Release protocol: core](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol.md) (private wiki; needs access to the DearDoc-Development organization)
- [Release protocol: full reference](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol-full.md) (private wiki)
- [Contributor Guide](./ContributorGuide.md): one page for outside contributors

## Source Control

This chapter describes how source control works with the release protocol: branching, commits, and pull requests.

All the code should be stored in a source control system. We will use [Github](github.com) as our source control system.

Every project should be created under our company Github account, [DearDoc](https://github.com/DearDoc-Development).

### Branching

When working in a project it is important to follow a branching strategy that allows for a clean and organized codebase.
We follow the [route of a change](#the-route-of-a-change) in the release protocol:

- `staging` is always releasable, because every release ships all of it. `main` is production.
- In repos with `staging`, create every branch from `staging`. Other repos follow their own README or CLAUDE.md. Name branches `<type>/<ticket-id>-<slug>` (for example `feat/1234567890-new-button`). The `<type>` uses the commit types below.
- Commit to that branch and push it regularly.
- Open a pull request to `staging` using the template. Keep it under 2,000 changed lines.
- Merging to `staging` deploys to staging. After the staging check passes, the change waits for the daily release.
- The Release Captain opens the release PR from `staging` to `main`. An engineer with no change in it checks and merges it. Merging deploys to production.
- Some protections are in force today and some are being set up. See [In force today](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol.md#in-force-today) for which is which. Until a control exists, follow the rule by hand (see [In force today](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol.md#in-force-today)).
- Admins can't bypass `main` today. Code Owner review is being set up. Branch protection applies to everyone, including administrators. The only way around it is [break-glass](#rollback-urgent-fix-and-break-glass).
- Customer-facing repos must have a `staging` branch and environment. A repo without one must add it. Until then it is an exception, tracked with a ticket. See [In force today](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol.md#in-force-today).
- Delete the feature or fix branch after it is merged.
- `vanta_production` is not covered by the release protocol. Ask the Tech Lead how it applies to your repo before relying on it.

### Commits

When pushing changes to a repository we should follow some guidelines on how to write the commit messages and the granularity of them.

Regarding the periodicity of commits, we should commit often and in small chunks. This will allow for a cleaner history and will make it easier to understand the changes made to the codebase.

Regarding the commit messages, we should follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification. In this way we should have a semantic way of writing commit messages that will allow for automatic changelog generation and other benefits.

Every branch and every PR carries a Monday ticket ID. A ticket is mandatory for each PR.

The structure of a commit should have this structure:

```md
<type>[optional scope]: <description>

[optional body]
[Monday ticket ID - Format: "Item Name #123456" or just "#123456"]

[optional footer(s)]
```

The types of commits are:

- `BREAKING CHANGE`: a commit that has a footer `BREAKING CHANGE: <description>`. Describe the change and its rollback plan in the ticket. Releases are tagged `vYYYY.MM.DD.N`, so a breaking change does not change the version number.
- `feat`: a new feature for the user, not a new feature for build script
- `fix`: a bug fix for the user, not a fix to a build script
- `docs`: changes to the documentation
- `chore`: others changes that don't modify src or test files

Example:

```md
feat: new API endpoint to get user data, profile picture and email

Includes a database migration with a new model in the backend to handle the user data and a new endpoint to get the user data, profile picture and email.
```

### PR and Deploy Mandatory Steps

#### Team commitments

- `24h review`: if you are named as a reviewer, you must respond or hand the review to someone else within 24 hours.
- `Monday link`: every PR links its Monday ticket. Write the item number in the PR title or description (see the commit example). No ticket, no PR.
- `SOC2 Compliance`: branch protection is never disabled to speed up a deployment, except through [break-glass](#rollback-urgent-fix-and-break-glass).

#### Versioning

Every production deploy is meant to be tagged `vYYYY.MM.DD.N`. Version tags are not live yet; they are being set up. Commit types do not set the version.

#### Mandatory steps

Every change follows the [route of a change](#the-route-of-a-change): plan in the ticket, PR to `staging`, Checkpoint 1 (CI and the Copilot verdict), staging check, then the release PR to `main` (Checkpoint 2) and the daily release.

- Required checks: the CI checks must pass on every PR.
- The author, or the sponsor for a contributor, records the staging check in the PR. There is no separate QA reviewer role.
- Copilot's approval never counts on `main`. The release check on `main` is done by an engineer with no change in the release.
- **Restricted areas** (the [list above](#restricted-areas)) need an approved plan, Code Owner approval on both PRs, and the Release Captain present at deploy. Copilot does not approve them, and does not approve while a High finding is open.

#### Rollback, urgent fix and break-glass

1. **Rollback.** Any core engineer redeploys the previous version tag with the `rollback` workflow (being built; until it exists, the Release Captain decides how to roll back). No approval is needed. Roll back first, investigate second. Then revert the change that caused it on `staging`, or the next release ships it again.
2. **Urgent fix.** Use the `#fix` route: a ticket tagged `#fix` with three lines (the change, the expected result, the rollback), one human review by the engineer who merges the release, and release as soon as it passes E2E. It must be in production within 8 hours or by the end of the day.
3. **Break-glass.** Admin Bypass is disabled to meet SOC 2 requirements. Break-glass also covers urgent compliance fixes made in the console. They must be codified afterwards. For critical production outages (P0) where the standard flow is too slow, alert the #engineer Slack channel. If a branch protection rule must be temporarily modified by an Org Admin to resolve a catastrophic failure (extreme cases only):
   - Document the change in a Monday.com ticket.
   - Re-enable the rule immediately after the fix.
   - Hold a post-mortem to explain why the bypass was necessary, for the compliance audit log.
   - Name every use at the next standup.

## Cloud and Infrastructure

DearDoc serves healthcare practices and is bound by **HIPAA** and **SOC 2**, so cloud resources have their own set of agreements. They apply to anyone creating or changing infrastructure, not only to engineers. The full document is [Cloud Best Practices](./CloudBestPractices.md).

**Quick reference:**

- **Two tiers.** AWS and GCP only for production and anything holding real patient or customer data — those are the platforms covered by our BAAs, intrusion detection, centralized logging, and Vanta monitoring. Other platforms (Vercel, Railway, Supabase, etc.) are for prototypes and internal tools, with **no real data and no production credentials, ever**.
- **Ticket first.** Every resource change is approved in Monday.com before it happens. This is our SOC 2 change-management control.
- **Terraform, not the console.** Production resources are created as code. Console changes are for emergencies, documented after the fact and codified immediately.
- **Compliant by construction.** Encryption, least-privilege IAM, private-by-default networking, an owner tag, alarms, and explicit log retention ship in the *same change* as the resource — not as a follow-up cleanup.
- **Never deploy without telemetry.** A resource with no alarms is not production-ready and will fail Vanta. Alarms route into the existing notification channels; don't create new ones without a ticket.

If a compliance rule blocks your work, the answer is a ticket and a documented exception — never a silent workaround.

## Claude Code Setup

The Claude Code setup lives in the private [ai-toolkit](https://github.com/DearDoc-Development/ai-toolkit) repository. You need access to the DearDoc-Development organization to open it.

- [Org-wide CLAUDE.md](https://github.com/DearDoc-Development/ai-toolkit/blob/main/agents/org-claude.md): the instructions every DearDoc repository shares. [How to import it](https://github.com/DearDoc-Development/ai-toolkit/blob/main/skills/INTEGRATION.md#importing-the-org-wide-claudemd).
- [Claude Code guardrails](https://github.com/DearDoc-Development/ai-toolkit/blob/main/knowledge/claude-code-guardrails.md): how every engineer sets up Claude Code on their machine.

- **Approved tools only, no real patient data.** Use only approved AI tools. Never put real patient data in a prompt. Say in the PR how much of the change was AI-generated.

## Project Documentation

To maintain a standard structure while developing new projects, it is recommended to use the [Technical Documentation](./TechnicalDocumentation.md) file as a template for your project's README.md file. When a change alters the architecture, update the Architecture document in the same PR.

## Modifications

This document and every other file in this repository are living documents and will be updated as the team grows and the company evolves.

If you have any suggestions or want to propose a change, please open a pull request and submit it.
