---
name: threat-modeling
description: "STRIDE threat model for a system flow or trust boundary. Use when asked to threat model, security review, or assess attack surface of a design. Produces a structured threat table per trust boundary crossing."
user-invocable: true
argument-hint: <flow or component to threat model — describe the trust boundaries, actors, and data involved>
---

# STRIDE Threat Modeling

Produce a structured STRIDE threat model for the described flow or component.

## STEP 1: Understand the Scope

Read `$ARGUMENTS`. Identify:

- **Actors** — who initiates actions (users, services, external systems)
- **Trust boundaries** — where authority changes (public internet → API, API → DB, service → service, etc.)
- **Data flows** — what data crosses each boundary
- **Assets** — what is worth protecting (credentials, PII, business data, availability)

If scope is unclear or trust boundaries are unspecified, ask before proceeding.

If relevant code or design docs exist, read them to anchor the model in reality.

## STEP 1b: Classify Data

Before enumerating threats, classify every data type that flows through the system:

| Class | Examples | Baseline controls |
|---|---|---|
| **Secret** | Credentials, tokens, private keys | Never logged, encrypted at rest, short-lived |
| **PII** | Names, emails, addresses, device IDs | Minimized, right-to-delete path exists, audit log |
| **Internal** | Internal IDs, business logic, config | Not exposed in error messages or public APIs |
| **Public** | Publicly visible content | No special controls |

This classification drives the weight of Information Disclosure threats in STEP 2.

If the flow involves an OAuth 2.0 / OIDC provider, also check:
- Token types in play (access token, ID token, refresh token) and their lifetimes
- PKCE used for authorization code flow? (required for public clients)
- Redirect URI validation — exact match enforced?
- Refresh token rotation — revocation on reuse?
- Session fixation — is the session ID regenerated post-login?
- Token replay — audience (`aud`) and issuer (`iss`) claims validated server-side?

## STEP 2: Enumerate Threats

For each trust boundary crossing, evaluate all six STRIDE categories:

| Category | Question to ask |
|---|---|
| **S**poofing | Can an attacker impersonate an actor or system at this boundary? |
| **T**ampering | Can data be modified in transit or at rest? |
| **R**epudiation | Can an actor deny having performed an action? |
| **I**nformation Disclosure | What data leaks if this boundary is breached? |
| **D**enial of Service | What can be exhausted, blocked, or crashed here? |
| **E**levation of Privilege | Can a low-trust caller gain high-trust access? |

Only include categories where a real threat exists. Skip inapplicable ones with a note.

## STEP 3: Produce the Threat Table

For each identified threat, produce one row:

| ID | Boundary | Category | Threat | Likelihood | Impact | Mitigation |
|----|----------|----------|--------|------------|--------|------------|
| T1 | User → API | Spoofing | Attacker forges JWT to impersonate another tenant | Medium | High | Verify JWT signature and `sub` claim server-side; reject tokens without expiry |

- **Likelihood**: Low / Medium / High
- **Impact**: Low / Medium / High
- **Mitigation**: Concrete control — not "add security"

## STEP 4: Summary

After the table, write a short paragraph (3–5 sentences):
- Highest-risk threats and why
- Any systemic weaknesses across boundaries
- Recommended priority order for mitigations

## STEP 5: Save to File

Save the threat model to `.ai/specs/<slug>/threat-model.md` where `<slug>` describes the flow (e.g., `tenant-login-flow`, `slot-booking-api`). Use today's date if no slug is obvious.

Output the file path when done.

---

## Hard Rules

- Never recommend vague controls ("validate input", "use encryption") — always specify what, where, and how
- Do not threat model in the abstract — anchor every threat to a specific actor, boundary, and data flow
- If a mitigation already exists in the codebase, say so — don't recommend what's already there
- Skip STRIDE categories that genuinely don't apply rather than forcing weak threats
