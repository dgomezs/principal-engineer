---
name: tech-design
description: "Generates high-level technical design documents from requirements and codebase research. Use when the user says 'tech design', 'create tech design', 'technical design for', 'design document', or provides a ticket-id and wants architecture/solution documentation. Also trigger when user wants to document a solution approach before implementation."
disable-model-invocation: true
user-invocable: true
argument-hint: <context: requirements, research findings, or ticket description>
---

# Technical Design Generator

Generate a high-level technical design document from provided context.

**Core principles**: precise, conservative, direct, efficient. Minimize tokens without losing information. Avoid colloquial language.

## STEP 1: Parse Context

The user provides all needed context via `$ARGUMENTS` — requirements, research findings, ticket descriptions, or a combination. Read and internalize everything provided.

If `$ARGUMENTS` is empty or insufficient to understand what needs to be built, stop and ask the user to provide requirements and any relevant codebase research.

## STEP 2: Gather Additional Context

**Knowledge discovery**: Check `CLAUDE.md` for instructions on loading relevant knowledge (architecture docs, domain concepts, coding guidelines). Query the knowledge system for domain concepts and architecture decisions that constrain the design.

If the provided context references files, specs, or code — read them.

## STEP 3: Clarify Requirements

Resolve ALL ambiguities before designing the solution.

**If invoked from the principal-engineer agent**: the agent will have already completed its quality-attribute interview. Do not re-ask those questions. Proceed with the decisions recorded as assumptions.

**If invoked directly**: invoke the `grill-me` skill before proceeding. Use `references/quality-attributes.md` as the agenda — cover domain decomposition, bounded contexts, and all relevant quality attributes. Do not proceed to Step 4 until the interview is complete or the user says "proceed".

## STEP 4: Identify Solution

Determine the high-level recommended solution. If the provided context lacks critical information, warn the user visibly and conduct additional targeted research via codebase exploration.

## STEP 5: Select Diagrams

**CRITICAL**: This is a separate step. Do NOT merge with Step 3. Do NOT generate diagrams yet.

Based on solution analysis, recommend applicable diagram types using AskUserQuestion with `multiSelect: true`:

Diagram types to consider:

- **Sequence diagrams**: API flows, multi-step processes, inter-service communication
- **Component diagrams**: System architecture, module relationships, service boundaries
- **State machines**: Lifecycle management, workflow states, status transitions
- **Entity relationships**: Data models, database schemas
- **Activity diagrams**: Complex business logic, decision trees, parallel processes
- **Deployment diagrams**: Infrastructure layout, service distribution

For each applicable type, suggest 1-5 specific diagrams based on the identified solution. Always include an "Other" option for custom requests.

Example question format:

> Which diagrams should be generated for this technical design?
>
> - [ ] Sequence diagram: Order creation API flow
> - [ ] Component diagram: Service architecture overview
> - [ ] State machine: Order lifecycle states
> - [ ] Other

## STEP 6: Generate Technical Document

Determine an appropriate output path. Default: `.ai/specs/[ticket-id]/tech-design.md` if a ticket-id is available, otherwise `.ai/specs/[descriptive-slug]/tech-design.md`.

Write the document following the schema below. After creating it, create ONLY the diagrams selected by the user as `.ai/specs/[id]/diagrams/[NN-description].md` files containing Mermaid diagram code blocks. Use numbered prefix (01-, 02-). Reference diagrams from tech-design.md.

## STEP 7: Coherence Check

Verify the document is:
- Logically consistent
- Complete (all sections filled appropriately)
- Feasible given the existing architecture
- Aligned with requirements
- Informative for the target audience (engineers familiar with the codebase)

## STEP 8: Remaining Questions

If ambiguities remain about the design or implementation approach, ask clear, direct, grouped questions using AskUserQuestion.

## STEP 9: Refine Document

Merge user input into the technical document.

## STEP 10: Final Output

Final coherence check. If issues found, return to Step 6. Otherwise, output the path to tech-design.md.

---

## Template and Rules

Read `references/tech-design-template.md` for the document schema, section rules, diagram file format, and hard rules. Follow it exactly when generating the tech design document in Step 6.
