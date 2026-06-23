---
name: generate-prd
description: Generate a structured Product Requirements Document (PRD) for OSAC features. Incorporates OSAC project context, canonical personas, cross-cutting feature dimensions, and review expectations. Use when asked to create a PRD, product spec, requirements document, or feature definition for the OSAC project.
---

# OSAC PRD Generation Skill

Generate a Product Requirements Document for OSAC using the template, project context, and guidelines below.

> **IMPORTANT**: Return ONLY the PRD content. Do not include any planning text, explanations of what you're doing, or meta-commentary. Start directly with the PRD title.

## Instructions

1. Find and read `osac-context.md` (located alongside this SKILL.md, e.g. `skills/*/generate-prd/osac-context.md`)
2. Find and read `prd-template.md` (located alongside this SKILL.md, e.g. `skills/*/generate-prd/prd-template.md`)
3. **Fetch attachments**: Check whether the feature ticket has any attachments (e.g. mockups, research docs, specs, diagrams). Use `mcp__atlassian__jira_download_attachments` or equivalent Jira tools to retrieve them. For each attachment, attempt to read or fetch its content and incorporate it as additional context. If an attachment cannot be read (e.g. unsupported binary format), note its filename and skip it.
4. **Explore relevant repos**: Based on the feature requirements, browse the relevant OSAC component repos (listed in osac-context.md) to understand the current codebase state. Focus on:
   - Existing API definitions (proto files) that the feature touches
   - Existing controllers and server implementations
   - Related enhancement proposals in the `enhancement-proposals` repo
5. Analyze the raw requirements combined with codebase context and attachment content
6. Fill in all sections of the template, using the OSAC personas and dimensions from osac-context.md
7. Validate against the quality checklist

## Size Calibration

Match PRD depth to feature complexity. A single config field or small enhancement does not need the same treatment as a new subsystem.

- **Personas**: Use the 4 OSAC canonical personas (Cloud Provider Admin, Cloud Infrastructure Admin, Tenant Admin, Tenant User). Only include personas actually affected by this feature.
- **Services**: Declare which OSAC services are in scope (BMaaS, CaaS, VMaaS, MaaS, Enclave).
- **Dimensions**: Address each cross-cutting dimension from osac-context.md. For dimensions not relevant to this feature, state "Not affected" — do not omit them silently.
- **User stories**: Write one story per distinct user goal. Three similar stories for the same user and goal should be one story with multiple acceptance criteria.
- **Glossary**: Only include terms that are non-obvious to engineers familiar with Kubernetes and cloud infrastructure.
- **Timeline**: Include milestone scoping (e.g., 0.1, 0.2). If no dates are provided, the timeline is TBD but the target milestone should still be declared.
- **Risks**: 2-4 specific, realistic risks. Each must name a concrete failure mode, not a vague category.

## Generation Rules

1. **Be Specific**: Avoid vague language. Every requirement must be testable.
2. **Prioritize**: Use MVP (must-have), non-MVP (should-have), nice-to-have.
3. **User-Centric**: Frame everything from the user's perspective using OSAC personas.
4. **Measurable**: Include specific metrics and acceptance criteria where meaningful. Do not invent metrics to fill the table.
5. **No Implementation**: Focus on WHAT, not HOW. No technical solutions — those belong in the design document (EP).
6. **Honest Constraints**: Only list constraints that are definitively known to apply. Do not speculate.
7. **No scope creep**: Only document requirements explicitly stated or strongly implied by the raw requirements.
8. **Dimension coverage**: Address every cross-cutting dimension from osac-context.md, even if just to say "Not applicable."
9. **Cross-repo awareness**: When a feature touches multiple OSAC components, note the cross-component dependencies.

## Markdown Formatting

Output must be valid markdown. For tables:
- Every row must start AND end with `|`
- All rows must have the same number of columns
- Include separator row after header: `|---|---|---|`

Example:
```markdown
| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | Description | MVP |
```

## Quality Checklist

Before returning the PRD, verify:

- [ ] Executive summary is concise (2-3 sentences)
- [ ] Problem statement clearly articulates the pain point
- [ ] Services in scope are declared (BMaaS, CaaS, VMaaS, MaaS, Enclave)
- [ ] Target milestone is declared
- [ ] At least 1 OSAC persona defined with goals and pain points
- [ ] All four OSAC personas are considered (even if only to say "not affected")
- [ ] All functional requirements have acceptance criteria
- [ ] Success metrics are quantifiable with specific targets (when included)
- [ ] Scope boundaries clearly defined (in/out of scope)
- [ ] Cross-cutting dimensions addressed (tenant onboarding, inventory, provisioning, networking, storage, installation)
- [ ] API surfaces affected are identified (Fulfillment API, OSAC CRDs, Catalog Items)
- [ ] Risks have mitigation strategies
- [ ] No technical implementation details included
- [ ] Cross-repo dependencies noted where applicable

## Output Format

Follow the structure in `prd-template.md` (located alongside this SKILL.md) exactly.
