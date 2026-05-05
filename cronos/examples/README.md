# Sanitized Cycle Examples

These are real cycles, with identifying details removed. They're written as case studies — onboarding material for engineers who want to see what a Cronos cycle actually feels like in practice, not just in template form.

The full unsanitized versions — with the actual prompts, mocks, and decisions — live in the private `ai-toolkit` repo under `cronos/cycles/`. If you have access there and want to learn from the real artifacts, start there. These public examples cover the *shape* of a cycle; the private cycles cover the *substance*.

## Index

- [multi-repo-payments-rework.md](multi-repo-payments-rework.md) — High-risk cycle touching four repos in one week. Removes a misplaced encryption layer, simplifies a vendor integration, fixes a CMS bug, and reworks an email + receipt flow. Demonstrates: cross-repo coordination, type-first refactors, the half-day Validator gate under high risk, and what happens when format scripts overreach.

## How to read these

Each example follows the same structure:

1. **The setup** — project, risk tier, repos touched, scope.
2. **How it ran against the playbook** — day-by-day, with what worked and what didn't.
3. **Lessons** — concrete takeaways that propagate to future cycles.

The "what didn't work" sections are the most valuable. Cronos cycles are designed to surface failure modes early — a cycle where nothing went wrong is rarer than one where two or three things did, were caught, and were fixed before shipping. Read the misses.

## Contributing a new example

When you finish a cycle worth documenting:

1. Write the full version in the private `ai-toolkit` repo first (cycles/<NNNN>-<slug>/).
2. Sanitize a copy for this directory — strip company name, commit SHAs, internal repo names, customer-identifying details.
3. Open a PR here. Keep the writing tight: case studies that read like sermons get skipped.
