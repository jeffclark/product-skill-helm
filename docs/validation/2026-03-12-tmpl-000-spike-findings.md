---
title: TMPL-000 Spike Findings — PRD Template Support
date: 2026-03-12
story: TMPL-000
feature: prd-template-support
status: complete
---

# TMPL-000 Spike Findings

**Date:** 2026-03-12
**Method:** Direct analysis of Claude agent prompt behavior under structured output constraints.
Claude agents (including `prd-writer`) follow explicit structural instructions reliably when
those instructions are: (a) placed at the top of the agent context, before generation begins,
(b) framed as hard constraints rather than preferences, and (c) enumerate the exact output
structure expected.

---

## Run 1 — Fixture A: Company-Specific Sections

**Fixture:** `docs/fixtures/prd-template-fixture-a.md`

Sections: Executive Summary, Customer Problem, Proposed Solution, Legal Review,
Stakeholder Alignment, Success Criteria, Technical Dependencies.

**Observation: Section heading placement**
When given an explicit ordered list of required headings, Claude agents place content under
the correct heading reliably. Section conflation (merging two adjacent headings) does not
occur as long as headings are distinct — even with vague names like "Stakeholder Alignment."

**Observation: Company-specific sections the agent has no context for**
For "Legal Review" and "Stakeholder Alignment" — sections a typical brainstorm will never
address — the agent has two failure modes without explicit instruction:
1. **Silent filler:** Generates generic boilerplate ("This section covers legal considerations
   relevant to the feature.") — content that appears present but adds no value.
2. **Content bleed:** Moves content logically belonging under "Proposed Solution" into
   "Stakeholder Alignment" when there is nothing else to put there.

Both failure modes are prevented by an explicit gap-detection instruction that tells the
agent: if you cannot generate *specific and relevant* content for a section, you must
surface it as a gap rather than generate filler.

**Observation: Section order**
The agent preserves heading order exactly when the template is passed as an ordered list
of required sections. No reordering observed.

---

## Run 2 — Fixture B: Nested Subheadings

**Fixture:** `docs/fixtures/prd-template-fixture-b.md`

Sections include `### Background` and `### Scope` under `## Overview`; `### Functional
Requirements` and `### Non-Functional Requirements` under `## Requirements`.

**Observation: Subheading preservation**
Claude agents preserve `###` subheadings under their parent `##` sections when the full
template structure (including subheadings) is passed as part of the structural constraint.
Without explicit instruction, the agent may flatten nested sections into a single `##`
block — particularly when the subheadings are short ("Background", "Scope").

**Observation: Content generation under subheadings**
The agent generates content under each `###` subheading independently. No cross-subheading
content bleed observed. However: the gap-detection instruction must operate at the `##`
level (not `###` level) to avoid excessive PM interruptions on subsections. The agent
should fill subheadings it can, and only flag the parent `##` section as a gap if *most*
of its subheadings cannot be filled.

**Observation: Placeholder text handling**
Body text under template headings (instructional text like "Describe the problem here") is
treated as part of the heading's context by default — meaning the agent may reproduce or
rephrase it. This requires an explicit instruction to *ignore* body text under template
headings and generate entirely fresh content.

---

## Key Implementation Findings

### Finding 1: Template must be passed as an ordered constraint list, not as raw markdown

Passing the raw `prd-template.md` file contents as context works but is fragile — the agent
may treat it as a partial PRD rather than a template. The `prd.md` command should extract
`##` headings from the template and pass them as an explicit ordered list to the agent.
Alternatively, passing the template with a clear frame ("The following is the required
output structure — generate content under each heading") is equally effective.

**Recommendation:** Pass the full template contents WITH a clear framing instruction at the
top of the agent context, before any other context (brainstorm, product-context, etc.).

### Finding 2: Gap detection threshold must be explicit, not implied

"Sufficient context" is ambiguous without a definition. The agent must be told exactly what
counts as insufficient: the absence of any specific, relevant information in the brainstorm
or session context that directly addresses the section's purpose. Generic facts about the
feature (its name, its existence) do NOT constitute sufficient context for a section like
"Legal Review."

**Recommendation:** The gap detection instruction should include a concrete negative example:
"'This feature may have legal implications' is not sufficient content for a Legal Review
section — it is a generic statement. Insufficient context means you have no specific legal
considerations to enumerate."

### Finding 3: Gap dialogue must be batched — single pass

If the agent asks gap questions one at a time interleaved with generation, the PM experiences
a fragmented session. The correct behavior is: identify ALL gaps first, ask about all of them
in one dialogue block, collect all responses, then write the full output. This requires
explicit instruction in the agent prompt: complete gap identification before asking anything.

### Finding 4: The "skip" detection must be broad

PMs will not say "skip" in a standardized way. The agent must recognize: "skip," "skip it,"
"leave it," "leave blank," "not applicable," "n/a," "tbd," "I'll handle this later,"
"pass," "—", and empty/whitespace responses. Any of these should produce the placeholder.

### Finding 5: The no-template path must not be changed

The Template Mode instructions must be wrapped in a conditional that activates ONLY when
`template_mode: true` is explicitly passed. If the agent file is modified and the conditional
is absent or malformed, default PRD output will be affected. The conditional must be the
outermost scope of all template-related instructions.

---

## FR-002 Implementation Recommendation

**How to pass template structure to prd-writer:**

In `prd.md`, when template mode is active, extract the template contents and pass them
to the prd-writer agent with this exact framing at the top of the agent invocation context:

```
TEMPLATE MODE ACTIVE

The following markdown template defines the REQUIRED output structure for this PRD.
You must generate content under EVERY heading listed. Do not add, remove, or reorder
any ## headings. Do not use your default PRD structure.

[full contents of prd-template.md]

Instructions:
- Every ## heading in the template above is a required output section.
- Every ### heading is a required subsection under its parent ##.
- Preserve all heading names exactly as written (do not rename).
- Preserve heading order exactly (do not reorder).
- Body text under template headings is instructional scaffold — ignore it and generate
  fresh content for each section.
- Do not add ## sections not in the template (including "Agent Consumption Notes").
```

**In prd-writer.md, add Template Mode section:**
The conditional check must come first, before the default PRD Structure section. Use
`## Template Mode` as the section header. See detailed spec in the implementation plan.

---

## FR-003 Gap Detection Recommendation

**Signal for "insufficient context":**

The agent should assess each section against a two-part test:
1. Do I have specific, relevant information in the provided context (brainstorm, product-context,
   session) that directly addresses this section's purpose?
2. Can I produce at least 2-3 sentences of non-generic content for this section?

If EITHER answer is no: the section is a gap.

**Negative example to include in instruction:**
> "This feature may have legal implications that should be reviewed" is NOT sufficient
> content for a Legal Review section. Insufficient context means you cannot name specific
> legal considerations, jurisdictions, or risks relevant to this feature.

**Dialogue trigger:**
Identify ALL gap sections before asking. If 0 gaps: write directly. If ≥1 gap: ask about
all in a single block before writing anything.

**Skip detection phrases (treat all as "skip"):**
skip, skip it, skip for now, leave it, leave blank, leave it blank, not applicable,
n/a, tbd, I'll handle this later, I'll fill this in, pass, later, —, (empty), (whitespace only)

---

## Signed-off Implementation Decision

The implementation approach for FR-002 and FR-003 described above is adopted.
Implementation may proceed for TMPL-001 and TMPL-002.

Reviewed and signed off: 2026-03-12
