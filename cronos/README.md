# Cronos

A lightweight framework for running AI-augmented engineering cycles without losing the verification rigor of professional software development.

Cronos pairs the speed of vibe coding (natural-language intent → AI-generated code) with three guardrails that prevent it from collapsing under its own velocity:

1. **Agent-ready PRDs** — requirements written so an AI can execute them deterministically, not paragraphs an AI has to interpret.
2. **One-week cycles** — a fixed cadence that forces validation to happen this week, not "later."
3. **Half-day peer validation** — someone other than the prompter mentally executes the code as an attacker. This catches Author's Bias, the trap where the engineer who prompted the model trusts its confident-sounding output too much.

## What lives here vs. in the AI toolkit

This repo (`engineering-work-agreements`) is the **public, methodology-level** view of Cronos. It defines the cadence, the gates, the templates, and the principles. Read this to understand *how we work*.

The actual operating artifacts — prompt libraries, agent skills, completed cycle archives with the prompts that produced them, mocks, decisions, and learnings — live in our private **`ai-toolkit`** repo. Read that to see *what we ran and what came out*.

Rough split:

| Lives here (public)                       | Lives in `ai-toolkit` (private)                 |
| ----------------------------------------- | ----------------------------------------------- |
| Methodology, cycle structure, principles  | Reusable prompts, meta-prompts                  |
| PRD + plan templates (blank shells)       | Filled-in cycle archives (one folder per cycle) |
| Sanitized example cycles for onboarding   | Full unsanitized cycles with prompts + mocks    |
| Agent-guide-as-policy                     | Claude skills (`SKILL.md`), agent definitions   |
| Validation checklist                      | Org-specific knowledge, conventions, glossaries |

> **Where to find `ai-toolkit`:** internal-only. Ask in `#engineering` on Slack, or check the team handbook for the private repo URL.

## Read order

If you are bootstrapping a new project:

1. **[methodology.md](methodology.md)** — what Cronos is and why each piece exists. ~5 min.
2. **[cycle-playbook.md](cycle-playbook.md)** — how a cycle actually runs, day by day, role by role. ~5 min.
3. **[prd-template.md](prd-template.md)** — copy this when starting a new project. The PM-side scope/stories/testing shell.
4. **[cycle-plan-template.md](cycle-plan-template.md)** — copy this on Monday morning before writing any code. The implementation plan that goes through Mission Control.
5. **[agent-guide.md](agent-guide.md)** — paste-ready policy for any AI agent operating inside a Cronos repo.
6. **[examples/](examples/)** — sanitized completed cycles. Onboarding gold; copy patterns, not specifics.

If you are an AI agent invoked inside a repo that uses Cronos, jump to **[agent-guide.md](agent-guide.md)** first.

## The shortest possible summary

> A PM writes a deterministic PRD. The team agrees on how many one-week cycles it'll take. Each cycle: scaffold Monday, build Tue–Wed, test Thursday, review-and-document Friday. Half a day of every cycle is mandatory peer validation by someone who didn't prompt the AI. Ship at the end of the week. Repeat.

That's it. Everything else is mechanics.
