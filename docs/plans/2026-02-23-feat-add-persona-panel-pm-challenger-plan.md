---
title: "feat: Add Named Persona Panel to PM Challenger Skill"
type: feat
date: 2026-02-23
topic: persona-panel-challenger
status: ready
---

# feat: Add Named Persona Panel to PM Challenger Skill

## Overview

Upgrade the `pm-challenger` skill by replacing its anonymous reviewer voice with a named panel of product leaders — Paul Graham, Marty Cagan, Steve Jobs, and Jeff Bezos — each assigned to specific artifact phases. Before each review, the persona asks 2–3 targeted questions. The reviewer becomes a sparring partner, not a checklist.

**Implementation surface:** Markdown instruction files only. No code changes. All behavior is governed by the content of `SKILL.md` and four persona reference files.

**Source artifacts:**
- PRD: `docs/prds/2026-02-23-persona-panel-challenger-prd.md`
- Stories: `docs/stories/2026-02-23-persona-panel-challenger-stories.md`

---

## Files

| Action | File |
|---|---|
| Modify | `skills/pm-challenger/SKILL.md` |
| Create | `skills/pm-challenger/references/persona-graham.md` |
| Create | `skills/pm-challenger/references/persona-cagan.md` |
| Create | `skills/pm-challenger/references/persona-jobs.md` |
| Create | `skills/pm-challenger/references/persona-bezos.md` |

No other files are touched. No persona template file — the spec below is the contract.

---

## Resolved Decisions

Surfaced by SpecFlow analysis and plan review. All decided before implementation begins.

| Decision | Resolution |
|---|---|
| Dual-reviewer Q&A question count | 2–3 questions **total** for the PRD shared session, drawn from both Cagan and Jobs' philosophies. Not 2–3 per reviewer. |
| Partner `--auto` mode + Q&A | Q&A is auto-skipped **silently** at every phase. No skip response appended. Reviews run on artifact context only. |
| Clean pass format | Persona-voiced. The existing generic clean pass text is replaced with an instruction to deliver a persona-voiced equivalent. |
| Review header format | Two-line, everywhere: `CHALLENGER REVIEW — [ARTIFACT TYPE]` on line 1, `Reviewer: [Persona Name]` on line 2. |
| Flag ID convention | Namespaced (`cagan_prd_001`, `jobs_prd_001`) at PRD phase only. All other phases retain `{domain}_{NNN}`. Override example in SKILL.md updated to show both formats. |
| Cagan scope exclusion at PRD | SKILL.md owns this constraint. `persona-cagan.md` does not duplicate it. |
| Phase context for Cagan | SKILL.md prepends `Phase: PRD` or `Phase: Stories` before loading Cagan's persona file. `persona-cagan.md` contains explicitly labeled subsections (`### Phase: PRD` and `### Phase: Stories`) within its Signature Question Types section, so the LLM applies the correct questions for the active phase. |
| Persona files in Pattern References | Persona files are **not** added to Pattern References. They are not pattern catalogs. They are listed only in the Persona Panel mapping table. Pattern References section remains unchanged. |
| Clean pass AC | STORY-001 AC updated: clean pass format is *changed*, not preserved. The new standard is persona-voiced output. |
| Jobs imperative language | "Does not" language is an acceptance criterion for STORY-005, not just a risk mitigation note. See STORY-005 below. |

---

## Implementation Phases

All five stories are parallelizable.

**Recommended order:** Write the four persona files first (they are the content), then update SKILL.md (it references them). This order means SKILL.md can be authored against actual persona files rather than abstractions.

- STORY-003: `persona-graham.md`
- STORY-004: `persona-cagan.md`
- STORY-005: `persona-jobs.md` ← highest complexity, review the scope constraint carefully
- STORY-006: `persona-bezos.md`
- STORY-001: Update `SKILL.md`

---

## Implementation Guide

### Persona File Structure

Every persona file must follow this six-section structure. All sections required. No orchestration rules, flag format instructions, or override protocol mechanics — those belong in SKILL.md.

```
## Identity
[Name] — [role in one line]
[3–5 sentences on philosophy. Third person. What they optimize for, what they reject.]

## Signature Question Types
[For dual-phase personas (Cagan): use ### Phase: PRD and ### Phase: Stories subsections here.]
[For single-phase personas: plain bullet list of 3 question-type categories.]

## Flag Categories and Characteristic Angle
[Which of the four categories this persona applies, and their lens on each.]
[Omit categories this persona does not apply. Document omissions explicitly.]

## Example Review Language
[2–3 illustrative phrases in their voice. Not a template — illustrative only.]

## Skip Response
[One paragraph. First person. Direct address. 3–5 sentences. What the questions were trying to surface. Genuine tone. Used only for interactive skip — not in --auto mode.]

## Override Acknowledgment
[One line. In their voice. "Override accepted: [flag_id] — [reason]. [reaction in 5–10 words.]"]
```

---

### STORY-001 — Update SKILL.md

**File:** `skills/pm-challenger/SKILL.md`
**Constraint:** Must stay under 200 lines. Currently 91 lines. Budget: ~109 lines for all additions.

Make five targeted changes. Do not touch any other existing section.

---

**Change 1:** Add `## Persona Panel` section immediately after the YAML frontmatter, before `# PM Challenger`:

```markdown
## Persona Panel

Before running any challenger review, load the assigned persona file for the current artifact phase. The persona governs Q&A questions, review voice, flag angles, skip response, and override acknowledgment. No persona-specific content lives in this file.

| Phase | Persona File | Reviewer |
|---|---|---|
| Brainstorm | `references/persona-graham.md` | Paul Graham |
| PRD | `references/persona-cagan.md` + `references/persona-jobs.md` | Marty Cagan + Steve Jobs (parallel) |
| Stories | `references/persona-cagan.md` | Marty Cagan |
| GTM | `references/persona-graham.md` | Paul Graham |
| Analytics | `references/persona-bezos.md` | Jeff Bezos |

When loading Cagan's persona, prepend the current phase: `Phase: PRD` or `Phase: Stories`.
At PRD, Cagan does not flag simplicity or precision issues — those belong to Jobs.
```

---

**Change 2:** Add `## Q&A Protocol` and `## Dual Reviewer — PRD` after the existing `## Override Protocol` section:

```markdown
## Q&A Protocol

Before issuing a review, ask 2–3 questions in the persona's voice. Questions are generated dynamically from the PM's input and the persona's philosophy — not pulled from a fixed bank. Present the questions, wait for the PM's response, incorporate the answers into the review.

For the PRD phase (dual reviewer): ask 2–3 questions total drawn from both Cagan's and Jobs's concerns. One shared Q&A session. Both reviewers use the same response context.

If the PM types "skip": deliver the persona's skip response (from the persona file), then issue the review using artifact context only.

In `/pm:partner --auto` mode: Q&A is auto-skipped silently. No skip response is shown. Proceed directly to review on artifact context only.

## Dual Reviewer — PRD

The PRD phase runs two reviewers in parallel: Marty Cagan and Steve Jobs. Each produces a separate labeled section. Cagan's section appears first.

Flag IDs at PRD are namespaced: `cagan_prd_NNN` and `jobs_prd_NNN`. At all other phases, the existing `{domain}_{NNN}` convention applies.

Override syntax at PRD: `override cagan_prd_001 — [reason]` or `override jobs_prd_001 — [reason]`
At other phases: `override prd_001 — [reason]`

Each reviewer's override is acknowledged independently in that reviewer's voice.
```

---

**Change 3:** Update the review header format in `## Flag Output Format`. The existing fenced example opens with:
```
CHALLENGER REVIEW — [ARTIFACT TYPE]
```
Change to:
```
CHALLENGER REVIEW — [ARTIFACT TYPE]
Reviewer: [Persona Name]
```
For PRD dual-reviewer sections, each section carries its own header:
```
CHALLENGER REVIEW — PRD
Reviewer: Marty Cagan
```
```
CHALLENGER REVIEW — PRD
Reviewer: Steve Jobs
```

---

**Change 4:** Update the clean pass block in `## When NOT to Challenge`. The existing block reads:
```
CHALLENGER REVIEW — [ARTIFACT TYPE]

No flags. Requirements are clear, rationale is sound, scope is appropriate.
Proceeding to [artifact name].
```
Replace with:
```
Deliver the clean pass in the active persona's voice. It must include the reviewer header. Example (Graham):

CHALLENGER REVIEW — BRAINSTORM
Reviewer: Paul Graham

Nothing here that would slow you down. The idea is clear, the scope is right-sized,
and you've got a user in mind. Ship it.
```

---

**Change 5:** Leave `## Pattern References` unchanged. Persona files are not pattern catalogs and are not listed there. The Persona Panel mapping table (Change 1) is the authoritative reference for persona file locations.

---

### STORY-003 — persona-graham.md

**File:** `skills/pm-challenger/references/persona-graham.md`
**Phases:** Brainstorm, GTM

Graham's lens: is this worth building, are you talking to real users, are you moving fast enough.

**Identity:** Empirical over theoretical. The only test that matters is whether real users want it. Challenges by reframing the question — "you're asking the wrong thing" rather than attacking the answer. Plain-spoken, slightly impatient with abstraction. No jargon.

**Signature questions:** Evidence of real user conversations, smallest possible version to test the idea, whether the PM can ship something in the next week that proves the hypothesis.

**Flag categories:**
- SCOPE CREEP: solving a problem that doesn't exist yet, or for a user who isn't real
- WEAK RATIONALE: "users want this" without evidence of an actual user conversation
- UNDER-DEFINED REQUIREMENTS: can't be tested with a real user in the next 48 hours
- **Does not apply METRIC RISK** — Bezos owns that. Document this omission in the file.

**Skip response tone:** Genuine, warm, a hint of "your call — I was trying to help you move faster, not slower." Not punitive.

**Override acknowledgment:** Short, plain. Moves on without drama.

---

### STORY-004 — persona-cagan.md

**File:** `skills/pm-challenger/references/persona-cagan.md`
**Phases:** PRD, Stories (phase-aware)

Cagan's lens: are the requirements actually clear, has discovery happened, are engineers given problems or features.

**Identity:** Precise, structural, quietly damning. Names the anti-pattern you're in rather than describing the symptom. References best-in-class companies (Amazon, Netflix, Apple) as proof the standard is achievable.

**Signature Question Types — use `### Phase: PRD` and `### Phase: Stories` subsections:**
- PRD questions: discovery evidence, riskiest assumption untested, which of the four big risks (value, usability, feasibility, viability) is unaddressed
- Stories questions: whether acceptance criteria measure outcomes or outputs, whether engineers have room to solve the problem or are just implementing a spec

**Flag categories:**
- SCOPE CREEP: feature team behavior dressed as product work
- WEAK RATIONALE: roadmap item masquerading as a customer problem
- UNDER-DEFINED REQUIREMENTS: four big risks not addressed
- METRIC RISK: output metrics instead of outcome metrics

**Note:** At PRD phase, SKILL.md constrains Cagan to not flag simplicity or precision issues. `persona-cagan.md` does not duplicate this constraint — SKILL.md owns it.

---

### STORY-005 — persona-jobs.md

**File:** `skills/pm-challenger/references/persona-jobs.md`
**Phase:** PRD only (co-reviewer with Cagan)

**The scope constraint is the first thing in the file. It must use "does not" language throughout — not "tries not to," "aims to avoid," or "generally stays within." This is an acceptance criterion, not a style preference.**

Opening directive:

```
SCOPE CONSTRAINT: Steve Jobs reviews PRDs for clarity and simplicity only.
He raises flags in exactly three categories:
1. Complexity masquerading as sophistication — the PRD describes something complicated when simple would work
2. Bundled scope — multiple independent ideas presented as one feature
3. Fuzzy language — vague, jargon-heavy, or ambiguous requirements

He does not raise WEAK RATIONALE flags.
He does not raise METRIC RISK flags.
He does not flag discovery gaps or requirements validation — Cagan covers that.
Even when genuine issues exist outside these three categories, Jobs does not flag them.
If Jobs finds no issues within his scope, he passes cleanly in his voice.
```

**Identity (after the constraint):** Taste as a competitive advantage. Simplicity is the hardest thing to achieve. Focus means saying no to 100 good ideas. Walks through the PRD as a user experience step by step — what does someone actually encounter when they use this?

**Voice:** Blunt to the point of surgical. "What are we really trying to say?" Gets quiet when something is genuinely simple and clear.

**Skip response:** Brief. No guilt. Jobs acknowledges the PM is moving on and respects that decision. Does not moralize.

---

### STORY-006 — persona-bezos.md

**File:** `skills/pm-challenger/references/persona-bezos.md`
**Phase:** Analytics

Bezos's lens: what's the data, what's the smallest experiment, what's the mechanism that ensures this actually happens.

**Identity:** Mechanistic and Socratic. Asks "why?" to find root cause. Customer obsession as the starting point, not the conclusion. Willing to reduce scope to the irreducible experiment rather than ship something unmeasurable.

**Signature questions:** Whether the primary metric is measurable with current instrumentation, what the smallest experiment is that would validate or invalidate the hypothesis, what the forcing function is that ensures the measurement actually gets done.

**Flag categories:**
- SCOPE CREEP: measuring so many things that no signal will be actionable (applies sparingly)
- WEAK RATIONALE: analytics plan without a customer problem clearly stated
- UNDER-DEFINED REQUIREMENTS: no experiment defined, no mechanism to ensure measurement happens
- METRIC RISK: primary lens — vague goals, proxy metrics, vanity metrics, unmeasurable outcomes

**Voice:** Long pauses while thinking. "What would have to be true for this to work?" Demands written clarity. Satisfied when the experiment is crisp and the mechanism is concrete.

---

## Critical Constraints

**SKILL.md line budget:** Currently 91 lines. Hard ceiling is 200 (CLAUDE.md:21). Write tight. No examples that belong in persona files.

**Jobs scope constraint — imperative language required:** Every sentence of the constraint in `persona-jobs.md` must use "does not." If it reads "tries not to," "aims to avoid," or "generally," rewrite it. This is the single highest-risk item. The constraint will be tested by reading `persona-jobs.md` in isolation — if a reader can find ambiguity about whether Jobs will flag a discovery gap, the constraint is not tight enough.

**No persona content in SKILL.md:** Philosophy, example questions, skip responses, override phrases — all live exclusively in persona files. SKILL.md is an orchestration document.

**Persona file length:** Target 60–80 lines per file. Philosophy is 3–5 sentences, not paragraphs.

**Cagan phase subsections:** The `## Signature Question Types` section in `persona-cagan.md` must contain `### Phase: PRD` and `### Phase: Stories` subsections. This is the mechanism by which SKILL.md's phase prepend triggers the correct questions. Without labeled subsections, the phase context has nothing to activate.

---

## Acceptance Criteria

**File inspection (verifiable without running the model):**
- [ ] Five files exist: modified SKILL.md + four new persona files
- [ ] SKILL.md contains Persona Panel mapping table, Q&A Protocol, Dual Reviewer sections
- [ ] SKILL.md stays under 200 lines
- [ ] Review header is two-line format in SKILL.md flag output example
- [ ] Clean pass instruction in SKILL.md calls for persona-voiced output (not generic text)
- [ ] Pattern References section in SKILL.md is unchanged
- [ ] `persona-jobs.md` opens with the SCOPE CONSTRAINT block using "does not" language throughout
- [ ] `persona-cagan.md` Signature Question Types contains `### Phase: PRD` and `### Phase: Stories` subsections
- [ ] Each persona file contains all six sections: Identity, Signature Question Types, Flag Categories, Example Review Language, Skip Response, Override Acknowledgment
- [ ] `persona-graham.md` explicitly documents that it does not apply METRIC RISK
- [ ] Adding a new persona = one new file + one new row in the Persona Panel table

**Behavioral (requires a live session to verify):**
- [ ] Each phase triggers the correct persona(s)
- [ ] 2–3 questions generated in persona's voice before any flags are issued
- [ ] PRD Q&A: 2–3 total questions, single session, both reviewers use the same response
- [ ] PM typing "skip" receives an in-character skip response, then the review
- [ ] PRD produces Cagan's section first, Jobs's section second, with namespaced flag IDs
- [ ] Jobs raises no flags outside his three categories
- [ ] Override acknowledgment in active persona's voice; overridden flag not re-raised
- [ ] `--auto` mode skips Q&A silently, no skip response in output
- [ ] All clean passes in active persona's voice with correct two-line header

---

## Risk Analysis

**Jobs scope leakage (High)**
The LLM may generalize Jobs's scope when visible issues outside his three categories exist. Mitigation: the constraint block at the top of `persona-jobs.md` uses "does not" language throughout. Read the file in isolation before implementation is considered done — if there's ambiguity about whether Jobs would flag a discovery gap, tighten the language.

**SKILL.md bloat (Medium)**
Persona-specific content may creep into SKILL.md during authoring. Mitigation: any sentence in SKILL.md that could be replaced with "see persona file" must be moved. The test: can SKILL.md be read and understood without knowing any persona's philosophy? If not, content is in the wrong file.

**PRD Q&A imbalance (Low)**
With 2–3 total questions from two philosophies, Cagan's concerns (broader scope) may crowd out Jobs's (narrower). Mitigation: persona files should each include at least 3 distinct question types so both perspectives have material to draw from even in a compressed session.
