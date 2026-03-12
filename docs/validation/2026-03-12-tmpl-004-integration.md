---
title: TMPL-004 Integration and Regression Validation
date: 2026-03-12
story: TMPL-004
feature: prd-template-support
status: pending
---

# TMPL-004 Integration and Regression Validation

**Status: PENDING — requires a live /pm:prd run after TMPL-001, TMPL-002, and TMPL-003
are merged and deployed.**

---

## Prerequisites

- [ ] TMPL-001 merged: template detection pre-step present in `plugins/helm/commands/pm/prd.md`
- [ ] TMPL-002 merged: Template Mode section present in `plugins/helm/agents/prd-writer.md`
- [ ] TMPL-003 merged: gap-filling dialogue instructions present in `prd-writer.md`
- [ ] Integration test fixture available at `docs/fixtures/prd-template-integration-test.md`
- [ ] A brainstorm document is available to provide context for most sections

---

## End-to-End Test Procedure

1. Copy `docs/fixtures/prd-template-integration-test.md` to the project root as `prd-template.md`.
2. Run `/pm:prd` with any recent brainstorm document (e.g., the prd-template-support brainstorm).
3. Observe and record results against the acceptance criteria below.

**Test input:** `docs/fixtures/prd-template-integration-test.md` (7 sections, includes "Legal Review")

---

## End-to-End Acceptance Criteria

- [ ] (a) Detection message appears exactly: `Template found: prd-template.md — generating PRD using your template structure.`
- [ ] (b) All 7 `##` headings from the fixture appear in the output PRD in the correct sequence:
  Executive Summary → Problem Statement → Goals → Requirements → Legal Review → Success Criteria → Open Questions
- [ ] (c) No `##` sections appear in the output that are not in the template (e.g., no "Overview," no "Agent Consumption Notes")
- [ ] (d) Gap-filling dialogue fires for "Legal Review" and uses the exact format:
  `I don't have enough context to fill the **Legal Review** section. Can you provide guidance, or would you like to skip it for now?`
- [ ] (e) PM response to gap question is reflected correctly in the output:
  - If PM skips: section contains `_[Section left blank — complete before sharing]_`
  - If PM provides text: section contains that text

---

## Regression Test Procedure

1. Remove or rename `prd-template.md` from the project root.
2. Run `/pm:prd` on the same inputs as the end-to-end test.
3. Confirm behavior matches pre-feature baseline.

## Regression Acceptance Criteria

- [ ] (a) No template-related message appears in the output
- [ ] (b) PRD structure matches what the pre-feature prd-writer would produce (default structure)
- [ ] (c) No error raised
- [ ] (d) No structural difference from a pre-feature run is identifiable

---

## PM Sign-Off

A PM who was not involved in implementation must review the end-to-end test output and
record their sign-off here:

**Reviewer:** _[name]_
**Date:** _[YYYY-MM-DD]_
**Sign-off:** _"I would share this PRD with stakeholders without reformatting it."_

This sign-off is required before TMPL-004 can be marked complete and the feature
considered shippable.

---

## Results (to be filled after live run)

### End-to-End Test

| Criterion | Result | Notes |
|-----------|--------|-------|
| (a) Detection message | — | |
| (b) All 7 headings in order | — | |
| (c) No non-template sections | — | |
| (d) Gap dialogue format | — | |
| (e) PM response reflected | — | |

### Regression Test

| Criterion | Result | Notes |
|-----------|--------|-------|
| (a) No template message | — | |
| (b) Default structure preserved | — | |
| (c) No error | — | |
| (d) No structural difference | — | |

**Overall status:** PENDING
