---
title: Named Persona Panel for PM Challenger — Stories
type: stories
date: 2026-02-23
topic: persona-panel-challenger
prd: docs/prds/2026-02-23-persona-panel-challenger-prd.md
status: draft
---

# Named Persona Panel for PM Challenger — Stories

## Summary

**Total stories:** 9
**Epic breakdown:**
- EPIC-1 (Skill orchestration): 1 story — SKILL.md update to add phase-to-persona mapping, Q&A protocol, skip flow, dual-reviewer rules, and override voice rules. Moderate complexity: touches every phase trigger and the override protocol.
- EPIC-2 (Persona file template): 1 story — Define and document the structural skeleton that all persona files must follow. Low complexity; foundational for EPIC-3.
- EPIC-3 (Persona files): 4 stories — One per persona (Graham, Cagan, Jobs, Bezos). Medium complexity each; Jobs carries the highest complexity due to hard scope constraint enforcement.
- EPIC-4 (Dual-reviewer PRD behavior): 1 story — Orchestration rules for running Cagan and Jobs in parallel at the PRD phase with namespaced flag IDs and shared Q&A.

**Additional cross-cutting stories:** 2 — Q&A protocol behavior (FR-002/FR-003) and skip flow behavior (FR-004) are tested across personas and are documented as integration-level acceptance stories once persona files and SKILL.md updates exist.

**Estimated complexity notes:**
- All 9 stories are independently implementable in parallel. No story requires another story's output to exist before it can be started or reviewed.
- The only soft sequencing recommendation is that the persona file template (EPIC-2) be authored before persona files are finalized (EPIC-3), so authors can use the template as a scaffold. It is not a hard dependency — persona files can be drafted and then validated against the template.
- EPIC-4 may be authored in parallel with EPIC-1; the dual-reviewer rules live in SKILL.md but are tested against the Cagan and Jobs persona files.

---

## Epic Summary

This story set delivers the full Named Persona Panel for the PM Challenger skill. EPIC-1 and EPIC-2 establish the orchestration rules and structural contract that govern how personas are loaded and invoked. EPIC-3 delivers the four persona files — Graham, Cagan, Jobs, and Bezos — each a self-contained instruction document that the LLM reads at review time. EPIC-4 specifies the dual-reviewer behavior unique to the PRD phase. Together, these stories replace the anonymous challenger with a named panel that asks questions, responds to skips, acknowledges overrides in character, and issues reviews under a named byline. Recommended authoring order: EPIC-2 first (establishes the template), then EPIC-1 and EPIC-3 in parallel, then EPIC-4 last (validates the dual-reviewer rules against the Cagan and Jobs files).

---

## Stories

---

### [EPIC-1] STORY-001: Update SKILL.md with phase-to-persona mapping and orchestration rules

**As a** PM using any Helm artifact command,
**I want** the challenger skill to load the correct named persona for my artifact phase and follow documented protocols for Q&A, skip flow, dual-reviewer behavior, and override acknowledgment,
**so that** every review is consistently attributed to a named reviewer and follows a predictable interaction model regardless of which artifact I am generating.

**Acceptance Criteria:**
1. SKILL.md contains a phase-to-persona mapping table that identifies, for each of the five artifact phases (brainstorm, PRD, stories, GTM, analytics), the persona file(s) to load before running the challenger review.
2. The mapping table entries are: brainstorm → `persona-graham.md`; PRD → `persona-cagan.md` AND `persona-jobs.md`; stories → `persona-cagan.md`; GTM → `persona-graham.md`; analytics → `persona-bezos.md`.
3. SKILL.md documents the Q&A protocol rule: before issuing flags, load the assigned persona file and generate 2–3 questions in that persona's voice derived from their philosophy and the PM's input.
4. SKILL.md documents the skip flow rule: when the PM types "skip," output the persona's skip response (sourced from the persona file), then confirm the skip and proceed to the formal review using available context only.
5. SKILL.md documents the dual-reviewer rule: at the PRD phase, run a single shared Q&A session, then produce two separate labeled review sections — one for Cagan, one for Jobs — each with its own reviewer header and namespaced flag IDs.
6. SKILL.md documents the override acknowledgment rule: override acknowledgment is issued in the active persona's voice (as defined in the persona file), not as a generic line.
7. SKILL.md contains no persona-specific language, example questions, skip response text, or flag examples attributed to any named persona. All persona content resides exclusively in the persona files.
8. The existing flag format structure, flag ID convention, four flag categories, maximum 5 flags per review, and no-flags clean pass format are preserved in SKILL.md unchanged.
9. The pattern references section at the bottom of SKILL.md is preserved unchanged.
10. A reader of SKILL.md can determine, without consulting any other file, which persona(s) are active for any given artifact phase, what the Q&A protocol steps are, and what to do when a PM types "skip."

**Files affected:**
- `skills/pm-challenger/SKILL.md` (modified)

**Dependencies:** None
**Parallelizable:** Yes
**Out of scope:** Persona file content; dual-reviewer Q&A question generation (governed by persona files once loaded); Jobs scope constraint enforcement (lives in `persona-jobs.md`).

---

### [EPIC-2] STORY-002: Define the persona file template

**As an** engineering agent or content author adding a new persona to the panel,
**I want** a documented structural template that every persona file must follow,
**so that** persona files are consistent, predictable for the LLM to parse, and straightforward to create or extend without reverse-engineering existing files.

**Acceptance Criteria:**
1. A persona file template document exists at `skills/pm-challenger/references/persona-template.md`.
2. The template defines the following required sections, in order, with a one-line description of what each section must contain: (a) persona identity (name, role, 3–5 sentence philosophy summary), (b) signature question types (the categories of concern this persona probes — not fixed questions, but framing patterns), (c) flag categories and characteristic angle (which of the four flag categories this persona applies and their distinctive interpretation), (d) example review language (2–3 short illustrative phrases or sentences in the persona's voice, demonstrating tone and vocabulary), (e) skip response template (one paragraph, first person, direct address to the PM, 3–5 sentences, explaining what the questions were trying to surface and why they mattered), (f) override acknowledgment line (one line, in the persona's voice, following the format "Override accepted: [flag_id] — [reason]. [persona reaction].").
3. The template marks every section as required, with no optional sections.
4. The template includes a note stating that persona files must not contain content that belongs in SKILL.md (orchestration rules, flag format, override protocol mechanics).
5. The template includes a note stating that all persona-specific content — including skip response and override acknowledgment — must live in the persona file, not in SKILL.md.
6. The template does not include fictional placeholder content that could be mistaken for a real persona file; it uses labeled placeholders (e.g., `[PERSONA NAME]`, `[INSERT PHILOSOPHY]`).
7. A content author reading only this template can produce a new persona file that is structurally compatible with how SKILL.md expects to load and invoke persona files.

**Files affected:**
- `skills/pm-challenger/references/persona-template.md` (created)

**Dependencies:** None
**Parallelizable:** Yes
**Out of scope:** Validation tooling or linting for persona file compliance; documentation beyond the template file itself.

---

### [EPIC-3] STORY-003: Author persona file for Paul Graham

**As a** PM running a brainstorm or GTM artifact,
**I want** the challenger to review my work through Paul Graham's lens — focused on whether this is worth building, whether I have talked to users, and whether I am moving fast enough,
**so that** the review surfaces early-stage viability problems in a voice I recognize and can engage with directly.

**Acceptance Criteria:**
1. `skills/pm-challenger/references/persona-graham.md` exists and follows the persona file template structure defined in STORY-002.
2. The philosophy section captures Graham's core stances in 3–5 sentences: start with something users want, talk to users directly, do things that don't scale first, move faster than you think you need to, and most ideas fail because the founder didn't talk to users early enough.
3. The signature question types section describes Graham's characteristic question patterns: probing whether the PM has direct user evidence (not survey data or market research); challenging whether the initial version is too large to ship in weeks; testing whether the team has found their first 10 users yet and what those users said.
4. The flag categories section specifies the angle Graham applies to each relevant category: SCOPE CREEP (are you building more than you need to find out if anyone wants this?); WEAK RATIONALE (is this based on user conversations or assumption?); UNDER-DEFINED REQUIREMENTS (are you over-specifying before you know what users actually need?). Graham does not apply METRIC RISK as a primary lens.
5. The example review language section contains 2–3 illustrative phrases in Graham's voice: direct, plain-spoken, slightly impatient with abstraction, focused on doing and learning rather than planning.
6. The skip response template is written in Graham's voice: first person, direct address to the PM, 3–5 sentences, specific to what his questions were trying to uncover (user evidence, speed, scope), no longer punitive.
7. The override acknowledgment line is in Graham's voice: direct, one line, does not re-litigate the flag.
8. The file contains no orchestration rules, flag format instructions, or content that belongs in SKILL.md.

**Files affected:**
- `skills/pm-challenger/references/persona-graham.md` (created)

**Dependencies:** None (STORY-002 is a soft dependency for structural conformance, not a hard blocker)
**Parallelizable:** Yes
**Out of scope:** Graham's venture investing opinions, Y Combinator-specific advice, or content not applicable to product artifact review.

---

### [EPIC-3] STORY-004: Author persona file for Marty Cagan

**As a** PM running a PRD or stories artifact,
**I want** the challenger to review my work through Marty Cagan's lens — focused on whether genuine product discovery happened, whether the riskiest assumptions are identified, and whether engineers are being given problems to solve rather than features to build,
**so that** the review pushes me toward outcome-driven, discovery-backed artifacts rather than output-driven feature lists.

**Acceptance Criteria:**
1. `skills/pm-challenger/references/persona-cagan.md` exists and follows the persona file template structure defined in STORY-002.
2. The philosophy section captures Cagan's core stances in 3–5 sentences: the root cause of most failed products is building what was asked for without validating value, feasibility, and usability first; product discovery is not optional; engineers are partners in discovery, not ticket-takers; outcome matters more than output.
3. The signature question types section distinguishes between phase contexts: when invoked for the PRD phase, Cagan probes discovery evidence, assumption risk, and testability of success criteria; when invoked for the stories phase, Cagan probes whether engineers are being given measurable problems or prescribed solutions, and whether acceptance criteria are outcome-based.
4. The flag categories section specifies the angle Cagan applies: SCOPE CREEP (are multiple unvalidated bets bundled into one delivery?); WEAK RATIONALE (has this been validated with users, or is it HiPPO-driven?); UNDER-DEFINED REQUIREMENTS (can an engineer build this without making assumptions about behavior, edge cases, or error states?); METRIC RISK (will this metric tell you if the product is working, or just that it shipped?).
5. The example review language section contains 2–3 illustrative phrases in Cagan's voice: direct, framework-grounded, focused on discovery gaps and outcome clarity.
6. The skip response template is written in Cagan's voice: first person, direct address to the PM, 3–5 sentences, referencing specifically what his questions were trying to uncover (discovery evidence, assumption validation, outcome measurability).
7. The override acknowledgment line is in Cagan's voice: direct, one line, does not re-litigate the flag.
8. The file contains no orchestration rules, flag format instructions, or content that belongs in SKILL.md.
9. A single `persona-cagan.md` file serves both PRD and Stories phases without duplication; phase context is applied at invocation time based on the artifact type passed by SKILL.md.

**Files affected:**
- `skills/pm-challenger/references/persona-cagan.md` (created)

**Dependencies:** None (STORY-002 is a soft dependency for structural conformance, not a hard blocker)
**Parallelizable:** Yes
**Out of scope:** Cagan's organizational change or coaching content; leadership/management topics unrelated to artifact review.

---

### [EPIC-3] STORY-005: Author persona file for Steve Jobs (scope-constrained)

**As a** PM running a PRD artifact,
**I want** the challenger to review my PRD through Steve Jobs's lens — focused strictly on whether the product is over-complicated, whether scope has been bundled unnecessarily, and whether the language is fuzzy or precise,
**so that** the review surfaces simplicity and clarity problems that Cagan's discovery-focused lens would not catch, without overlapping into discovery, quality, or requirements completeness.

**Acceptance Criteria:**
1. `skills/pm-challenger/references/persona-jobs.md` exists and follows the persona file template structure defined in STORY-002.
2. The philosophy section captures Jobs's relevant stances in 3–5 sentences: simplicity is not the absence of features but the presence of clarity; sophistication masquerading as complexity is a design failure; the best product decisions are subtractive; one thing done perfectly outperforms ten things done adequately.
3. The signature question types section describes Jobs's characteristic patterns: probing whether each listed requirement is truly necessary or could be cut; challenging whether the feature's description is clear to a non-expert; testing whether the product tries to do too many things at once.
4. The flag categories section specifies Jobs's hard scope constraint explicitly and unambiguously: Jobs raises flags in only three categories — (a) complexity masquerading as sophistication (a requirement or feature that is complicated without being powerful), (b) bundled scope (multiple independent jobs combined in a way that dilutes each), (c) fuzzy language (terms that cannot be defined precisely, leaving interpretation to engineers). Jobs does not apply WEAK RATIONALE or METRIC RISK flags. Jobs does not raise SCOPE CREEP flags that overlap with Cagan's scope or discovery concerns.
5. The hard scope constraint is stated as an instruction to the LLM in the first person or as a directive: Jobs must not raise flags outside the three specified categories, even when genuine issues in other categories are present in the PRD.
6. The example review language section contains 2–3 illustrative phrases in Jobs's voice: spare, declarative, occasionally blunt, focused on what should be cut rather than what should be added.
7. The skip response template is written in Jobs's voice: first person, direct address to the PM, 3–5 sentences, referencing what his review was looking for (over-complexity, bundled work, imprecise language), unambiguously in character.
8. The override acknowledgment line is in Jobs's voice: one line, terse, does not re-litigate the flag.
9. The file contains no orchestration rules, flag format instructions, or content that belongs in SKILL.md.
10. A reader of `persona-jobs.md` can determine without ambiguity that Jobs will not flag issues outside his three categories, even if the PRD has significant discovery or quality gaps.

**Files affected:**
- `skills/pm-challenger/references/persona-jobs.md` (created)

**Dependencies:** None (STORY-002 is a soft dependency for structural conformance, not a hard blocker)
**Parallelizable:** Yes
**Out of scope:** Jobs's hardware design philosophy, historical Apple product decisions, or biographical content not applicable to artifact review behavior.

---

### [EPIC-3] STORY-006: Author persona file for Jeff Bezos

**As a** PM running an analytics artifact,
**I want** the challenger to review my analytics plan through Jeff Bezos's lens — focused on whether the data mechanism is specified, whether the experiment is sized correctly, and whether the metric will actually reveal the mechanism at work,
**so that** the review surfaces analytical rigor gaps in a voice grounded in working backward, mechanism thinking, and small-bet experimentation.

**Acceptance Criteria:**
1. `skills/pm-challenger/references/persona-bezos.md` exists and follows the persona file template structure defined in STORY-002.
2. The philosophy section captures Bezos's relevant stances in 3–5 sentences: work backward from the customer outcome; the smallest experiment that produces a real signal is more valuable than the largest study that produces a proxy; data is useful when you understand the mechanism it measures; if you cannot describe the mechanism, you do not understand the product.
3. The signature question types section describes Bezos's characteristic patterns: demanding that the PM identify the causal mechanism the metric is meant to track; probing whether the experiment is sized to the question (not over-built); challenging whether the proposed metric is a leading indicator or a lagging vanity number.
4. The flag categories section specifies the angle Bezos applies: METRIC RISK (does this metric measure the mechanism, or just activity?); WEAK RATIONALE (is the analytics plan working backward from a customer outcome, or forward from available data?); UNDER-DEFINED REQUIREMENTS (can the experiment be run without knowing what a result looks like in advance?). Bezos applies SCOPE CREEP sparingly — only when the analytics plan is measuring so many things that no signal will be actionable.
5. The example review language section contains 2–3 illustrative phrases in Bezos's voice: methodical, mechanism-focused, willing to reduce scope to the irreducible experiment, occasionally Socratic.
6. The skip response template is written in Bezos's voice: first person, direct address to the PM, 3–5 sentences, specific to what his questions were trying to uncover (mechanism clarity, experiment sizing, metric validity), in character.
7. The override acknowledgment line is in Bezos's voice: one line, direct, does not re-litigate the flag.
8. The file contains no orchestration rules, flag format instructions, or content that belongs in SKILL.md.

**Files affected:**
- `skills/pm-challenger/references/persona-bezos.md` (created)

**Dependencies:** None (STORY-002 is a soft dependency for structural conformance, not a hard blocker)
**Parallelizable:** Yes
**Out of scope:** Bezos's logistics, supply chain, or business strategy opinions not applicable to product analytics artifact review.

---

### [EPIC-4] STORY-007: Implement dual-reviewer behavior for PRD phase

**As a** PM running the PRD command,
**I want** to receive separate, clearly labeled challenger reviews from both Marty Cagan and Steve Jobs — with a single shared Q&A session before both reviews are issued — and have each reviewer's flags namespaced so I can address them independently,
**so that** I get two complementary lenses on my PRD (discovery rigor from Cagan, simplicity and clarity from Jobs) without being asked the same questions twice or having their flags collide.

**Acceptance Criteria:**
1. When the PRD phase challenger runs, a single Q&A session is presented to the PM before any flags are issued. The questions draw from both Cagan's and Jobs's philosophy but are presented as a unified set, not attributed per-reviewer.
2. The PM answers the Q&A once. Both Cagan and Jobs use the PM's answers as context for their respective reviews. There is no second Q&A round.
3. If the PM types "skip" at the PRD Q&A, both personas deliver their skip responses (Cagan's then Jobs's, or Jobs's then Cagan's — order is consistent and defined in SKILL.md), then both proceed to their respective reviews using available context only.
4. The PRD challenger output contains two clearly labeled sections, each with its own reviewer header in the format: `CHALLENGER REVIEW — PRD / Reviewer: Marty Cagan` and `CHALLENGER REVIEW — PRD / Reviewer: Steve Jobs`.
5. Cagan's flags use the namespace prefix `cagan_prd_` (e.g., `cagan_prd_001`). Jobs's flags use the namespace prefix `jobs_prd_` (e.g., `jobs_prd_001`).
6. The PM can override a Cagan flag independently of Jobs flags, and vice versa. Override acknowledgment is issued in the voice of the reviewer whose flag was overridden.
7. Jobs's review section contains only flags in his three permitted categories (complexity masquerading as sophistication, bundled scope, fuzzy language). Jobs raises no flags in categories outside his scope, even when genuine issues exist in the PRD.
8. Cagan's review section contains flags in any of the four categories, with the exception that Cagan does not flag simplicity or precision issues already covered by Jobs's scope.
9. If Cagan finds no genuine issues, his section outputs a clean pass in his voice. If Jobs finds no genuine issues, his section outputs a clean pass in his voice. Both sections are always present in the output; neither section is omitted when the reviewer has no flags.
10. A PM reading the PRD challenger output can identify which reviewer raised each flag without reading the flag ID — the section label makes attribution unambiguous.

**Files affected:**
- `skills/pm-challenger/SKILL.md` (modified — dual-reviewer rules; this story extends STORY-001)
- `skills/pm-challenger/references/persona-cagan.md` (read at runtime; must exist per STORY-004)
- `skills/pm-challenger/references/persona-jobs.md` (read at runtime; must exist per STORY-005)

**Dependencies:** STORY-001 (SKILL.md orchestration rules must exist); STORY-004 (Cagan persona file); STORY-005 (Jobs persona file). This story can be authored in parallel but cannot be fully tested until those three files exist.
**Parallelizable:** Yes (authoring); testing requires STORY-001, STORY-004, STORY-005
**Out of scope:** Dual-reviewer behavior for any artifact phase other than PRD; changes to Cagan's behavior at the stories phase.

---

### [EPIC-1] STORY-008: Q&A protocol behavior — pre-review questions and response incorporation

**As a** PM beginning a challenger review for any artifact phase,
**I want** the assigned persona to ask me 2–3 targeted questions before issuing any flags, and to incorporate my answers into the review,
**so that** the review reflects my actual context and assumptions rather than generic concerns about my artifact type.

**Acceptance Criteria:**
1. For every artifact phase, the challenger presents 2–3 questions before displaying any flags. No flags are issued until after the PM responds to the Q&A (or types "skip").
2. The questions are generated from the PM's input and the loaded persona's philosophy. The questions are specific to what the PM has written — two PRDs on unrelated topics produce different questions from the same persona.
3. Questions are written in the assigned persona's voice. A Graham question sounds like Graham; a Bezos question sounds like Bezos. The questions are not generic challenger prompts.
4. The minimum question count is 2 and the maximum is 3. The system does not ask 1 question or more than 3.
5. After the PM responds, the formal review (flags or clean pass) incorporates the PM's answers. A flag that would have been raised but was resolved by the PM's answer is not raised. A flag that is strengthened by the PM's answer reflects that context.
6. The formal review is not issued before the PM has responded to the Q&A. The system waits for input.
7. The review header identifies the persona by name: `CHALLENGER REVIEW — [ARTIFACT TYPE] / Reviewer: [Persona Name]`.

**Files affected:**
- `skills/pm-challenger/SKILL.md` (relies on rules added in STORY-001)
- All persona files (read at runtime)

**Dependencies:** STORY-001 (Q&A protocol rules in SKILL.md); at least one persona file from STORY-003 through STORY-006 to test per-persona behavior.
**Parallelizable:** Yes (authoring); cross-persona testing requires STORY-001 and relevant persona files
**Out of scope:** Multi-turn Q&A (the Q&A is a single exchange — one set of questions, one PM response); Q&A history persistence across sessions.

---

### [EPIC-1] STORY-009: Skip flow behavior — in-character skip response across all personas

**As a** PM who types "skip" at a challenger Q&A prompt,
**I want** the assigned persona to respond in character — explaining what they were trying to uncover and why it mattered — before proceeding to the formal review,
**so that** skipping the Q&A has a cost of engagement (I read a real response from a named reviewer) without being blocked or punished.

**Acceptance Criteria:**
1. When the PM types "skip" at the Q&A prompt for any artifact phase, the system does not proceed silently. It first outputs the persona's skip response.
2. The skip response is sourced from the skip response template in the loaded persona file. The response is one paragraph, in the persona's voice, written as direct address to the PM.
3. The skip response is 3–5 sentences. It explains what the questions were trying to surface and why that information mattered — specific to that persona's philosophy, not generic.
4. The skip response is written in the first person. It reads as though the persona is speaking directly to the PM who just declined to engage.
5. The tone of the skip response is genuine, not punitive. The persona does not lecture, guilt, or withhold. After the skip response, the persona confirms the skip and proceeds to the formal review.
6. The formal review issued after a skip uses available context only (the PM's original artifact input). The review does not invent information that would have come from Q&A answers.
7. For the PRD dual-reviewer phase, both Cagan and Jobs deliver skip responses when the PM skips. The order is: Cagan skip response first, then Jobs skip response, then both formal reviews in their labeled sections.
8. The skip response differs across personas. Graham's skip response sounds like Graham; Cagan's sounds like Cagan; Jobs's sounds like Jobs; Bezos's sounds like Bezos. A single skip response cannot plausibly be attributed to the wrong persona.
9. After the skip response and confirmation, the formal review header format is unchanged: `CHALLENGER REVIEW — [ARTIFACT TYPE] / Reviewer: [Persona Name]`.

**Files affected:**
- `skills/pm-challenger/SKILL.md` (relies on skip flow rules added in STORY-001)
- All persona files (skip response templates read at runtime)

**Dependencies:** STORY-001 (skip flow rules in SKILL.md); at least one persona file from STORY-003 through STORY-006 to test per-persona behavior.
**Parallelizable:** Yes (authoring); cross-persona testing requires STORY-001 and relevant persona files
**Out of scope:** Persistent skip tracking across sessions; analytics or logging of skip events; a "skip challenger" full bypass (that behavior is defined in the existing SKILL.md and is unchanged by this feature).

---

## Implementation Order

1. **STORY-002** — Define persona file template first. Establishes the structural contract that STORY-003 through STORY-006 authors can use as a scaffold. Not a hard dependency, but eliminates rework.
2. **STORY-001** — Update SKILL.md with orchestration rules. Establishes phase-to-persona mapping, Q&A protocol, skip flow, and override voice rules. All other stories depend on these rules being documented.
3. **STORY-003, STORY-004, STORY-005, STORY-006** — Author the four persona files in any order or in parallel. Each is independent. Jobs (STORY-005) carries the highest complexity due to the hard scope constraint.
4. **STORY-007** — Dual-reviewer PRD behavior. Extends SKILL.md rules and requires Cagan (STORY-004) and Jobs (STORY-005) persona files to exist for full testing.
5. **STORY-008, STORY-009** — Q&A protocol and skip flow behavior validation. These stories can be authored in parallel with persona files but require STORY-001 and at least one persona file to test. They are integration-level acceptance stories that verify cross-persona consistency.
