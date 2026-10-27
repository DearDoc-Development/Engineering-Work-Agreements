# Engineering Work Agreements

The purpose of this repository is to provide a set of guidelines and best practices for the development of software projects at [DearDoc](https://getdeardoc.com/).

This document is a living document and will be updated as the team grows and the company evolves.

## Table of Contents

- [Engineering Work Agreements](#engineering-work-agreements)
  - [Table of Contents](#table-of-contents)
  - [Project Management](#project-management)
    - [Agile](#agile)
    - [Cronos](#cronos)
    - [Tools](#tools)
  - [People](#people)
  - [Source Control](#source-control)
    - [Branching](#branching)
    - [Commits](#commits)
    - [PR and Deploy Mandatory Steps](#pr-and-deploy-mandatory-steps)
      - [Team commitments](#team-commitments)
      - [Versioning Definitions](#versioning-definitions)
      - [Mandatory steps](#mandatory-steps)
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

For feature development we use one-week **Cronos cycles** — a structured methodology for human-validated, AI-augmented engineering. The full framework documentation lives in [`cronos/`](./cronos/).

**Quick reference:**

| Day | Phase | Focus |
|-----|-------|-------|
| Monday | Initialization | Plan review, scaffolding, architecture decisions |
| Tue–Wed | Vibe Coding | Iterative AI-assisted implementation |
| Thursday | Validation | Automated testing, security scans, QA |
| Friday | Finalization | Documentation, polish, peer review |

**Key rules:**

- Every cycle includes a **mandatory half-day peer review** by someone other than the author.
- For any non-trivial change (new feature, schema change, API surface, cross-repo integration): **present the plan, wait for explicit approval, then implement**.
- Bug fixes and small refactors do not need the approval gate unless they cross repo boundaries.
- Cycles can span multiple repos — coordinate via a shared Epic and Monday.com item.

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

## Source Control

This chapter will describe the best practices for source control, including branching, commits, and pull requests.

All the code should be stored in a source control system. We will use [Github](github.com) as our source control system.

Every project should be created under our company Github account, [DearDoc](https://github.com/DearDoc-Development).

### Branching

When working in a project it is important to follow a branching strategy that allows for a clean and organized codebase.
We will use [Github Flow](https://githubflow.github.io/) as our branching strategy. This strategy is simple and effective, and it is based on the following principles:

- Anything in the `main` branch is always deployable.
- To work on something new, create a descriptively named branch off of `main` (ie: `new-oauth2-scopes`).
- Commit to that branch locally and regularly push your work to the same named branch on the server.
- When you need feedback or help, or you think the branch is ready for merging, open a pull request to staging if available.
- `Staging Environment`: If a Staging environment exists, it must be used for verification before merging to `main` (Production) following the instructions bellow.
- `No Staging Environment`: If no Staging environment exists, all mandatory checks and reviews must be performed before merging directly to main.
- Mandatory Protection: The `main` branch (and/or vanta_production) is a protected branch.
- No Admin Bypass: Branch protection settings apply to everyone, including administrators. No one can push directly to main or merge without meeting approval requirements.
- Review: Every merge needs an approving review from someone other than the author. On `staging`, that review can come from Copilot code review in repositories where Copilot approvals are enabled (see the note under Mandatory steps). Production branches always need at least one human peer approval: Copilot's approval never merges to production on its own.
- Once it is merged and pushed to `main`, you can and should deploy immediately.
- After checking the main deploy delete the feature/fix branch to keep the repo clean

### Commits

When pushing changes to a repository we should follow some guidelines on how to write the commit messages and the granularity of them.

Regarding the periodicity of commits, we should commit often and in small chunks. This will allow for a cleaner history and will make it easier to understand the changes made to the codebase.

Regarding the commit messages, we should follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification. In this way we should have a semantic way of writing commit messages that will allow for automatic changelog generation and other benefits.

The structure of a commit should have this structure:

```md
<type>[optional scope]: <description>

[optional body]
[optional Monday Ticket if related with any task or support issue - Format: "Item Name #123456" or just "#123456"]

[optional footer(s)]
```

The types of commits are:

- `BREAKING CHANGE`: a commit that has a footer `BREAKING CHANGE: <description>` will be part of the next major release.
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

- `24h review`: if one is mentioned as a reviwer then in less than 24 hours he must respond or designate another person for the job.
- `Monday link`: If the PR is related with a ticket in Monday Dev platform the number of the item must be writen in your GitHub PR title or description (check commit example).
- `SOC2 Compliance`: Under no circumstances should branch protections be disabled to speed up a deployment.

#### Versioning Definitions

- `Fix/small PR`: Treated as a minor version.
- `Feature`: Treated as a major version.

#### Mandatory steps

1. **Fix** - `fix/docs/chore` - Excepting Emergency Hotfix (below)
     - Step 1: Localhost - Develop and test changes locally.
     - Step 2: PR to Staging
       - **Required Reviews:** 1 Peer Reviewer + Copilot Review.
       - **Required Checks:** CI/CD (GitHub Action checks) must pass.
     - Step 3: PR to Production (main)
       - **Required Reviews:** 1 Peer Reviewer + Copilot Review.
       - **Required Checks:** CI/CD (GitHub Action checks) must pass.

2. **Small PRs group (Minor Version)** - `feat/s` (small)
      - Step 1: Localhost - Develop and test changes locally.
      - Step 2: PR to Staging
        - **Required Reviews:** 1 Peer Reviewer + Copilot Review + QA Reviewer.
        - **Required Checks:** CI/CD (GitHub Action checks) must pass.
      - Step 3: PR to Production (main)
        - **Required Reviews:** 1 Peer Reviewer + Copilot Review.
        - **Required Checks:** CI/CD (GitHub Action checks) must pass.

3. **Feature (Major Version)** `feat/BREAKING CHANGE`
      - Step 1: Localhost - Develop and test changes locally.
      - Step 2: PR to Staging
        - **Required Reviews:** 1 Peer Reviewer + Copilot Review + QA Reviewer.
        - **Required Checks:** CI/CD (GitHub Action checks) must pass.
      - Step 3: PR to Production (main)
        - **Tagging: Tag the Feature code.**
        - **Required Reviews:** 1 Peer Reviewer + Copilot Review.
        - **Required Checks:** CI/CD (GitHub Action checks) must pass.

  Note: "Copilot Review" means Copilot code review runs automatically on the pull request. In repositories where Copilot approvals are enabled, a Copilot approval on a `staging` pull request counts as the Peer Reviewer under our SOC 2 change-management control. Changes to CI/CD, deployment config and agent instruction files still need a human code owner. On `main`, Copilot reviews and a human peer approves. Repositories with Copilot approvals enabled: `deardoc-dashboard-frontend` (trial).

1. **Emergency Hotfix Protocol** ("Break Glass")
Since Admin Bypass is disabled to meet SOC2 requirements, the following protocol must be followed for critical production outages (P0) where the standard flow is too slow:
  a. Accelerated Review: During an outage, the developer must alert the #engineer Slack channel.
  b. Solution 1: Rollback the problematic changes.
  c. Solution 2 (If rollback is not feasible):
      - Commit Fix: Push the fix to a new branch and immediately open a PR.
      - On-Call Approval: Alert the #engineer Slack channel. Any available engineer (regardless of seniority) can perform a rapid review to provide the mandatory 1-approval.
      - Audit Trail: If a branch protection rule must be temporarily modified by an Org Admin to resolve a catastrophic failure (extreme cases only):
          - The change must be documented in a Monday.com ticket.
              - The rule must be re-enabled immediately after the hotfix.
              - A post-mortem must be conducted to explain why the bypass was necessary for the compliance audit log.

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

## Project Documentation

To maintain a standard structure while developing new projects, it is recommended to use the [Technical Documentation](./TechnicalDocumentation.md) file as a template for your project's README.md file.

## Modifications

This document and every other file in this repository are living documents and will be updated as the team grows and the company evolves.

If you have any suggestions or want to propose a change, please open a pull request and submit it.
