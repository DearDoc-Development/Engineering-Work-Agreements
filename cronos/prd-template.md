# Agent-Ready PRD Template

Copy this file to `docs/projects/<project-name>/project-requirements.md` and fill in the sections. Delete the explanatory blockquotes when you're done.

A PRD is "agent-ready" when an AI can implement it without asking clarifying questions. The test: if two different agents read this and built meaningfully different things, the PRD isn't done yet.

---

# <Project Name>

## 1. Summary

> One paragraph. What problem this solves, who it's for, what shipping it looks like. No marketing copy.

## 2. Goals & non-goals

**Goals:**
- ...

**Non-goals:**
- ...

> Non-goals are load-bearing. They tell the agent what *not* to build. Without them, scope creeps in every prompt.

## 3. Stakeholders & risk tier

| Role        | Person   | Responsibility                |
| ----------- | -------- | ----------------------------- |
| PM          |          | Owns this doc                 |
| Implementer |          | Owns the build                |
| Validator (optional) |  | Reviews risky work if the team asks. Not a release gate |
| Sponsor     |          | Core engineer who reads a contributor's code and checks it in production. Needed if the Implementer is a contributor |
| Code Owner  |          | Approves the plan and both PRs for restricted areas |

**Risk tier:** Low / Medium / High
**Why:** _e.g., touches payments → High; copy change → Low._
**Restricted areas touched?** Yes / No. _If yes, list them (see the [restricted areas](../README.md#restricted-areas)). Any restricted area is High._

## 4. User stories

> One per row. Each links to specific acceptance criteria in §8.

| ID | As a... | I want to... | So that... | Acceptance |
| -- | ------- | ------------ | ---------- | ---------- |
| US-1 | | | | §8.1 |
| US-2 | | | | §8.2 |

## 5. State enumeration

> List every UI/system state the feature can be in. "Make it responsive" is not a state. "Loading", "empty (first-run)", "empty (filtered)", "partial-failure", "error (retryable)", "error (terminal)", "success" are states.

**Component / View: <name>**
- `idle`
- `loading`
- `empty:first-run`
- `empty:filtered`
- `error:network`
- `error:permission`
- `success`

Repeat per component.

## 6. Data model

> Tables / collections / Firestore paths / DynamoDB keys. Field names, types, indexes. Migration plan if existing data must change shape.

**Path:** `accounts/{accountId}/locations/{locationId}/<resource>/{id}`

| Field         | Type    | Required | Notes                                |
| ------------- | ------- | -------- | ------------------------------------ |
|               |         |          |                                      |

**Migration:** _e.g., backfill default value X for existing docs in cycle N+1._ Data changes can't be rolled back, so add first and remove in a later release (expand → contract). Migrations are a restricted area. For a major change, write a one-page major release plan first (template in Appendix D of the [full release protocol](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol-full.md), private wiki).

## 7. API contracts

> Exact request/response shapes per endpoint. Status codes. Error envelope. Auth model. Examples use synthetic data only, never real patient data.

### POST /api/<resource>

**Auth:** `Bearer <firebase-jwt>`, custom claim `role=admin`.

**Request:**
```json
{ "field": "string" }
```

**Response (200):**
```json
{ "id": "string", "createdAt": "ISO-8601" }
```

**Errors:**
- `400` — validation; body: `{ code: "INVALID", details: [...] }`
- `403` — auth fails
- `409` — duplicate

Repeat per endpoint.

## 8. Acceptance criteria

> Per requirement, testable. The agent uses these as its definition-of-done.

### 8.1 <User story US-1>

- [ ] Given <state>, when <action>, then <result>.
- [ ] Field <X> is required and rejects empty string with 400.
- [ ] Audit log records <event> with `{actorId, resourceId, timestamp}`.

### 8.2 <User story US-2>

- [ ] ...

## 9. Tech constraints

**Stack:**
- Frontend: <framework + version>
- Backend: <runtime + framework + version>
- Storage: <db>
- Auth: <provider>

**Mandatory libraries:**
- ...

**Forbidden patterns:**
- ...

> _e.g., "no `eval`", "no `any` in TypeScript", "no direct DOM mutation outside refs", "no synchronous Firestore writes from cloud functions"._

## 10. Security & privacy

- PII fields touched: ... (examples and fixtures use synthetic data only)
- New auth surface: ...
- Logging policy: _what may NOT be logged_.
- Threat model: _at minimum, can a different tenant read this? Can an unauthenticated user hit this URL?_

## 11. Out of scope (explicit)

> A bulleted list of things you considered and decided *not* to do this cycle. Future-cycle candidates can be linked.

- ...

## 12. Open questions

> Tracked, not hidden. Each one needs an owner and a target answer date. Empty list is fine — better than handwaving.

| Q | Owner | Needed by |
| - | ----- | --------- |
|   |       |           |

## 13. Cycle plan

| Cycle | Scope                                      | Risk | Validator (optional) |
| ----- | ------------------------------------------ | ---- | --------- |
| 1     | Schema + scaffolding + happy path          | Low  |           |
| 2     | Edge cases + auth + tests                  | Med  |           |
| 3     | Cross-repo wiring + release prep           | High |           |

> Update this table as cycles complete — it becomes the project's history.

---

## Companion files in `docs/projects/<project-name>/`

- `project-requirements.md` — this file.
- `architecture.md` — diagrams, sequence flows, key decisions.
- `schema.md` — data model with rationale (why this shape, why not another).
- `operating.md` — runbook: how to debug and monitor. Deploy and rollback follow the [release protocol](../README.md#release-protocol).
- `testing.md` — test plan and coverage targets.
- `pr-descriptions.md` — draft PR descriptions per repo per cycle. Use the PR template from the release protocol (one ticket per PR).
- `sop.md` — standard operating procedures for the people running the project once it ships.

You don't need all of these on day one. Add them as the project demands.
