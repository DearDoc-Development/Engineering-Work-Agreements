# The Cronos Methodology

## Why a new framework

Two-week sprints were calibrated for human typing speed. With AI agents that can scaffold a service in minutes, the bottleneck moved from *writing code* to *validating code*. Cronos reorganizes the SDLC around that new bottleneck.

Two failure modes Cronos is specifically engineered against:

- **Ambiguity** → the AI guesses, hallucinates APIs, invents schemas. Fixed by deterministic PRDs.
- **Author's Bias** → the engineer who wrote the prompt trusts the model's confident output and skips real review. Fixed by mandatory cross-engineer validation.

## Core unit: the one-week cycle

A cycle is 5 working days. Every cycle ends with a release decision — ship, hold, or roll back. The cadence is non-negotiable; the *contents* of a cycle scale with the project.

| Day        | Phase           | Output                                          |
| ---------- | --------------- | ----------------------------------------------- |
| Mon        | Initialization  | Plan approved, scaffolding committed            |
| Tue–Wed    | Implementation  | Features built via prompted iteration           |
| Thu        | Verification    | Automated tests, security scans, visual audit   |
| Thu PM/Fri | Validation      | Half-day peer review by another engineer        |
| Fri        | Finalization    | Docs, release notes, ship-or-hold decision      |

Half a day of validation is the load-bearing constraint. Cut it and the framework stops working.

## The four roles

- **PM** — translates stakeholder intent into an agent-ready PRD. Owns the requirement determinism.
- **Implementer** — prompts the AI, reviews each generation, drives the cycle. Owns "the vibe."
- **Validator** — a different engineer who does the half-day review. Owns adversarial thinking.
- **Agent (Claude / Cursor / Copilot)** — generates code under guardrails set in `CLAUDE.md` / `.cursorrules`. Owns volume, not judgment.

The Implementer and Validator must be different people. This is the single most-violated rule in practice and the one that produces the most regressions.

## What makes a PRD "agent-ready"

A PRD is agent-ready when an AI can implement it without asking clarifying questions. Concretely:

- **State enumeration** — every UI/system state listed (idle, loading, empty, error, partial, success). No "make it responsive."
- **API contracts** — exact request/response shapes, status codes, error envelopes.
- **Tech constraints** — pinned framework versions, mandatory libraries, forbidden patterns.
- **Acceptance criteria** — per-requirement, testable. "Definition of done" the AI can self-check against.

If two different agents reading the PRD would build meaningfully different things, it isn't agent-ready yet. See [prd-template.md](prd-template.md).

## Mission Control

For any non-trivial change (new feature, schema change, API surface change, cross-repo work), the Implementer presents the plan to the PM/team and **waits for explicit approval before writing code**. Bug fixes and small refactors don't need the gate unless they cross repo boundaries.

This is the cheapest place to catch a misaligned implementation — before it exists.

## The Vibe-to-Verify Ratio

Calibrate review intensity to risk. A CRUD form built from a known pattern needs less verify than an auth flow or a payments integration. The PM marks each Epic with its expected ratio in the PRD; the Validator uses it to allocate their half-day.

| Risk tier  | Examples                              | Verify allocation                       |
| ---------- | ------------------------------------- | --------------------------------------- |
| Low        | CRUD UI, settings page, copy change   | Skim diff, run smoke test               |
| Medium     | New API endpoint, data migration      | Mental execution, edge cases, tests     |
| High       | Auth, payments, PII, cross-repo       | Full half-day, attacker mindset, sec scan |

## Technical Health Cycles

Every fourth or fifth cycle is a **Technical Health Cycle** — no new features. The goal is paying down AI-generated debt: code that works but is opaque, tightly coupled, or hides assumptions. Without this, cycles 6–10 slow to a crawl as each new feature touches more brittle ground.

## Sovereignty

Default to local, code-first agents (Claude Code, Cursor) operating on a standard Git repo. Browser-based, walled-garden agents lock you into someone else's prompt format and someone else's release schedule. Cronos assumes the codebase is portable.

## What Cronos is not

- Not a project management tool. Use Linear, Monday, Jira — Cronos is the cadence and the gates, not the tracker.
- Not a coding standard. Repos still need their own `CLAUDE.md` / style guide / architecture rules.
- Not a replacement for testing. The verification phase assumes the testing pyramid already exists; Cronos schedules when to use it.

## Read next

- [cycle-playbook.md](cycle-playbook.md) — operational walk-through of one cycle.
- [prd-template.md](prd-template.md) — what an agent-ready PRD actually looks like.
- [examples/multi-repo-payments-rework.md](examples/multi-repo-payments-rework.md) — a real cycle, with what went wrong.
