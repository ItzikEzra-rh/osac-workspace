---
name: generate-spec
description: Generate a Technical Design Document (Enhancement Proposal) for OSAC features from a PRD. Uses the OSAC EP template with sections for RBAC/tenancy, failure handling, observability, security, and more. Use when asked to create a spec, technical specification, design document, or enhancement proposal for the OSAC project.
---

# OSAC Specification (Enhancement Proposal) Generation Skill

Generate a Technical Design Document using the OSAC Enhancement Proposal template, section guidance, and project context below.

> **IMPORTANT**: Return ONLY the EP content. Do not include any planning text, explanations of what you're doing, or meta-commentary. Start directly with the YAML frontmatter.

## Core Purpose

An EP translates PRD requirements into a **detailed technical design**. It answers "exactly how does this behave and how do we build it?" — not "why are we building it?" (that's the PRD).

**Do not repeat PRD content.** The EP assumes the reader has access to the PRD. Skip business context, personas, and rationale — go straight to technical approach and behavior.

## Instructions

1. Find and read `osac-context.md` (located alongside this SKILL.md, e.g. `skills/*/generate-spec/osac-context.md`)
2. Find and read `spec-template.md` (located alongside this SKILL.md, e.g. `skills/*/generate-spec/spec-template.md`)
3. Find and read `section-guidance.md` (located alongside this SKILL.md, e.g. `skills/*/generate-spec/section-guidance.md`)
4. Read the parent PRD content (provided as input)
5. **Explore relevant repos**: Based on the PRD requirements, browse the relevant OSAC component repos (listed in osac-context.md) to understand:
   - Existing API definitions (proto files) for resources this feature touches
   - Existing controllers and their reconciliation patterns
   - Existing enhancement proposals in the `enhancement-proposals` repo for reference
   - Cross-repo dependency implications
6. Design the solution using the codebase context and PRD requirements
7. Fill in all sections of the EP template following the section guidance
8. Validate against the quality checklist

## Size Calibration

Match design depth to feature complexity. Only include a section if it adds information specific to this feature — not because the template has a slot for it.

Ask for each section: "Does this feature have something concrete and specific to say here?" If the answer is no, explain why the section is not applicable but do not remove it — the enhancement-proposals repo enforces required sections via linter CI.

## Generation Rules

1. **Trace to PRD**: Reference FR/US IDs from the parent PRD where relevant. Use source markers: `[PRD: FR-3]`, `[Codebase: path/to/file]`, `[Assumption]`.
2. **Consolidate markers**: Don't tag every statement with `[PRD: §X.Y]` — use markers only when the source is non-obvious or differs from the primary PRD.
3. **No ambiguity**: Use precise language. Avoid "should", "might", "could".
4. **Error format matches the stack**: Go/CLI tools use field validation errors, not HTTP error codes. Web APIs use HTTP status codes.
5. **OSAC proto conventions**: `snake_case` fields, `PascalCase` messages, `SCREAMING_SNAKE_CASE` enums.
6. **Controller patterns**: Follow osac-operator reconciliation pattern (finalizer → status update → provisioning lifecycle).
7. **Tenant isolation**: All new resources MUST include `osac.openshift.io/tenant` and `osac.openshift.io/owner-reference` annotations.
8. **Keep all required headers**: The enhancement-proposals repo enforces required sections via linter CI. If a section does not apply, explain why but do not remove it.
9. **Incorporate, don't narrate**: When a PRD revision changed scope, write the design decision in its final form. Do not describe what the original PRD said or what was removed.
10. **No scope reduction**: Never use "simplified version", "v2", "placeholder", or "future enhancement" to silently reduce scope.

## Markdown Formatting

Output must be valid markdown. For tables:
- Every row must start AND end with `|`
- All rows must have the same number of columns
- Include separator row after header: `|---|---|---|`

Use Mermaid diagrams when they add clarity. Keep diagrams simple: labeled nodes, clear edge labels, no styling directives (`style`, `classDef`, color codes). Every diagram must be accompanied by narrative explaining what it shows.

## Quality Checklist

Before returning the EP, verify:

- [ ] YAML frontmatter is complete (title, authors, dates, tracking-link, prd reference)
- [ ] Summary is 1-2 sentences with PRD reference
- [ ] All template sections are present (even if marked N/A with justification)
- [ ] Every design decision is traceable to the PRD or marked as an assumption
- [ ] Proto schemas follow OSAC conventions (snake_case fields, PascalCase messages)
- [ ] Controller logic follows osac-operator reconciliation pattern
- [ ] Tenant isolation metadata is specified for all new resources
- [ ] RBAC/Tenancy section addresses OPA policies and visibility constraints
- [ ] Failure handling covers controller-side, API-side, and integration failures
- [ ] Security considerations address tenant isolation enforcement
- [ ] Observability section lists new metrics, events, or log events (or states "none needed")
- [ ] Alternatives section includes at least one alternative for each non-trivial design decision
- [ ] Test plan specifies unit, integration, and e2e strategy
- [ ] Cross-repo dependencies are identified and addressed

## Output Format

Follow the structure in `spec-template.md` (located alongside this SKILL.md) exactly, using `section-guidance.md` for per-section writing instructions.
