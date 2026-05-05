# principal-engineer

A Claude Code agent configuration that acts as a principal engineer — providing high-level technical guidance, architecture reviews, and formal design artifacts without ever producing implementation code.

## What it does

The `principal-engineer` agent challenges assumptions, identifies risks, and gives direct opinions. It routes requests to specialized skills based on what's needed:

| Request type | Output |
|---|---|
| Architecture / design decision | Technical design document (`.ai/specs/`) |
| Standalone decision to document | ADR in `docs/adrs/` |
| REST or async API contract | OpenAPI / AsyncAPI spec |
| Security review of a flow | STRIDE threat model |
| Exploratory question | Inline analysis (no artifact) |

For every design-committing request, the agent runs a quality-attribute interview first (`grill-me`) before routing to the relevant skill.

## Skills

| Skill | Trigger |
|---|---|
| `tech-design` | "tech design", "design document", solution architecture |
| `adr` | "ADR", architectural decision to record |
| `api-design` | "design an API", "review this spec", OpenAPI / AsyncAPI work |
| `threat-modeling` | "threat model", "security review", attack surface analysis |
| `grill-me` | "grill me", stress-testing a plan or design |

## Using the agent

Invoke the agent from Claude Code:

```
claude --agent principal-engineer
```

Or invoke skills directly:

```
/adr <decision context>
/tech-design <requirements or ticket description>
/api-design <resource and operations to design>
/threat-modeling <flow or component to assess>
/grill-me <plan to stress-test>
```

## Hard constraints

- The agent never produces source code, method bodies, type signatures, or infrastructure scripts — design only.
- Design-committing requests always go through a quality-attribute interview before any artifact is produced. Skip it by saying "proceed".
- ADR numbers are sequential and never reused.
- Threat mitigations are always concrete — no vague "add security" advice.
- API specs never invent domain terminology outside project glossary and existing specs.

## Output locations

| Artifact | Path |
|---|---|
| Technical designs | `.ai/specs/<slug>/tech-design.md` |
| Diagrams | `.ai/specs/<slug>/diagrams/NN-description.md` |
| ADRs | `docs/adrs/NNNN-title.md` |
| Threat models | `.ai/specs/<slug>/threat-model.md` |
| API specs | `packages/api-clients/<service-name>/` |
