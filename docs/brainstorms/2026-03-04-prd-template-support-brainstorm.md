---
date: 2026-03-04
topic: prd-template-support
---

# PRD Template Support

## What We're Building

Product Skill Helm generates PRDs using its built-in `prd-writer` agent, but organizations using the tool have their own PRD standards — section naming, ordering, required fields — and the generated output doesn't match. This forces PMs to reformat every PRD before sharing internally, defeating one of the tool's core value propositions.

We're adding support for a project-level PRD template file (`prd-template.md`). When this file is present in the project root, the `/pm:prd` command passes it to the `prd-writer` agent as a structural constraint. The agent generates content section-by-section according to the template. When it lacks sufficient context to fill a section, it asks the PM directly. The PM can provide content or explicitly mark the section as "skip for now." The final output is the template's structure, fully populated.

## Why This Approach

**Three approaches considered:**

- **Approach A (selected): Template-Aware PRD Command** — `prd-template.md` in the project root; `prd-writer` uses it as a structural constraint; interactive gap-filling for unknown sections.
- **Approach B: Template Init + Validation Pass** — Scaffold command plus post-generation validation. Rejected: two-pass approach creates rework within the tool before it even exits, and doesn't solve the "agent has no content for a section" problem natively.
- **Approach C: Template in product-context.yaml** — Template sections as YAML config. Rejected: product-context.yaml is project metadata, not content structure. Harder to edit for non-technical PMs.

Approach A was selected because it requires no new infrastructure, degrades gracefully when no template is present, and the interactive gap-filling model is a natural extension of how the tool already works.

## Key Decisions

- **Template format is markdown:** PMs can read, edit, and version-control a `.md` file directly. No custom schema required.
- **Template lives in the project repo:** It travels with the product-context.yaml and gets committed alongside the project, ensuring consistency across the team.
- **Every section gets content:** The agent must attempt every section. If it lacks confidence, it asks the PM. "Skip for now" is a valid PM answer — the section is left with a placeholder rather than silently omitted.
- **Graceful degradation:** If no `prd-template.md` is present, `/pm:prd` behaves exactly as it does today. No breaking changes.
- **V1 scope is structure only:** Heading-level conformance and content completeness. Vocabulary harmonization (matching company-specific terminology) is explicitly out of scope for this version.

## Resolved Challenges

**[brainstorm_001] UNDER-DEFINED REQUIREMENTS — Resolved**
Clarified exactly what "using the template" means: every top-level heading in the template appears in the output in the same order; agent content is placed under the matching heading; sections the agent cannot fill trigger a PM question; "skip for now" is accepted and produces a placeholder.

**[brainstorm_002] WEAK RATIONALE (research-sourced) — Resolved**
ChatPRD positions its template library as a core differentiator including vocabulary/terminology alignment. For v1, we are explicitly scoping to heading-structure and content completeness only. Terminology harmonization is a v2 consideration. This decision is documented so PMs don't expect vocabulary matching and are not surprised.

## Open Questions

- Should `prd-template.md` be auto-detected silently, or should the PM be notified at the start of `/pm:prd` that a template was found and will be used?
- Should there be a way to bypass the template for a specific run (e.g., `--no-template` flag)?
- How should the challenger review interact with template-constrained sections? If the template has a "Legal Review" section and the challenger flags it as out of scope, does the PM still fill it?
- Is there a need for a default/sample `prd-template.md` that the tool can scaffold, or is it purely bring-your-own?

## Next Steps

→ Run `/pm:prd` to define requirements
