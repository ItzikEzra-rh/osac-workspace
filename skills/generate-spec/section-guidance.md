# OSAC Design Document (EP) — Section Guidance

Instructions for how to fill each section of the OSAC design template.
This file is read during generation. It is not included in the final output.

## General Rules

- **Favor conciseness.** Every sentence should earn its place.
- Write in third person, present tense.
- **Be specific.** No vague language: "efficient data structure" -> name the structure. "Appropriate caching" -> specify the cache strategy. "Standard error handling" -> define the error taxonomy.
- Every design decision must be traceable to source material. Use source markers: `[PRD: FR-3]`, `[PRD: NFR-2]`, `[Locked: D{N}]`, `[User]`, `[Assumption]`, `[Codebase: path/to/file]`.
- **Consolidate markers.** Tag each decision with its source only when non-obvious. Rely on frontmatter `tracking-link` and `prd` for overall reference. Reserve inline markers for clarification-derived changes, user instructions, codebase-derived decisions, and assumptions.
- **Incorporate, don't narrate.** Write the design decision in its final form. Do not describe what was removed or why a previous position was abandoned.
- Do NOT invent requirements. If the PRD doesn't specify something, mark it as an assumption or open question.
- If information is unavailable, write "To be determined — {what's needed}".
- **No scope reduction.** Never use "simplified version", "v2", "placeholder", or "future enhancement" to silently reduce scope.
- **Formatting restraint.** Use bold sparingly for genuine emphasis.
- **Diagrams:** Use Mermaid diagrams when they add clarity. Keep diagrams simple: labeled nodes, clear edge labels, no styling directives. Every diagram must have narrative explaining what it shows.
- **Keep all required headers.** The enhancement-proposals repo enforces required sections via linter CI.

## Per-Section Guidance

### YAML Frontmatter

- **title**: Lowercase slug with hyphens (e.g., `networking-api`, `bare-metal-fulfillment`)
- **authors**: Email addresses (e.g., `agentil@redhat.com`)
- **creation-date**: ISO date format (YYYY-MM-DD)
- **last-updated**: ISO date format, update when making significant changes
- **tracking-link**: Full Jira URL (e.g., `https://redhat.atlassian.net/browse/OSAC-356`)
- **prd**: Relative path to the PRD document (typically `prd.md`)
- **see-also**: Related enhancements as paths (e.g., `/enhancements/networking`)
- **replaces/superseded-by**: Usually `N/A` for new proposals

### Summary

1-2 sentences. What this design achieves and the technical approach. End with a PRD reference: "See [PRD](prd.md) for detailed requirements."

### Motivation

2-4 paragraphs. Restate the problem in implementation terms. Do NOT include user stories. Bridge from the PRD: assume the reader needs enough context to understand the design decisions, but direct them to the PRD for the full picture.

### Goals

- Design-scoped goals that constrain the implementation, not product outcomes.
- "Reuse the existing controller reconciliation pattern" is a design goal.
- "Tenants can create VirtualNetworks" is a product goal (belongs in the PRD).
- 3-5 goals, each one sentence.

### Non-Goals

- Prevent scope creep at the implementation level.
- 2-4 non-goals, each one sentence.

### Proposal

1-2 paragraphs introducing the key resources/APIs. For OSAC, this typically means naming new CRDs, gRPC services, and controller changes. Explain how they relate.

### Workflow Description

- Define actors using OSAC personas (Provider Admin, Infrastructure Admin, Tenant Admin, Tenant User).
- Enumerate steps from a defined starting state.
- Be explicit about APIs (gRPC, REST, kubectl).
- Include error handling and alternative paths.
- Use Mermaid sequence diagrams for multi-step interactions.

### API Extensions

For OSAC, typically includes:
- New gRPC services in fulfillment-service
- New CRDs in osac-operator
- Webhooks for validation/defaulting
- Finalizers for cleanup
- Changes to existing resources owned by other teams

List each extension and note operational impact.

### Implementation Details/Notes/Constraints

Technical depth:
- Proto schema snippets (fields, enums, validation rules)
- Database schema (new tables, migrations)
- Controller reconciliation logic (state machine, finalizer flow)
- Integration with existing components (fulfillment-service, osac-operator, osac-aap)
- Follow OSAC proto conventions: `snake_case` fields, `PascalCase` messages, `SCREAMING_SNAKE_CASE` enums

### Security Considerations

- Input validation, auth/authz changes, data exposure risks.
- For multi-tenant features: OPA policies, namespace scoping, `osac.openshift.io/tenant` annotation filtering.
- If inheriting existing model: state that and explain why sufficient.
- Do not invent security concerns that don't apply.

### Failure Handling and Recovery

- Enumerate concrete failure modes (not generic categories).
- For each: what happens, how the system recovers, what the user sees.
- Cover controller-side (reconciliation errors, stale caches), API-side (validation, database), and integration failures (AAP job timeouts, network provisioning).
- Note retry behavior and idempotency.
- Describe behavior when controller is restarted mid-reconciliation.

### RBAC / Tenancy

- All new resources MUST include tenant isolation metadata: `osac.openshift.io/tenant`, `osac.openshift.io/owner-reference`.
- Describe OPA policy enforcement.
- Visibility constraints: can a tenant see resources from other tenants? What about platform-defined resources (NetworkClass, PublicIPPool)?
- If no changes: "No RBAC or tenancy changes required." with justification.

### Observability and Monitoring

- New Prometheus metrics: name, type (counter/gauge/histogram), labels, problem threshold.
- New Kubernetes events: type (Normal/Warning), reason, when it fires.
- New structured log events.
- If none: "No new observability changes. Existing monitoring mechanisms apply."

### Risks and Mitigations

- Technical risks only (product risks are in the PRD).
- Each with concrete mitigation or "To be determined."
- Consider: version skew, performance bottlenecks, security exposure, backwards compatibility, cross-component coordination.

### Drawbacks

- Steel-man argument against the proposal.
- Trade-offs, maintenance burden, API complexity.

### Alternatives (Not Implemented)

- At least one alternative for each non-trivial decision.
- Include "Do nothing" if applicable.
- Brief description, pros, cons, rejection reason.

### Open Questions

- Each numbered. Frame as clear, answerable questions.
- Remove when resolved by incorporating answers into relevant sections.
- Optional — omit if no open questions remain.

### Test Plan

- Strategy: unit (Ginkgo), integration (Kind), e2e (pytest via osac-test-infra).
- Call out tricky areas: CIDR parsing, dual-stack, concurrent reconciliation.
- If details depend on implementation: "Test plan will be developed during implementation. Expected coverage: [describe]."

### Graduation Criteria

- If not targeting a release: "Graduation criteria will be defined when targeting a release. Expected stages: Dev Preview -> Tech Preview -> GA."
- If targeting: define maturity levels and success signals.

### Upgrade / Downgrade Strategy

- New APIs: "New API with no upgrade impact. Downgrade requires deleting all instances before reverting."
- Existing APIs: describe migration steps and backward compatibility.

### Version Skew Strategy

- How fulfillment-service and osac-operator handle version skew during upgrades.
- CRD version migration requirements.

### Support Procedures

- Failure detection: symptoms (events, metrics, alerts, log output).
- Disabling: how and consequences on cluster health, workloads.
- Recovery: re-enable and consistency.

### Infrastructure Needed

- Usually "None" for OSAC EPs.
- If needed: new test infrastructure, repos, or CI changes.
