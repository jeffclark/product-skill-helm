---
title: "feat: PRD Template Support"
type: feat
date: 2026-03-12
stories: docs/stories/2026-03-04-prd-template-support-stories.md
prd: docs/prds/2026-03-04-prd-template-support-prd.md
---

# ✨ feat: PRD Template Support

## Overview

Add support for a project-level `prd-template.md` file that governs the output structure of `/pm:prd`. When the file is present at the project root, the `prd-writer` agent generates content section-by-section according to the template, asks the PM to fill gaps it cannot cover from context, and produces a PRD that conforms to the company's template without any manual reformatting.

This is a markdown-only feature — no data models, no new API endpoints, no new files created (aside from the spike and validation documents). All changes are confined to two existing files in `plugins/helm/`.

---

## Problem Statement

PMs using Product Skill Helm at companies with defined PRD standards must reformat every generated PRD before sharing internally. The `prd-writer` agent uses its own default structure regardless of organizational conventions, creating a friction point that undermines the tool's core value proposition.

---

## Implementation Phases

### Phase 0: Prototype Spike (TMPL-000)

**Goal:** Establish the implementation contract for Phases 1–2 before any code is written.

**What to do:**

1. Create two fixture template files in `docs/fixtures/`:

   **`docs/fixtures/prd-template-fixture-a.md`** — company-specific sections:
   ```markdown
   ## Executive Summary

   ## Customer Problem

   ## Proposed Solution

   ## Legal Review

   ## Stakeholder Alignment

   ## Success Criteria

   ## Technical Dependencies
   ```

   **`docs/fixtures/prd-template-fixture-b.md`** — nested subheadings:
   ```markdown
   ## Overview

   ### Background
   ### Scope

   ## Requirements

   ### Functional Requirements
   ### Non-Functional Requirements

   ## Launch Plan

   ## Open Questions
   ```

2. Run a `/pm:prd` session for a real or simulated feature with each fixture as the active `prd-template.md`.

3. For each run, observe and document:
   - Does the agent place content under the correct `##` heading, or does it conflate adjacent sections?
   - Does the agent preserve `###` subheading order, or does it flatten/merge subheadings?
   - For sections it cannot fill (e.g., "Legal Review", "Stakeholder Alignment"), does it generate low-quality filler, leave them blank, or recognize the gap and ask?
   - Are there heading names that cause the agent to merge content across sections?

4. Write findings to `docs/validation/2026-03-12-tmpl-000-spike-findings.md` using the template below.

5. Based on findings, document the **implementation recommendation** for:
   - FR-002: How to pass template structure to `prd-writer` so it enforces section ordering and suppresses default sections
   - FR-003: What observable signal indicates "not enough context for this section" and how to trigger gap dialogue

**Spike findings document template:**
```markdown
---
title: TMPL-000 Spike Findings
date: YYYY-MM-DD
story: TMPL-000
status: complete
---

# TMPL-000 Spike Findings

## Run 1 — Fixture A (company-specific sections)
...observations...

## Run 2 — Fixture B (nested subheadings)
...observations...

## FR-002 Implementation Recommendation
...how prd-writer should receive and enforce template structure...

## FR-003 Gap Detection Recommendation
...what signal to use, how to trigger dialogue...

## Signed-off Implementation Decision
[Engineer signature/date]
```

**Gate:** TMPL-001 and TMPL-002 may not begin until spike findings are written and signed off.

---

### Phase 1: Template Detection (TMPL-001)

**Files changed:** `plugins/helm/commands/pm/prd.md` only

**What to add:**

Insert a new **pre-step** at the top of the Execution section in `prd.md`, before the existing "Step 0: Load research context":

```markdown
### Pre-step: Template Detection

Check for `prd-template.md` in the project root:

- **File found:** Print exactly:
  `Template found: prd-template.md — generating PRD using your template structure.`
  Set template mode active. Read the full contents of `prd-template.md` into session
  context as `template_contents`. Continue to Step 0.
- **File not found:** Continue to Step 0 with no message, no mention of templates.
  Template mode is inactive. All subsequent steps run exactly as before.
```

**Placement:** This step runs before `research-context.yaml` is loaded. It must be the first thing the command does.

**Key constraint:** Do NOT parse or validate the template file contents in this step — that happens in TMPL-002. This step is file-presence check + message + read only.

**Regression check:** After implementing, run `/pm:prd` without a template file present. Output must be byte-for-byte identical to a pre-TMPL-001 run on the same inputs.

---

### Phase 2: Template-Driven PRD Structure (TMPL-002)

**Files changed:** `plugins/helm/commands/pm/prd.md` + `plugins/helm/agents/prd-writer.md`

**Changes to `prd.md`:**

In the "Invoke prd-writer agent" step (currently Step 3), add a conditional branch:

```markdown
### 3. Invoke prd-writer agent

If template mode is active:
  Spawn `prd-writer` agent with all standard context PLUS:
  - `template_contents`: the full text of `prd-template.md`
  - `template_mode: true`
  The agent must use the template as its structural constraint (see prd-writer.md).

If template mode is inactive:
  Spawn `prd-writer` agent with standard context only (current behavior unchanged).
```

**Changes to `prd-writer.md`:**

Add a new section **before** the PRD Structure section:

```markdown
## Template Mode

When `template_mode: true` is passed and `template_contents` is provided:

### Structure rules (FR-002)

1. Parse every `##` heading from `template_contents` as a required output section.
   These are the ONLY `##` sections that may appear in the output PRD.
2. Preserve the heading sequence from the template exactly. Do not reorder.
3. Parse every `###` heading nested under each `##` section. Preserve these as
   subsections in the output, in the same sequence, under their parent `##` section.
4. Generate content under each heading. Do not use the default PRD Structure below
   when template mode is active — the template replaces it entirely.
5. Ignore any body text under template headings (placeholder/instruction text).
   Generate fresh content; do not preserve placeholder wording.
6. Do not add `##` sections not present in the template (e.g., do not append
   "Agent Consumption Notes" unless it appears in the template).

### Gap detection (FR-003)

Before writing any output:

1. For each `##` section, assess whether you have sufficient context from the
   brainstorm, product-context.yaml, and session context to generate high-quality
   content.
2. "Sufficient context" means you have specific, relevant information that directly
   addresses the section's purpose. Generic statements ("This section covers...") do
   not count as sufficient.
3. Collect all sections where you cannot generate high-quality content.
4. If any gap sections exist: surface them ALL to the PM in a single dialogue pass
   BEFORE writing any output. For each:
   > I don't have enough context to fill the **[Section Name]** section. Can you
   > provide guidance, or would you like to skip it for now?
   Wait for the PM's response before proceeding to the next gap section.
5. PM response handling:
   - Substantive text → use as section content
   - "skip", "skip for now", "leave it blank", empty string, or equivalent →
     write the section as: `_[Section left blank — complete before sharing]_`
6. After all gap responses are collected, write the complete output file.
7. If no gap sections exist: skip dialogue entirely and write the output directly.

### When template mode is inactive

Use the default PRD Structure defined below. No behavior change.
```

**Edge cases to handle in implementation:**

| Edge case | Correct behavior |
|-----------|-----------------|
| `prd-template.md` exists but is empty (no `##` headings) | Print warning: `Warning: prd-template.md has no ## headings — falling back to default PRD structure.` Proceed as no-template. |
| Template has only H1 (`#`) headings, no H2 (`##`) | Same as empty template — fallback to default. |
| Template headings match agent default headings exactly | Template headings take precedence. Sequence follows template, not agent default. |
| PM provides empty string response to gap question | Treat as "skip" — write the placeholder. |
| All sections have sufficient context | Skip gap dialogue entirely. Do not ask any questions. |

---

### Phase 3: Gap-Filling Dialogue (TMPL-003)

TMPL-003 is already covered by the `prd-writer.md` changes in Phase 2 (the gap detection section above). However, the **orchestration** side in `prd.md` needs one addition:

**Changes to `prd.md`:**

In the "Invoke prd-writer agent" step, add to the template-mode branch:

```markdown
If template mode is active:
  Spawn `prd-writer` agent and wait for it to complete gap identification.
  The agent will surface gap questions interactively before writing.
  Do NOT proceed to "Present draft and iterate" until the agent signals it has
  completed the gap dialogue and written the output file.
```

This ensures the command's orchestration layer waits for the full gap dialogue cycle before presenting the draft to the PM.

---

### Phase 4: Integration and Regression (TMPL-004)

**No code changes.** Validation only.

**What to do:**

1. Create test template `docs/fixtures/prd-template-integration-test.md`:
   ```markdown
   ## Executive Summary
   ## Problem Statement
   ## Goals
   ## Requirements
   ## Legal Review
   ## Success Criteria
   ## Open Questions
   ```
   (Includes "Legal Review" — a section the agent cannot fill from typical brainstorm context.)

2. Run `/pm:prd` with this template against a real or synthetic brainstorm.

3. Verify all TMPL-004 acceptance criteria:
   - [ ] Detection message appears exactly as specified
   - [ ] All 7 `##` headings appear in output in correct sequence
   - [ ] No non-template `##` sections in output
   - [ ] Gap dialogue fires for "Legal Review" with exact specified format
   - [ ] PM response reflected correctly in output

4. Regression test: rename/remove `prd-template.md`, re-run. Confirm no template output.

5. Write results to `docs/validation/2026-03-12-tmpl-004-integration.md`.

6. Get explicit PM sign-off: "I would share this PRD with stakeholders without reformatting it." Record in validation document.

---

## Files Changed

| File | Stories | Change Type |
|------|---------|-------------|
| `plugins/helm/commands/pm/prd.md` | TMPL-001, TMPL-002, TMPL-003 | Modify — add pre-step + template branch in Step 3 |
| `plugins/helm/agents/prd-writer.md` | TMPL-002, TMPL-003 | Modify — add Template Mode section before PRD Structure |
| `docs/fixtures/prd-template-fixture-a.md` | TMPL-000 | Create — spike fixture |
| `docs/fixtures/prd-template-fixture-b.md` | TMPL-000 | Create — spike fixture |
| `docs/fixtures/prd-template-integration-test.md` | TMPL-004 | Create — integration test fixture |
| `docs/validation/2026-03-12-tmpl-000-spike-findings.md` | TMPL-000 | Create — spike findings |
| `docs/validation/2026-03-12-tmpl-004-integration.md` | TMPL-004 | Create — integration results |

---

## Dependency Map

```
TMPL-000 (Spike)
    ↓ gates
TMPL-001 (Detection) ← modify prd.md (pre-step only)
    ↓
TMPL-002 (Structure) ← modify prd.md (Step 3 branch) + prd-writer.md (Template Mode section)
    ↓
TMPL-003 (Gap Dialogue) ← already in prd-writer.md from TMPL-002; add orchestration wait to prd.md
    ↓
TMPL-004 (Integration) ← validation + PM sign-off
```

---

## Acceptance Criteria Summary

- [x] TMPL-000: Spike findings written to `docs/validation/`, signed-off implementation decision present
- [x] TMPL-001: Detection message appears when template present; zero output when absent; regression clean
- [x] TMPL-002: All `##` headings present, in order, no non-template headings, `###` subheadings preserved; PM confirms no reformatting needed
- [x] TMPL-003: Gap dialogue fires in single pass before writing; exact format used; "skip" produces placeholder; no dialogue when all sections fillable
- [ ] TMPL-004: End-to-end run passes all above criteria in one run; regression test passes; PM sign-off recorded (pending live run)

---

## Open Questions (from PRD)

These are V1 exclusions. Note them here so the implementation doesn't accidentally scope them in:

1. **`--no-template` bypass flag** — Not in scope. If added later, it would be a new detection branch in the pre-step in `prd.md`.
2. **Challenger review + template section tension** — Not addressed. If a challenger flags a template-mandated section, the PM must resolve it manually.
3. **Default template scaffolding (`/pm:template init`)** — Not in scope. Would be a separate command.
4. **Vocabulary harmonization** — Explicitly V2. Do not include in prd-writer changes.

---

## References

- Stories: `docs/stories/2026-03-04-prd-template-support-stories.md`
- PRD: `docs/prds/2026-03-04-prd-template-support-prd.md`
- Brainstorm: `docs/brainstorms/2026-03-04-prd-template-support-brainstorm.md`
- prd command: `plugins/helm/commands/pm/prd.md`
- prd-writer agent: `plugins/helm/agents/prd-writer.md`
- CLAUDE.md: `plugins/helm/CLAUDE.md`
- Validation pattern: `docs/validation/2026-02-24-story-006-validation.md`
- Fixture pattern: `docs/fixtures/research-context-sample.yaml`
