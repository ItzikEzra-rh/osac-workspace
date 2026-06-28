---
name: prd-review-5dim
description: |
  Review an OSAC PRD using the same depth as prd-review but scoring on
  5 standardized dimensions (what/why/how/task/size, 0-2 each) compatible
  with the Org Pulse dashboard assessment format.

  Trigger when reviewing a PRD for dashboard-compatible scoring.
---

# OSAC PRD Reviewer (5-Dimension Scoring)

## Overview

This skill reviews a Product Requirements Document with the same depth and
checks as the original `prd-review` skill, but scores on 5 standardized
dimensions (0-2 each, total 0-10) compatible with the Org Pulse RFE Review
dashboard.

The review checks are identical to `prd-review` — every checklist item,
scoring guide, and severity classification is preserved. Only the output
scoring structure differs.

## When to Use

- Automated PRD review via the EP Review Agent (dashboard integration)
- When scores need to match the Org Pulse assessment format

## Input Detection

Same as `prd-review`:

1. **PR URL or number** → `gh pr diff <N> --repo osac-project/enhancement-proposals`
2. **Local file path** → `enhancement-proposals/enhancements/<slug>/prd.md`
3. **Context dir** → `.context/pr-diff.txt` (when run by the EP Review Agent)

## Load Context

Before reviewing, read these files if available:

1. `.design/context/osac-dimensions.md` — services, personas, cross-cutting dimensions
2. `.design/context/review-patterns.md` — reviewer feedback themes
3. `.context/skill-prompt.md` — review instructions (agent mode)
4. `.context/template.md` — PRD template reference (agent mode)

## Review Dimensions (5-Dimension Scoring)

Each dimension is scored 0-2:

| Score | Meaning |
|-------|---------|
| 2 | Solid — meets or exceeds expectations |
| 1 | Present but weak — gaps that need attention |
| 0 | Missing or broken — fundamental problems |

PASS threshold: total >= 5 out of 10.

---

### Dimension 1: What (0-2) — Problem Clarity

*Does the PRD clearly describe the desired outcome?*

Maps to the original Clarity dimension. Check:
- [ ] Problem statement leads with user pain, not solution (3-5 sentences)
- [ ] Problem statement quantifies impact if source material supports it
- [ ] Goals are measurable outcomes, not activities ("Users can deploy X" not "Implement deployment")
- [ ] No vague language ("appropriate", "efficient", "standard" without specifics)
- [ ] Terminology is consistent throughout
- [ ] Each section has substantive content

**Scoring guide:**
- 2: Problem is specific, goals are measurable outcomes, no vague language
- 1: Problem is stated but goals describe activities, or some vague language present
- 0: No clear problem statement, or goals are implementation tasks

---

### Dimension 2: Why (0-2) — Business Justification & Persona Coverage

*Is there a compelling motivation? Are all relevant personas addressed?*

Maps to original Clarity (motivation) + Coverage (personas). Check:
- [ ] Problem statement argues for the feature from the user's perspective
- [ ] **All four OSAC personas considered**: Cloud Provider Admin, Cloud Infrastructure Admin, Tenant Admin, Tenant User
- [ ] For each in-scope service, persona interactions are described
- [ ] User stories or use cases are present (even if informal)
- [ ] **Services declared**: Which services (BMaaS, CaaS, VMaaS, MaaS, Enclave) are in scope?

**Scoring guide:**
- 2: Compelling motivation, all relevant personas covered for in-scope services
- 1: Motivation present but some personas missing, or user perspective unclear
- 0: No user-facing motivation, or only one persona considered

---

### Dimension 3: How (0-2) — Requirements Specificity & OSAC Coverage

*Are the requirements concrete? Are relevant OSAC dimensions addressed?*

Maps to original Coverage (dimensions) + Scope (requirements). Check:
- [ ] Functional requirements are enumerable (each has a stable FR-N ID)
- [ ] Non-goals are specific, not vague ("Auto-scaling is out of scope" not "Advanced features")
- [ ] No scope creep signals ("and related functionality", "all necessary changes")
- [ ] **OSAC dimensions addressed** (where relevant):
  - Tenant onboarding (RBAC, IDP, auto-provisioned resources)
  - Inventory (backends, API integration)
  - Provisioning (mechanism, lifecycle stages)
  - Networking (backends, API-integrated vs side-channel)
  - Storage (prerequisites, automation, per-tenant)
  - Installation (Helm/kustomize, CI, osac-installer)
  - API resources (affected resources per service)
- [ ] Dependencies identified with ordering constraints
- [ ] Risks have owners and mitigations (not generic statements)

**Scoring guide:**
- 2: Requirements are specific and enumerable, relevant OSAC dimensions addressed
- 1: Requirements mostly clear but some gaps, some OSAC dimensions not mentioned
- 0: Requirements are vague, no FR-N IDs, OSAC dimensions ignored

---

### Dimension 4: Task (0-2) — Testability & Acceptance Criteria

*Can the requirements be verified? Are acceptance criteria concrete?*

Maps to original Testability dimension. Check:
- [ ] Each functional requirement (FR-N) is testable
- [ ] Acceptance criteria are concrete, verifiable conditions (checkboxes)
- [ ] Acceptance criteria cover the primary use cases
- [ ] Non-functional requirements are measurable ("API response under 200ms at p95" not "fast")
- [ ] Success metrics have targets and baselines (when included)
- [ ] No scope reduction language ("v2", "simplified", "placeholder")

**Scoring guide:**
- 2: Every requirement is testable, acceptance criteria are concrete assertions
- 1: Most requirements testable, some acceptance criteria are vague
- 0: Requirements describe activities, acceptance criteria are untestable

---

### Dimension 5: Size (0-2) — Scope Appropriateness & Completeness

*Is the PRD right-sized? Is it complete enough for the design phase?*

Maps to original Scope (boundaries) + overall completeness. Check:
- [ ] Target milestone is declared (e.g., 0.1, 0.2)
- [ ] 3-5 goals (more suggests scope is too broad)
- [ ] What's NOT covered is explicit — deferred capabilities listed as non-goals
- [ ] Non-goals prevent reasonable misinterpretations of scope
- [ ] Document has appropriate depth for a PRD (not too thin, not a design doc)
- [ ] All required PRD sections present with substantive content

**Scoring guide:**
- 2: Clear boundaries, explicit milestone, well-scoped, all sections present
- 1: Boundaries mostly clear, some non-goals could be more specific, minor gaps
- 0: Scope is unbounded, no milestone, missing major sections

---

## Severity Classification

Same as original `prd-review`:

- **Critical**: Missing required sections, no personas identified, scope unbounded, requirements untestable, OSAC dimensions completely ignored
- **Important**: Vague non-goals, missing personas, some dimensions not addressed, weak acceptance criteria, scope creep signals
- **Suggestion**: Style improvements, additional non-goals, deeper risk analysis, more specific metrics

## Notes

- Score based on what's in the PRD, not what you think should be there — "TBD" markers are acceptable
- The OSAC dimensions in the "How" criterion are a checklist, not a requirement — features that don't touch networking shouldn't lose points for not addressing networking
- Every check item from the original `prd-review` skill is preserved — only the scoring buckets differ

$ARGUMENTS
