# Example Cycle: Multi-Repo Payments Rework

A real Cronos cycle, sanitized. Useful as a worked example of what a high-risk, multi-repo cycle looks like in practice — what worked, what didn't, what the playbook caught and what it missed.

## The setup

**Project:** A patient-facing payment-link product integrated with a third-party card-processing vendor (Run Payments / CardConnect on the Cardpointe gateway).

**Project doc location:** `docs/projects/<project-name>/` at the workspace root — single source of truth across all four repos. PRD, architecture, schema, operating notes, and PR-description scaffolds all live there, not duplicated per repo.

**Repos touched:**

1. **Backend** — Node/TypeScript on Firebase Functions
2. **Frontend** — React 19 / TanStack Query / Firebase
3. **Admin CMS** — FireCMS-based internal tooling
4. **Web integration** — embeddable widgets (the public pay page)

**Risk tier:** High — payments + cross-repo + Firestore rules + secret removal.

**Cycle scope (explicit, given verbatim by the PM at kickoff):**

- Remove the encryption-at-rest layer for vendor credentials. Drop the `secrets/<id>` subcollection, drop the `*_CREDENTIAL_ENCRYPTION_KEY` env secret, store credentials plaintext on the product config doc, protected by Firestore rules.
- Allow internal admins (any internal user, not just super-admins) to delete payment-transaction docs.
- In the admin CMS, resolve `contactId` as a contact-entity reference, not a raw string.
- Drop the location-level QR code; generate a per-payment-request QR and surface it on the dashboard.
- Remove the webhook secret entirely.
- Email cleanup — drop the redundant center "Amount Due" panel.
- Receipt PDF — drop the browser-injected page header/footer; match the on-screen confirmation view.

Seven distinct changes across four repos. Tight scope but high coordination.

## How it ran against the playbook

### Monday — Initialization ✓

- Read the project doc (PRD + architecture + schema) end-to-end before opening an editor. The cross-repo project doc paid for itself immediately — the encryption removal touched all four repos in different ways, and having one source meant no contradiction between them.
- Drafted a per-repo plan using [cycle-plan-template.md](../cycle-plan-template.md). File-level table for each repo. Hot spots flagged: Firestore-rule expansion (admin scope), unauthenticated webhook (no more secret), CMS contactId resolution (FireCMS limits).
- Got Mission Control approval. *Then* started writing.

Cost: ~90 minutes to produce the plans. They held throughout the cycle — no scope drift, no "I noticed this and also fixed it" interruptions.

### Tue–Wed — Implementation ✓ with notes

What worked:

- **Type-first refactor.** Started by editing the central types file to remove the now-dead encrypted shape. TypeScript compiler errors then walked the agent to every callsite that needed updating. Faster and more thorough than `grep` would have been.
- **Service-layer rewrite as a thin facade.** Rebuilt the credentials service around `put` / `get` / `getOrNull` / `updateTokens` — small named methods that map directly to the new plaintext model. No leftover encryption shape lurking in helper functions.
- **Firestore dot-path updates** for token rotation (`config.credentials.accessToken` etc.) preserved sibling fields without race-prone read-modify-write loops.

Where the framework caught a mistake:

- Tried to write `.claude/settings.local.json` to grant the agent broader permissions for unattended work. The framework correctly blocked this as **"self-modification of the agent's own permission file."** Fix: added the file to `.gitignore` and gave the human the JSON to paste manually. **The agent must never grant itself permissions** — this is a hard line in `agent-guide.md` for a reason.

### Thursday — Verification ✓ partial

What ran:

- Backend: full test suite passed (~1,400 tests), lint clean.
- Frontend: type-check clean. Lint surfaced **7 pre-existing errors in unrelated files** — acknowledged and *deferred* to a Technical Health Cycle. Not in scope for this PR.
- Admin CMS: full quality gate clean.
- Web integration: type-check + tests pass.

What didn't run:

- Browser smoke test of the new payment-link → QR → public pay-page flow with a sandbox merchant. Documented as a follow-up. **Calling this out matters more than hiding it** — the Validator (or the PM) decides whether it's blocking.

### Validation (the half-day) — partial

This particular cycle was solo (Implementer = the human reviewing AI output, no separate Validator). That's a **deviation from the playbook on a high-risk cycle** and worth naming. In a normal high-risk cycle the half-day Validator would have been adversarially testing:

- **The unauthenticated webhook endpoint** (after secret removal). The reasoning held — the endpoint only flips status on existing docs keyed by the vendor's transaction ID, so there's no privilege escalation. But this should be challenged adversarially by a different engineer, not waved through by the prompter. Author's Bias is strongest precisely when the implementation went smoothly.
- **The Firestore rule expansion.** Adversarial test: can an internal user delete transactions across tenants? Yes — that's the intent. But the intent should be confirmed against compliance posture, not assumed.

**Lesson:** "high-risk tier" means the half-day Validator slot is non-negotiable, even when the cycle goes smoothly. Don't skip it because the diff "looks right."

### Friday — Finalization ✓ with one miss

What worked:

- Comments rewritten to explain *why* (vendor CVV bundling, sandbox-vs-prod env mode, FireCMS PropertyPreview limitation), not what.
- Project doc updated alongside code, so the next cycle's Implementer reading the project folder sees the new shape, not the old one.
- One commit per repo, focused message per commit.

What missed:

- **A wide formatter pass overreached.** Running `npm run format` at the repo root in the web-integration repo reformatted ~60 files outside the directories actually touched this cycle. Caught and reverted with `git checkout -- <unrelated paths>`. The PR was rescoped back to just the two files actually changed. **Scope formatters narrowly.** Run them on the directories you edited, not the repo root.
- **An unrelated todo file** got swept into the backend commit — a workspace artifact from a different in-flight effort. Flagged for follow-up rather than amending the commit (per the no-amend rule in `agent-guide.md`).
- **`.env` couldn't be committed** (gitignored). The live edit removed the now-dead encryption key, but the change isn't tracked in git history. Acceptable here; worth noting for environments where env files are tracked.

## Outputs at end of cycle

| Repo            | Status                                            |
| --------------- | ------------------------------------------------- |
| Backend         | Tests + lint clean, one commit ahead of origin   |
| Frontend        | Type-check clean, pre-existing lint debt deferred |
| Admin CMS       | Quality gate clean                                |
| Web integration | Type-check + tests pass                           |

All four working trees clean, each repo +1 commit ahead of origin. Push and PR-open delegated to the human (gated — the agent does not push).

## Lessons (carryover into the next cycle)

1. **Cross-repo project doc paid for itself.** Touching four repos in one cycle would have been chaos without `docs/projects/<project>/` as the single source. Keep this pattern for all multi-repo work; resist the temptation to copy excerpts into per-repo `README`s.
2. **Scope formatters narrowly.** Run `prettier --write` on the directories you actually edited, not the repo root. Wide reformats turn a focused PR into a 60-file review that nobody reads carefully.
3. **Pre-existing debt is not in scope.** Catalog it (lint backlog, unrelated todo files, stale formatter state) and surface it to the human, but don't fix it in the same PR. Fixes hide inside reformats; reformats hide regressions.
4. **Permission boundaries hold under pressure.** The "agent cannot self-grant permissions" rule blocked an attempt that *felt* reasonable in context. The rule was right; the attempt was wrong. The agent should never try again, in any cycle, in any project.
5. **High-risk cycles need the Validator slot, even when the diff looks clean.** Author's Bias is strongest precisely when the implementation went smoothly. Skipping the half-day on the grounds that "this one went well" is the exact failure mode the gate exists to prevent.
6. **Type-first refactors propagate cleanly.** When removing a model-level distinction (encrypted vs. plaintext), edit the type definition first and let the compiler walk you to every callsite. Faster and more thorough than grep.

## What this cycle would have looked like in pure-Agile

- Two-week sprint instead of one-week.
- The encryption removal alone would likely have been its own ticket; the QR rework another; the email/receipt cleanup a third — three sprints minimum.
- Validation would have been a 15-minute PR review per repo, not a half-day adversarial pass.
- The CMS contactId fix would have been deferred to a "polish sprint" and never shipped.

The Cronos cycle landed seven distinct changes across four repos in one week, with explicit verification, type-system-driven safety, and a clear paper trail. The half-day Validator gate is what makes that pace sustainable — without it, the pace compounds into regressions by cycle 3.

## Read next

- [../methodology.md](../methodology.md) — what each piece of the framework is for.
- [../cycle-playbook.md](../cycle-playbook.md) — how to run your own cycle.
- [../cycle-plan-template.md](../cycle-plan-template.md) — the Monday-morning plan template.
- [../agent-guide.md](../agent-guide.md) — what to tell your AI agent before they start.
