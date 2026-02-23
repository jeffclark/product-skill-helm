---
title: Named Persona Panel for PM Challenger — PRD
type: prd
date: 2026-02-23
topic: persona-panel-challenger
status: draft
---

# Named Persona Panel for PM Challenger

## Overview

The PM Challenger currently reviews every artifact as an anonymous voice. This feature replaces the anonymous challenger with a named panel of product leaders — Paul Graham, Marty Cagan, Steve Jobs, and Jeff Bezos — each assigned to specific artifact phases. Each persona asks 2–3 targeted questions before issuing their review, turning a skipped checklist into an active conversation.

## Problem Statement

PMs using the Helm plugin skip challenger reviews at high rates because the current challenger has no identity and no warmup — it fires flags without context, and there is no cost to ignoring it. A PM who skips a Cagan review is skipping a known framework with a known standard. A PM who skips an anonymous review is skipping a widget. The fix is not more flags; it is more identity. Named personas make the review legible, make skipping it feel like a real decision, and give PMs a frame for understanding what kind of objection they are looking at.

## Target User

PM or product-adjacent practitioner using the Helm plugin to generate artifacts (brainstorms, PRDs, stories, GTM plans, analytics plans). Moderate-to-high frequency user. Familiar with product development conventions but not necessarily with each persona's philosophy in detail. The user values speed but also values shipping the right thing — they are not adversarial to the review; they are just insufficiently engaged with it.

## Goals

- PMs read and engage with challenger reviews instead of skipping them, evidenced by a rise in override statements (which require reading the flags to write).
- A PM reading a Cagan review or a Graham review knows exactly what lens is being applied and what the reviewer cares about.
- Adding or swapping a persona requires editing or adding one file. No changes to SKILL.md or commands are needed.

## Non-Goals

- Instrumentation, analytics, or telemetry on review engagement rates.
- New artifact phases beyond the existing five (brainstorm, PRD, stories, GTM, analytics).
- New flag categories beyond the existing four (SCOPE CREEP, WEAK RATIONALE, UNDER-DEFINED REQUIREMENTS, METRIC RISK).
- Changes to flag format structure, flag IDs, or override syntax.
- Personas that block artifact generation — the challenger still does not refuse.

## Panel Assignment

| Phase | Reviewer(s) | Primary lens |
|---|---|---|
| Brainstorm | Paul Graham | Is this worth building? Are you talking to users? Are you moving fast enough? |
| PRD | Marty Cagan | Are requirements clear? Has discovery happened? What is the riskiest assumption? |
| PRD (co-reviewer) | Steve Jobs | Is this simple? What would you cut? Are you saying ten things when you should say one? |
| Stories | Marty Cagan | Are engineers solving problems or building features? Are acceptance criteria measurable? |
| GTM | Paul Graham | What is the scrappy, unscalable way to get first users? Is distribution part of the product? |
| Analytics | Jeff Bezos | What is the data? What is the smallest experiment? What is the mechanism? |

**Jobs scope constraint:** Jobs reviews PRDs for clarity and simplicity only. He does not block on quality or perfectionism. His flag category is strictly: complexity masquerading as sophistication, bundled scope, and fuzzy language. His scope does not overlap with Cagan's. PMs respond to each reviewer independently.

## Interaction Model

### Q&A Protocol

Before issuing a formal review, the assigned persona asks 2–3 targeted questions to surface gaps and challenge assumptions. The questions are written in the persona's voice and reflect their philosophy. The Q&A is the intake; the structured flag review is the output that follows after the PM responds.

The questions are not generic. Graham asks about user conversations and time-to-ship. Cagan asks about discovery and risk. Jobs asks what can be cut. Bezos asks what the mechanism is and how small the first test can be.

### Skip Flow

The PM may type "skip" at the Q&A prompt to bypass questions and go directly to the review. When skipped, the persona responds in their own voice — one short paragraph explaining in character why the questions mattered and what they were trying to surface, as though speaking directly to the PM who just declined to engage. The tone is genuine, not punitive. Then the persona confirms the skip and issues the formal review using available context.

The skip response must feel like a real human reaction: telling Paul Graham you appreciate his time and are moving on, and having him respond authentically before stepping aside.

### Override Protocol

Unchanged from the current SKILL.md. When a PM overrides a flag, the persona acknowledges the override in one line and proceeds. Overridden flags are not re-raised in the same session for the same topic.

```
Override accepted: prd_001 — splitting is blocked by Q3 deadline. Noted.
```

## Architecture

```
skills/pm-challenger/SKILL.md                          # Orchestration layer
skills/pm-challenger/references/persona-graham.md      # Paul Graham persona
skills/pm-challenger/references/persona-cagan.md       # Marty Cagan persona
skills/pm-challenger/references/persona-jobs.md        # Steve Jobs persona (with scope constraints)
skills/pm-challenger/references/persona-bezos.md       # Jeff Bezos persona
```

**SKILL.md** contains: phase-to-persona mapping table, Q&A protocol rules, skip flow rules, review format rules, and override rules. It does not contain persona-specific content.

**Each persona file** contains: the persona's philosophy (3–5 sentences), their signature question types, the flag categories they apply and their characteristic angle on each, example review language in their voice, and their skip response template.

Adding a persona = creating one new file and adding one row to the mapping table in SKILL.md. Swapping a persona = replacing the file and updating the mapping row.

## Functional Requirements

**FR-001:** SKILL.md shall include a phase-to-persona mapping table that specifies, for each artifact phase, which persona file(s) to load before running the challenger review.

**FR-002:** Before issuing a challenger review for any artifact phase, the system shall load the assigned persona file and generate 2–3 questions written in that persona's voice, derived from the persona's philosophy and the PM's input, before any flags are issued.

**FR-003:** The system shall present the Q&A questions and wait for the PM's response before generating the formal challenger review. The review shall incorporate the PM's answers as additional context.

**FR-004:** When a PM types "skip" at the Q&A prompt, the system shall generate a skip response in the persona's voice — one paragraph, in character, explaining what the questions were trying to surface and why they mattered — then confirm the skip and proceed to the formal review using available context only.

**FR-005:** For the PRD phase, the system shall run two reviewers in parallel: Marty Cagan and Steve Jobs. Each reviewer shall produce a separate, clearly labeled review section. Flag IDs shall be namespaced to distinguish the two reviewers (e.g., `cagan_prd_001`, `jobs_prd_001`). The PM shall address each reviewer's flags independently.

**FR-006:** Jobs's review scope shall be constrained to three flag types only: complexity masquerading as sophistication, bundled scope, and fuzzy language. Jobs shall not raise flags in categories outside this scope. This constraint shall be documented in `persona-jobs.md` and enforced by the instructions in that file.

**FR-007:** Each persona file shall specify the flag categories that persona applies and their characteristic angle. The four flag categories (SCOPE CREEP, WEAK RATIONALE, UNDER-DEFINED REQUIREMENTS, METRIC RISK) remain unchanged. Personas apply them through their own lens; they do not introduce new categories.

**FR-008:** The override protocol shall remain unchanged. When a PM overrides a flag, the currently active persona shall acknowledge the override in one line (in the persona's voice) and proceed. The system shall not re-raise overridden flags in the same session for the same topic.

**FR-009:** If no genuine issues exist after Q&A, the persona shall output a clean pass in their own voice, consistent with the existing no-flags format, rather than inventing flags to fill the structure.

**FR-010:** The challenger review header shall identify the active persona by name:

```
CHALLENGER REVIEW — [ARTIFACT TYPE]
Reviewer: [Persona Name]
```

For dual-reviewer PRDs, each section shall carry its own reviewer header.

**FR-011:** Each persona file shall include a skip response template written in that persona's voice, used by FR-004. The template shall be written as a direct address to the PM — first person, specific to what that persona's questions would have uncovered, no longer than 3–5 sentences.

**FR-012:** SKILL.md shall document the Q&A protocol, skip flow, dual-reviewer PRD behavior, and override protocol as orchestration rules, without containing any persona-specific language, questions, or skip responses. All persona content shall live exclusively in the persona files.

## Success Criteria

- [ ] Each of the five artifact phases triggers the correct assigned persona(s) as specified in the panel assignment table.
- [ ] For each phase, the system generates 2–3 pre-review questions in the persona's voice before issuing flags.
- [ ] A PM typing "skip" receives an in-character skip response from the assigned persona before the review is issued.
- [ ] PRD phase produces two separate, labeled review sections — one from Cagan, one from Jobs — with non-overlapping flag scopes.
- [ ] Jobs raises no flags outside the three categories specified in his scope constraint (complexity masquerading as sophistication, bundled scope, fuzzy language).
- [ ] Override acknowledgment is issued in the active persona's voice and does not re-raise the overridden flag in the same session.
- [ ] Adding a new persona requires creating one file and updating one row in SKILL.md, with no other files modified.
- [ ] A PM reading any challenger review can identify the reviewer by name and understand the philosophical lens being applied without consulting external documentation.

## Decisions

- **Cagan dual role:** One `persona-cagan.md` file serves both PRD and Stories phases. Phase context is passed at invocation so questions and emphasis can vary by phase without duplicating the file.
- **Question generation:** Questions are generated dynamically from the PM's input in the persona's voice. Persona files provide philosophy, signature concerns, and question framing — not a fixed bank.
- **Dual reviewer Q&A:** Shared Q&A session at PRD phase. The PM answers once; both Cagan and Jobs use that context for their respective reviews. No separate Q&A per reviewer.

## Agent Consumption Notes

- Data model changes expected: No.
- New API endpoints: No.
- UI changes: No.
- New files: `skills/pm-challenger/references/persona-graham.md`, `persona-cagan.md`, `persona-jobs.md`, `persona-bezos.md` (four new files). `skills/pm-challenger/SKILL.md` is modified to add phase-to-persona mapping, Q&A protocol, skip flow, and dual-reviewer rules.
- Third-party integrations: No.
- Permissions/roles affected: No.
- Primary implementation surface: Markdown instruction files read by the LLM at skill invocation time. No code changes. All behavior is governed by the content of SKILL.md and the four persona files.
- Key constraint for agents: Jobs's scope is hard-constrained in `persona-jobs.md`. The implementation must make this constraint explicit enough that the LLM does not generalize Jobs's review beyond the three specified flag types, even when genuine issues outside his scope exist.
