---
name: api-design
description: "Design and review API contracts (REST or async/event). Use when asked to design an API, create an OpenAPI/AsyncAPI spec, or review an existing contract. Produces a spec file and reviews it against project conventions."
user-invocable: true
argument-hint: <API to design or spec file to review — describe the resource, operations, or event flows involved>
---

# API Design

Design or review an API contract following project conventions.

API conventions are bundled in `references/` alongside this skill — read the relevant files before producing or reviewing any spec.

| File | When to read |
|---|---|
| `references/openapi-spec.md` | Any REST/OpenAPI work |
| `references/openapi-search.md` | Search endpoints |
| `references/asyncapi-spec.md` | Async/event APIs |
| `references/contract-first.md` | Workflow and file layout |

## STEP 1: Determine Mode

From `$ARGUMENTS`, determine whether this is:

- **Design** — produce a new spec from requirements
- **Review** — critique an existing spec against project conventions

If ambiguous, ask.

---

## Design Mode

### STEP 2: Understand the API

Identify from `$ARGUMENTS`:

- **API type** — REST (OpenAPI) or async/event (AsyncAPI)
- **Resources or events** — what entities or domain events are involved
- **Operations** — what actions consumers need to perform
- **Consumers** — who calls this API and what do they need

Read existing specs in the repo (search `packages/api-clients/` and `apps/backend/`) to understand conventions already in use. Read related domain docs in `docs/domain/` for correct terminology.

If requirements are thin, ask — one batch of questions.

### STEP 3: Clarify Design Questions

Resolve before writing the spec:

- Which operations are needed (CRUD subset, search, etc.)?
- Are there list vs. search distinctions?
- What events are published vs. consumed?
- Are there any non-obvious error cases or state transitions?
- What authentication/authorization applies at this boundary?

### STEP 3.5: Resolve Cross-Cutting Concerns

Before writing the spec, answer these for every mutation and high-traffic query:

| Concern | Questions to answer |
|---|---|
| **Idempotency** | Do mutation endpoints accept a client-supplied idempotency key? Safe to retry on network failure? |
| **Rate limiting** | What are the per-client / per-consumer limits? What is the response when exceeded (429 + `Retry-After`)? |
| **Pagination bounds** | Is there a maximum `limit`? What happens when the caller exceeds it? |
| **Error envelope** | Is the error shape consistent across all endpoints (type, code, message, detail)? |
| **Retry semantics** | Which responses are safe to retry (5xx, 429)? Which are not (400, 409)? Is this documented? |
| **Deprecation policy** | How are breaking changes communicated? Version in path (`/v2/`) or header? Sunset header used? |
| **Authentication scope** | Which endpoints are public, authenticated, or privileged? Documented per-operation? |

Don't design these from scratch — check `references/contract-first.md` and `references/openapi-spec.md` first.

### STEP 4: Write the Spec

Read the relevant files from `references/` before writing:
- `references/openapi-spec.md` for REST APIs
- `references/asyncapi-spec.md` for event/async APIs
- `references/openapi-search.md` if search endpoints are involved
- `references/contract-first.md` for contract-first conventions

Save the spec to `packages/api-clients/<service-name>/` or the path the user specifies.

### STEP 5: Review the Spec

After writing, review it yourself against the rules (see Review Mode below). Fix issues before surfacing the output.

---

## Review Mode

### STEP 2: Read the Spec

Read the spec file from the path in `$ARGUMENTS`.

### STEP 3: Read the Rules

Read all files in `references/` relevant to the spec type.

### STEP 4: Produce the Review

For each issue found, produce one entry:

| Severity | Location | Issue | Fix |
|---|---|---|---|
| Error / Warning / Suggestion | `path`, field, or operation | What's wrong | What to do instead |

- **Error** — violates a hard rule; blocks acceptance
- **Warning** — likely wrong; should be fixed
- **Suggestion** — improvement that doesn't break conventions

End with a summary: pass / needs changes, and the highest-severity issues to address first.

### STEP 5: Fix (if asked)

If the user asks to apply fixes, edit the spec file directly. Re-run the review after fixing to confirm clean.

---

## Hard Rules

- Never invent domain terminology — use terms from `docs/glossary.md` and existing specs
- Never duplicate the convention rules inline — they live in `references/`
- No implementation code or backend logic — contracts only
