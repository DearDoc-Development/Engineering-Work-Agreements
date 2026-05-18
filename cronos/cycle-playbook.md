# Cycle Playbook

Operational walk-through of a single one-week Cronos cycle. Adapt the times, not the gates.

## Pre-cycle: the Kickoff (≈ 1 hour)

Happens once per project, then again whenever a new Epic starts.

**PM brings:**
- Agent-ready PRD (see [prd-template.md](prd-template.md))
- Epic breakdown — how the work decomposes
- Risk tier per Epic (low/medium/high)

**Team agrees on:**
- How many cycles each Epic takes ("vibe-ability" check — standard CRUD = 1 cycle, payments integration = 3+).
- Architectural hot spots where the AI is likely to struggle (race conditions, non-linear state, vendor SDKs with thin docs).
- Rules of engagement — update the repo's `CLAUDE.md` / `.cursorrules` with any project-specific guardrails.
- Who's the Validator for each cycle (must be a different person than the Implementer).

Output: an Epic in the tracker, a project doc at `docs/projects/<project-name>/`, and an updated `CLAUDE.md`.

## Day 1 — Monday: Initialization

**Implementer:**
1. Reads the PRD and the project doc end-to-end.
2. Drafts a *plan* — not code. File-by-file changes, new endpoints, schema deltas, test approach.
3. Posts the plan and **waits for PM approval** (Mission Control gate).
4. Once approved, scaffolds with the agent: schemas, routes, empty handlers, type definitions.

**PM:**
- Available for plan-review questions. This is the cheapest hour of the week — use it.

**Done by EOD Monday:** plan approved, scaffolding committed, tracker updated.

If the plan reveals the PRD is ambiguous, **stop and fix the PRD**. Do not implement against a guess.

## Days 2–3 — Tuesday/Wednesday: Implementation

The "vibe coding" phase. Conversational loop with the agent: *see stuff, say stuff, run stuff.*

**Discipline during this phase:**

- **Mid-loop verification** — every 30–60 minutes, mentally execute the diff. Don't rely on "tests pass" as the only signal. The agent will write tests that pass against its own assumptions.
- **One change per commit** — even when the agent produced five changes in a single response. Squash later if you must.
- **Surface every assumption** — if the agent invents a function name, library version, or env variable, verify it exists before moving on. Hallucinations compound silently.
- **Use the agent to explain its own code** — paste a generated block back and ask "what does this do and what assumptions does it make?" Forces both you and the model to surface the reasoning.

**When to stop and re-plan:**

- The agent is "fixing" the same bug for the third time.
- A required vendor SDK doesn't behave like the agent thinks it does.
- A schema change cascades further than the plan anticipated.

In all three cases, return to Mission Control. Don't push through.

## Day 4 — Thursday: Verification

Generation stops. Verification starts. Three layers:

1. **Automated tests** — agent writes unit + integration tests against the PRD's acceptance criteria. Run them. Fix failures *as the implementer*, then re-run.
2. **Visual / functional audit** — for UI work, click through the actual feature in a browser. Type checking and tests verify code correctness, not feature correctness. If you can't run the UI, say so explicitly — don't claim success.
3. **Security scan** — `npm audit`, `snyk`, or your equivalent. For higher-risk tiers, run a prompt-injection / OWASP Top-10 pass with a tool that knows AI patterns (Promptfoo, Checkmarx One Assist).

The Implementer is still on the hook for fixing what verification finds. The Validator hasn't started yet.

## Day 4 PM / Day 5 AM — Validation (the half-day gate)

**A different engineer does this. Not the Implementer.** This is the load-bearing rule.

The Validator's job is *not* to rubber-stamp. It's to mentally execute the code as an attacker.

**Validation checklist:**

| Category         | Question                                                              |
| ---------------- | --------------------------------------------------------------------- |
| Hallucination    | Does every imported symbol actually exist in our environment?         |
| Auth             | Is authorization tied to server logic, not just UI state?             |
| Bypass           | Can a user skip the UI and call this endpoint directly with curl?     |
| Concurrency      | What happens if two requests hit this code path simultaneously?       |
| State            | What happens on partial failure? On retry? On a half-written write?   |
| PII / secrets    | Is anything sensitive logged, returned, or stored unencrypted?        |
| Edge cases       | Negative numbers, null, empty array, very large input, expired token. |
| Cross-repo       | Did the contract change? Did the consumer change in lockstep?         |

The Validator either signs off or sends it back. There is no "looks good but…" — either it ships or it doesn't.

## Day 5 PM — Finalization & Review

**Implementer:**
- Inline comments only where the *why* is non-obvious. Skip "// gets the user" — keep "// retry up to 3x because vendor returns 429 on cold start."
- Update the project doc at `docs/projects/<project-name>/` if anything material changed.
- Generate release notes from commits.

**PM + Implementer + Validator (15 min):**
- Demo working software, not slides.
- Decide: ship, hold, or roll back.
- If hold: what's the unblock condition, and is it next cycle's first task?

Hand off to deploy. Vibe deploy = one-click to staging then prod, gated by the Validator's sign-off.

## Cross-cycle hygiene

- **Cycle 4 or 5 = Technical Health Cycle.** No features, just refactors of AI-generated code that's accumulating debt.
- **Project doc lives at `docs/projects/<name>/`** — single source of truth, not duplicated per repo. Update it as scope evolves; archive when the project ships.
- **Cross-repo changes coordinate releases.** Backend contract change on `main` → frontend / integration change must land same cycle.
- **Memory the agent collects belongs to the agent's profile, not the project doc.** Project doc captures decisions and context; agent memory captures collaboration preferences.

## Failure modes to watch for

- **Skipped Mission Control** → "I'll just try it real quick" → 4 hours later, broken architecture.
- **Validator = Implementer** → review becomes ceremony, regressions ship.
- **PRD written after the fact** → you can't validate against intent that didn't exist.
- **Format/lint scripts that touch unrelated files** → scope your auto-format to the directories you actually changed. A wide reformatter dragging 60 files into your PR is a review-killer.
- **"Tests pass" as the validation signal** → tests pass. The agent wrote them to pass. Mental execution still required.

## Read next

- [agent-guide.md](agent-guide.md) — what the AI agent should be told to do during each phase.
- [examples/multi-repo-payments-rework.md](examples/multi-repo-payments-rework.md) — a real cycle and the things this playbook didn't catch.
