---
title: PRD Template Support — User Stories
type: stories
date: 2026-03-04
topic: prd-template-support
prd: docs/prds/2026-03-04-prd-template-support-prd.md
---

# PRD Template Support — User Stories

## Epic Summary

This story set delivers project-level PRD template support for the `/pm:prd` command. TMPL-000 is a required prototype spike that must close before any implementation begins — its findings govern the implementation approach for template-driven structure and gap detection. TMPL-001 delivers template detection and the no-template regression path. TMPL-002 delivers template-driven PRD structure, consuming the spike findings for section parsing and heading fidelity. TMPL-003 delivers the gap-filling dialogue that prevents empty sections from reaching the output file. TMPL-004 is the integration and regression story that validates all three implementation stories together and produces explicit PM sign-off. Recommended implementation order: TMPL-000 → TMPL-001 → TMPL-002 → TMPL-003 → TMPL-004.

---

## Stories

---

### [TMPL-000] Prototype Spike — Agent Template Behavior

**As an** engineer preparing to implement PRD template support,
**I want** to run the `prd-writer` agent against 2–3 real-world template structures
**so that** I understand its actual behavior on ambiguous headings, unfillable sections, and nested subheadings before committing to an implementation approach.

**Acceptance Criteria:**
- [ ] The agent is run against at least 2 distinct `prd-template.md` structures. Structure (a) must include at least one company-specific section name that has no direct analogue in the brainstorm or product-context.yaml (e.g., "Legal Review," "Stakeholder Alignment"). Structure (b) must include at least one section with nested `###` subheadings.
- [ ] A findings document is written to `docs/fixtures/` or `docs/validation/` covering three questions: (1) Does the agent reliably place content under the correct heading, or does it conflate adjacent sections? (2) Under what conditions does the agent generate low-quality filler rather than recognizing it lacks context? (3) Are there heading naming patterns that cause section conflation?
- [ ] The findings document includes an explicit implementation recommendation for FR-002 (template-driven structure): specifically, how the prd-writer agent prompt should be modified to enforce section ordering and suppress content generation outside template headings.
- [ ] The findings document includes an explicit recommendation for FR-003 (gap detection): what observable signal indicates the agent lacks context for a section, and how the gap-filling dialogue should be triggered.
- [ ] The engineering team reviews the findings document and records a signed-off implementation decision (a comment, PR review, or annotation on the document) before this spike is marked closed.
- [ ] TMPL-001 and TMPL-002 are not started until this story is marked complete.

**Technical Notes:**
Affects `plugins/helm/agents/prd-writer.md` and `plugins/helm/commands/pm/prd.md` as read context for the spike. Findings inform prompt modifications — do not modify agent or command files as part of this story. Write fixture templates to `docs/fixtures/` alongside any existing fixtures.

**Dependencies:** None
**Out of scope:** Any code changes to prd-writer.md or prd.md; implementation of FR-001, FR-002, or FR-003; conclusions about edge cases not encountered during the spike runs.

---

### [TMPL-001] Template Detection

**As a** PM using Product Skill Helm,
**I want** `/pm:prd` to detect `prd-template.md` in the project root automatically at the start of each run
**so that** the template is used without any manual flag or configuration on my part, and projects without a template continue to work exactly as before.

**Acceptance Criteria:**
- [ ] When `prd-template.md` exists in the project root and `/pm:prd` runs, the command prints exactly: `Template found: prd-template.md — generating PRD using your template structure.` This message appears before any brainstorm is loaded and before any context is read.
- [ ] When `prd-template.md` does not exist in the project root, `/pm:prd` runs with no template-related output of any kind — no message, no warning, no mention of templates.
- [ ] Detection is file-presence-only: the command checks for the file path `prd-template.md` at the project root and takes the appropriate branch. No validation of the file's contents occurs in this story.
- [ ] The detection step runs as step 0 of the execution sequence in `prd.md`, before step 0 (research context load). The existing execution sequence from step 0 onward is unchanged.
- [ ] A run without `prd-template.md` produces output identical to a pre-feature run on the same inputs. No structural change, no performance degradation, no altered prompts.

**Technical Notes:**
Affects `plugins/helm/commands/pm/prd.md` only. Adds a template detection step at the top of the Execution section. The detection branch sets a "template mode" flag that TMPL-002 will act on; for this story, template mode simply prints the confirmation message and continues. No changes to `plugins/helm/agents/prd-writer.md`.

**Dependencies:** TMPL-000 (spike must close before implementation begins)
**Out of scope:** Parsing or reading the template file contents; template-driven output structure (TMPL-002); a `--no-template` bypass flag; template validation or error handling for malformed template files.

---

### [TMPL-002] Template-Driven PRD Structure

**As a** PM with a company PRD template,
**I want** the PRD generated by `/pm:prd` to follow my template's section order and headings exactly — with content under every heading and no extra sections added by the agent —
**so that** I can share the output with stakeholders without reformatting.

**Acceptance Criteria:**
- [ ] Every `##` heading from `prd-template.md` appears in the output PRD, in the same sequence as in the template. No `##` heading from the template is omitted.
- [ ] Every `###` subheading nested within a `##` template section appears in the output PRD under its parent section, in the same sequence as in the template. Agent-generated content appears under each `###` subheading.
- [ ] No `##` section appears in the output PRD that is not present in the template. The agent does not add its own default sections (e.g., "Overview," "Agent Consumption Notes") when a template is active.
- [ ] Body text under template headings that serves as placeholder or instructional scaffold (e.g., "Describe the problem here," "[Insert success criteria]") does not appear in the output PRD. The agent treats it as context, not as content to preserve.
- [ ] The section sequence in the output matches the template sequence exactly. A template with sections ordered A, B, C produces an output with sections ordered A, B, C — not reordered.
- [ ] A PM reviews the output against the template and confirms that no heading was renamed, reordered, or silently merged with an adjacent section.
- [ ] When no `prd-template.md` is present, the prd-writer agent uses its default PRD structure unchanged. This story introduces no changes to the default (no-template) output path.

**Technical Notes:**
Affects `plugins/helm/agents/prd-writer.md` (prompt modification to accept template structure as a structural constraint) and `plugins/helm/commands/pm/prd.md` (pass template contents to prd-writer agent when template mode is active). Implementation approach is informed by TMPL-000 spike findings. The prd-writer agent receives the parsed template as part of its context when spawned by `prd.md`.

**Dependencies:** TMPL-000 (implementation approach); TMPL-001 (template detection and template mode flag must exist)
**Out of scope:** Gap-filling dialogue when the agent lacks context for a section (TMPL-003); vocabulary or terminology harmonization between template language and agent output; template validation or error handling for malformed files; support for `####` or deeper nesting levels beyond `###`.

---

### [TMPL-003] Gap-Filling Dialogue

**As a** PM generating a PRD with a company template,
**I want** the agent to ask me directly when it cannot fill a section with meaningful content —
**so that** every section in the final PRD has substantive content or an explicit placeholder, and no section is silently left empty or padded with generic filler.

**Acceptance Criteria:**
- [ ] When the agent identifies one or more template sections it cannot fill with high-quality content (no relevant information available from brainstorm, product-context.yaml, or session context), it identifies all such sections before writing anything to the output file.
- [ ] All unfillable sections are surfaced to the PM in a single dialogue pass. The agent does not interleave gap questions with writing; it completes its gap identification first, then asks, then writes.
- [ ] For each unfillable section, the agent outputs exactly: `I don't have enough context to fill the **[Section Name]** section. Can you provide guidance, or would you like to skip it for now?` where `[Section Name]` is the exact `##` heading text from the template.
- [ ] When the PM provides substantive text in response to a gap question, that text is used as the section content (verbatim or lightly formatted for markdown). The section does not receive a placeholder.
- [ ] When the PM responds with "skip," "skip for now," "leave it blank," or any clear equivalent, the section is written as: `_[Section left blank — complete before sharing]_`
- [ ] When the PM provides substantive content for a gap section, the placeholder `_[Section left blank — complete before sharing]_` does not appear in that section.
- [ ] If the agent has sufficient context to fill all sections, no gap-filling dialogue occurs and the agent proceeds directly to writing the output file.
- [ ] Sections the agent fills with agent-generated content — without asking — do not receive a gap question and do not receive a placeholder.

**Technical Notes:**
Affects `plugins/helm/agents/prd-writer.md` (gap detection logic and dialogue format) and `plugins/helm/commands/pm/prd.md` (orchestration: wait for gap dialogue to complete before invoking the write step). Gap detection approach is informed by TMPL-000 spike findings. The exact threshold for "cannot fill with high-quality content" is defined by the spike findings and must be documented in the implementation plan.

**Dependencies:** TMPL-000 (gap detection approach); TMPL-001 (template detection); TMPL-002 (template-driven structure must exist before gap dialogue can be layered on top)
**Out of scope:** Gap-filling for the no-template (default) path; multi-turn gap dialogue (one PM response per section is sufficient); persistent session memory of PM gap responses across separate `/pm:prd` runs; gap detection for `###` subheadings — gap detection operates at the `##` section level only.

---

### [TMPL-004] End-to-End Integration and Regression

**As a** PM considering adopting PRD template support,
**I want** to confirm the full feature works end-to-end in a single run and that removing the template restores pre-feature behavior exactly —
**so that** I can adopt the feature with confidence and know it won't break projects that don't use a template.

**Acceptance Criteria:**
- [ ] End-to-end test: Create a `prd-template.md` with 5 or more `##` sections, including at least one company-specific section the agent has no context for (e.g., "Legal Review" or "Stakeholder Sign-Off"). Run `/pm:prd`. Confirm in a single run: (a) the template detection message appears; (b) every template `##` heading appears in the output in the correct sequence; (c) no non-template `##` sections appear in the output; (d) the gap-filling dialogue is triggered for the unfillable section and uses the exact format specified in TMPL-003; (e) the PM's response to the gap question is reflected correctly in the output.
- [ ] Regression test: Remove `prd-template.md` from the project root and run `/pm:prd` on the same inputs. Confirm: (a) no template-related message appears; (b) the output structure matches what the pre-feature prd-writer would produce; (c) no error is raised; (d) no structural difference from a pre-feature run can be identified.
- [ ] A PM who was not involved in implementation reviews the end-to-end test output and confirms: "I would share this PRD with stakeholders without reformatting it." This explicit sign-off is recorded as a comment, annotation, or PR review before the story is closed.
- [ ] All acceptance criteria from TMPL-001, TMPL-002, and TMPL-003 are verifiable as met from the artifacts produced in the end-to-end test run — no separate test runs are required to verify them.
- [ ] The end-to-end test and regression test are documented in `docs/validation/` with the date, inputs used, and results observed.

**Technical Notes:**
No new files are modified in this story. Touches `plugins/helm/commands/pm/prd.md` and `plugins/helm/agents/prd-writer.md` only as read context for validation. Test artifacts go to `docs/validation/`. PM sign-off is required to close this story.

**Dependencies:** TMPL-000, TMPL-001, TMPL-002, TMPL-003 (all must be complete)
**Out of scope:** Automated test infrastructure; CI/CD integration; performance benchmarking; template validation linting; testing against artifact types other than PRD.

---

## Implementation Order

1. **TMPL-000** — Run the prototype spike first. Its findings are the implementation contract for TMPL-001 and TMPL-002. No implementation story may begin until TMPL-000 is closed and signed off.
2. **TMPL-001** — Implement template detection next. It establishes the "template mode" control flow that TMPL-002 and TMPL-003 build on. Detection is a thin change to `prd.md` and can be verified independently before any agent changes.
3. **TMPL-002** — Implement template-driven PRD structure. Depends on TMPL-001 for the template mode flag and on TMPL-000 for the implementation approach for the prd-writer prompt.
4. **TMPL-003** — Implement gap-filling dialogue. Depends on TMPL-002 because gap detection operates within the template-driven output path; the dialogue layer sits on top of the structure established in TMPL-002.
5. **TMPL-004** — Run integration and regression validation last, after all implementation stories are complete. Produces the PM sign-off required to mark the feature shippable.
