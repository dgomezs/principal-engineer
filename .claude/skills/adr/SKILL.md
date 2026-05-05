---
name: adr
description: "Write an Architecture Decision Record. Use when a significant technical decision needs to be documented — build-vs-buy, framework choice, protocol selection, structural constraint. Produces a numbered ADR in docs/adrs/ following project conventions."
user-invocable: true
argument-hint: <decision to document — describe the context, what was decided, and why>
---

# ADR Writer

Produce a numbered ADR in `docs/adrs/` following project conventions.

## STEP 1: Parse Context

Read `$ARGUMENTS`. Identify:

- **The decision** — what was chosen
- **The context** — what situation forced the decision
- **Alternatives** — what else was considered
- **Rationale** — why this option over others

If any of these are missing and cannot be inferred, ask before proceeding.

## STEP 2: Anchor in Reality

Read `docs/adrs/` to:
- Find the next sequential ADR number (scan existing files, take highest + 1)
- Check for related existing ADRs to cross-reference
- Understand the decision's relationship to prior choices

Read relevant code, config, or design docs if they inform the context or consequences.

## STEP 3: Clarify if Needed

If the decision is unclear or the rationale is thin, ask targeted questions — one batch, not one at a time:

- What forces made this decision necessary now?
- What was the strongest alternative and why was it rejected?
- What does this decision make harder?
- Are there conditions under which this decision should be revisited?

Skip this step if `$ARGUMENTS` is already complete.

## STEP 4: Write the ADR

Use the next sequential number. File name: `NNNN-kebab-case-title.md`.

Follow this exact format (matches `docs/adrs/template.md`):

```markdown
---
tags: [adr, <domain-tag>, decisions-accepted]
aliases: [ADR-NNNN, <Short Title>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# ADR-NNNN: <Title>

**Date**: YYYY-MM-DD
**Status**: proposed | accepted | superseded by [ADR-XXXX](XXXX-title.md)

## Context

What situation prompted this decision? What forces, constraints, or requirements were in play? Keep this factual — describe the problem, not the solution.

## Decision

What was decided? State it clearly in one or two sentences.

## Consequences

**Positive**: what does this decision enable or improve?

**Negative / trade-offs**: what does this decision cost or constrain?

**Risks**: what could go wrong, and how is it mitigated?

## Alternatives considered

What other options were evaluated and why were they rejected?

## Related

- Links to other ADRs, domain pages, or features this decision affects
```

Today's date is used for `created`, `updated`, and **Date**.

## STEP 5: Update the Index

Append an entry to `docs/log.md` (newest first):

```
## [YYYY-MM-DD] adr | ADR-NNNN: <Title>
```

Check `docs/index.md` — if ADRs are listed there, add the new entry.

## STEP 6: Output

State the file path and ADR number. If related ADRs should be updated to reference this one, say so explicitly — but do not edit them without asking.

---

## Hard Rules

- ADR numbers are sequential and never reused — always read existing files to find the next number
- Status is `proposed` by default unless the user confirms the decision is already accepted
- Do not invent alternatives that weren't genuinely considered — ask if unsure
- Context describes the problem, not the solution — keep them separate
- No implementation code, pseudocode, or config snippets
