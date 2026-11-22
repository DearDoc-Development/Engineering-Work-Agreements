# The Cronos Methodology

## Why a new framework

Two-week sprints were calibrated for human typing speed. With AI agents that can scaffold a service in minutes, the bottleneck moved from *writing code* to *validating code*. Cronos reorganizes the SDLC around that new bottleneck.

Two failure modes Cronos is specifically engineered against:

- **Ambiguity** → the AI guesses, hallucinates APIs, invents schemas. Fixed by deterministic PRDs.
- **Author's Bias** → the engineer who wrote the prompt trusts the model's confident output and skips real review. Fixed by independent review: Copilot at Checkpoint 1 and a second engineer at Checkpoint 2 of the [release protocol](../README.md#release-protocol).

## Core unit: the one-week cycle

A cycle is 5 working days. The cadence is non-negotiable; the *contents* of a cycle scale with the project. A cycle organises the work. Code ships through the daily release, not at the end of the cycle. Friday is a cycle review, not a ship-or-hold decision.

| Day        | Phase           | Output                                          |
| ---------- | --------------- | ----------------------------------------------- |
| Mon        | Initialization  | Tickets with file-level plans and rollbacks, scaffolding |
| Tue–Wed    | Implementation  | Features built via prompted iteration           |
| Thu        | Verification    | Automated tests, security scans, visual audit on staging |
| Fri        | Finalization    | Docs, release notes, cycle review               |

Independent review is the load-bearing constraint. It happens at PR time (Copilot, Checkpoint 1) and at release time (Checkpoint 2), not in a block at the end of the week. A team may still ask someone to validate risky work. That is optional and is not a gate.

## The four roles

- **PM** — translates stakeholder intent into an agent-ready PRD. Owns the requirement determinism.
- **Implementer** — prompts the AI, reviews each generation, drives the cycle. Owns "the vibe." Maps to the **Contributor** or **Core Engineer** of the release protocol.
- **Validator** (optional) — a different engineer who reviews risky work if the team asks. Owns adversarial thinking. The protocol's reviewers are Copilot (Checkpoint 1) and the engineer who checks the release (Checkpoint 2). Code Owners approve [restricted areas](../README.md#restricted-areas).
- **Agent (Claude / Cursor / Copilot)** — generates code under guardrails set in `CLAUDE.md` / `.cursorrules`. Owns volume, not judgment. Copilot is also the Checkpoint 1 reviewer.

The Implementer and the reviewer must be different people. This is the single most-violated rule in practice and the one that produces the most regressions. The protocol enforces it: nobody ships their own change alone.

## What makes a PRD "agent-ready"

A PRD is agent-ready when an AI can implement it without asking clarifying questions. Concretely:

- **State enumeration** — every UI/system state listed (idle, loading, empty, error, partial, success). No "make it responsive."
- **API contracts** — exact request/response shapes, status codes, error envelopes.
- **Tech constraints** — pinned framework versions, mandatory libraries, forbidden patterns.
- **Acceptance criteria** — per-requirement, testable. "Definition of done" the AI can self-check against.

If two different agents reading the PRD would build meaningfully different things, it isn't agent-ready yet. See [prd-template.md](prd-template.md).

## Plans and Mission Control

Every change has a file-level plan, with its rollback, written in the Monday ticket before any code. A `#fix` needs three lines: the change, the expected result, the rollback.

Approval before code is needed only when the change touches a [restricted area](../README.md#restricted-areas). The approver is a Code Owner. The Implementer presents the plan and **waits for that approval before writing code**.

This is the cheapest place to catch a misaligned implementation — before it exists.

## The Vibe-to-Verify Ratio

Calibrate review intensity to risk. A CRUD form built from a known pattern needs less verify than an auth flow or a payments integration. The PM marks each Epic with its expected ratio in the PRD; reviewers use it to decide how deep to read.

| Risk tier  | Examples                              | Verify allocation                       |
| ---------- | ------------------------------------- | --------------------------------------- |
| Low        | CRUD UI, settings page, copy change   | Skim diff, run smoke test               |
| Medium     | New API endpoint                      | Mental execution, edge cases, tests     |
| High       | Data migrations, any restricted area (see the list), or cross-repo work | Code Owner approval, attacker mindset, sec scan |

## Technical Health Cycles

Every fourth or fifth cycle is a **Technical Health Cycle** — no new features. The goal is paying down AI-generated debt: code that works but is opaque, tightly coupled, or hides assumptions. Without this, cycles 6–10 slow to a crawl as each new feature touches more brittle ground.

## Sovereignty

Default to local, code-first agents (Claude Code, Cursor) operating on a standard Git repo. Browser-based, walled-garden agents lock you into someone else's prompt format and someone else's release schedule. Cronos assumes the codebase is portable.

## What Cronos is not

- Not a project management tool. The tracker is Monday. Cronos is the cadence and the templates, not the tracker.
- Not a coding standard. Repos still need their own `CLAUDE.md` / style guide / architecture rules.
- Not a replacement for testing. The verification phase assumes the testing pyramid already exists; Cronos schedules when to use it.

## Read next

- [cycle-playbook.md](cycle-playbook.md) — operational walk-through of one cycle.
- [prd-template.md](prd-template.md) — what an agent-ready PRD actually looks like.
- [examples/multi-repo-payments-rework.md](examples/multi-repo-payments-rework.md) — a real cycle, with what went wrong.
