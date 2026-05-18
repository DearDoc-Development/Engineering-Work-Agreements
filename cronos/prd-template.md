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
| Validator   |          | Owns the half-day review      |

**Risk tier:** Low / Medium / High
**Why:** _e.g., touches payments → High; copy change → Low._

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

**Migration:** _e.g., backfill default value X for existing docs in cycle N+1._

## 7. API contracts

> Exact request/response shapes per endpoint. Status codes. Error envelope. Auth model.

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

- PII fields touched: ...
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

| Cycle | Scope                                      | Risk | Validator |
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
- `operating.md` — runbook: how to deploy, roll back, debug, monitor.
- `testing.md` — test plan and coverage targets.
- `pr-descriptions.md` — paste-ready PR descriptions per repo per cycle.
- `sop.md` — standard operating procedures for the people running the project once it ships.

You don't need all of these on day one. Add them as the project demands.
