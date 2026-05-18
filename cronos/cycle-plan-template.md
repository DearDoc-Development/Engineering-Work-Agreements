# Cycle Plan Template (the "Rigorous Plan")

The Implementer writes this on Monday morning and posts it before writing any code. It goes through **Mission Control** — the PM (or tech lead) approves it before implementation starts. This is the cheapest hour of the cycle: catching a misaligned approach in prose costs minutes; catching it in committed code costs days.

The PRD ([prd-template.md](prd-template.md)) defines *what* and *why*. This plan defines *how* — at a level concrete enough that approval means something.

Copy this file to your project tracker (Linear / Monday / Notion) or to `docs/projects/<name>/cycle-N-plan.md`. Delete the explanatory blockquotes when filling it in.

---

# Cycle <N> Plan — <Project Name>

**Implementer:** <name>
**Validator:** <different name — must not be the Implementer>
**Risk tier:** Low / Medium / High (from the PRD)
**Cycle dates:** YYYY-MM-DD → YYYY-MM-DD

## 1. Scope (in)

> Bullets, one per concrete change. Each bullet should be small enough that you can point at the file(s) it touches.

- ...
- ...

## 2. Scope (out)

> Things you considered and explicitly are *not* doing this cycle. The Validator and PM use this to challenge scope drift later.

- ...

## 3. PRD requirements covered

> Map each scope item back to a PRD acceptance criterion. If something in scope doesn't map, either the PRD is missing it (update PRD first) or the work isn't justified.

| Scope item | PRD §  | Acceptance criterion ID |
| ---------- | ------ | ----------------------- |
|            |        |                         |

## 4. File-level changes

> The actual diff plan. New files, modified files, deleted files. One row per file. This is the main artifact the reviewer reads.

| Repo           | File                                  | Change          | Why                                   |
| -------------- | ------------------------------------- | --------------- | ------------------------------------- |
| backend        | `src/services/foo.ts`                 | new             | houses the new `bar` operation         |
| backend        | `src/types.ts`                        | modified        | add `BarRequest` / `BarResponse`       |
| frontend       | `src/hooks/useBar.ts`                 | new             | client wrapper for the new endpoint    |
| frontend       | `src/components/BarPanel.tsx`         | modified        | wire to `useBar`                       |
| (any)          | `<old encrypted module>`              | deleted         | replaced by plaintext-on-config model  |

## 5. Schema / contract changes

> Anything that crosses a boundary. Fill out even when the answer is "none" — saying it explicitly is the point.

- **Database schema:** ...
- **API contract:** ...
- **Auth / claims:** ...
- **Permissions / rules:** ...
- **Backwards compatibility:** does this break existing clients? If yes, what's the migration path?

## 6. Test approach

| Layer        | What we'll test                                       | Tool / location                |
| ------------ | ----------------------------------------------------- | ------------------------------ |
| Unit         | Pure logic in `bar` service                            | `__tests__/bar.test.ts`        |
| Integration  | Endpoint round-trip with auth + DB                    | `__tests__/api/bar.int.test.ts`|
| E2E / UI     | User can complete the new flow in browser             | Manual smoke + screenshots     |
| Security     | Bypass attempts, malformed input, expired tokens      | Validator's adversarial pass   |

> If a row is "n/a", say so explicitly. "n/a" is fine; silence is not.

## 7. Hot spots

> Places you expect the AI to struggle. Concurrency, race conditions, vendor SDKs with thin docs, non-linear state. Surface them now so the Validator knows where to look hardest.

- ...

## 8. Validator's pre-brief

> One paragraph the Validator can read on Thursday afternoon to start their half-day cold. Tell them what changed, what to challenge, what to ignore as out-of-scope.

...

## 9. Cross-repo coordination

> If this cycle touches more than one repo, list the order commits / PRs land in and what depends on what. Backend contract changes land before consumers; consumers can't ship without them.

- Repo A merges first: ...
- Repo B merges second, depends on: ...

## 10. Rollout / rollback

- **Feature flag?** y/n. If yes, name it.
- **Migration ordering?** ...
- **How to roll back if Friday's deploy goes wrong:** ...

## 11. Mission Control sign-off

- [ ] PRD requirements above are covered and don't drift
- [ ] File-level plan is concrete enough to review
- [ ] Test approach matches risk tier
- [ ] Validator named and available Thursday PM / Friday AM
- [ ] Cross-repo dependencies sequenced

**Approver:** <name>
**Approved at:** YYYY-MM-DD HH:MM

---

## How to use this template well

- **Write it before talking to the AI.** Once you've prompted the model, the model's structure starts shaping your structure. The plan should reflect *your* analysis of the PRD, not the agent's first sketch.
- **The file table is non-negotiable.** "I'll touch the user service" is not a plan. "I'll add `getUserPreferences()` to `src/services/user.ts` and update `Preferences` in `src/types.ts`" is.
- **If you can't fill in §3, stop.** Either the work isn't grounded in the PRD or the PRD is incomplete. Both block; both are fixable in 30 minutes if caught now.
- **The plan is allowed to change mid-cycle** — but only by amending this doc and re-routing through Mission Control if the change is material (new file, new contract, new dependency). "I noticed and fixed an unrelated bug" → that's a separate cycle, not a free rider on this one.
