---
title: "feat: Build Product Management Skill Plugin"
type: feat
date: 2026-02-22
deepened: 2026-02-22
reviewed: 2026-02-22
---

# feat: Build Product Management Skill Plugin

## Review Summary

**Reviewed on:** 2026-02-22
**Reviewers:** DHH, Kieran, Simplicity Reviewer

### Changes Applied from Review

**Cut (all 3 reviewers in agreement):**
- Teammate swarm pattern for GTM + analytics → replaced with sequential synchronous subagents
- Structured YAML challenger output contract → replaced with structured prose + clear terminal signal
- YAML block embedded in project CLAUDE.md → replaced with separate `product-context.yaml` file
- `plugin.json` capabilities array → minimal manifest only (name, version, description, author)
- `skills/pm-challenger/templates/challenge-output.md` → output format described inline in SKILL.md

**Added from Kieran's gap analysis:**
- Challenger output format completed: `resolution: accepted | override`, flag ID convention, `clear` state schema, max flag policy
- Brainstorm document format defined (was the only command with no output template)
- `$ARGUMENTS` disambiguation rule (file path vs. feature name)
- Topic slug derivation rules
- `schema_version: 1` in handoff YAML
- Concurrent write prohibition added to `gtm-strategist.md` and `analytics-planner.md` specs
- `/pm:partner` partial failure handling specified
- AARRR vs HEART choice consequence defined

**Kept:**
- `skills/pm-challenger/references/` subdirectory (4 pattern catalogs — core IP)
- `docs/handoffs/` directory (simplified — prose summary is primary, YAML for `/workflows:plan` detection)
- Sequential orchestration throughout `/pm:partner`

---

## Overview

A Claude Code skill plugin that acts as a force multiplier for AI-native product and engineering teams. Six modular PM commands, each with an inline challenger layer, plus a PM Partner orchestrator for full end-to-end lifecycle runs. The plugin produces structured PM artifacts — brainstorm docs, PRDs, user stories, GTM plans, analytics plans — while actively pushing back on feature bloat, under-defined requirements, and bad ideas.

This is not PM automation. It is a senior PM co-pilot that produces outputs designed to feed directly into compound-engineering workflows.

Brainstorm reference: `docs/brainstorms/2026-02-22-product-management-skill-brainstorm.md`

---

## Problem Statement

Product managers working with AI-native engineering teams have no purpose-built tooling that both *produces* PM artifacts and *challenges* their thinking. Existing approaches either automate PM tasks (removing PM judgment) or are general-purpose assistants (no PM-specific process). Neither acts as a genuine force multiplier. This plugin fills that gap.

---

## Proposed Solution

A modular Claude Code plugin with:

- **1 core skill** encoding the challenger methodology and PM process knowledge (with `references/` subdirectory for pattern catalogs)
- **6 user-facing commands** (`/pm:brainstorm`, `/pm:prd`, `/pm:stories`, `/pm:gtm`, `/pm:analytics`, `/pm:partner`)
- **4 specialist agents** that generate the actual deliverables (prd-writer, story-writer, gtm-strategist, analytics-planner)
- **Context persistence** — new product context is written to `product-context.yaml` at the project root after each session
- **Handoff summary** — `/pm:partner` writes a minimal YAML file to `docs/handoffs/` for `/workflows:plan` auto-detection

---

## Plugin Architecture

```
product-skill/
├── .claude-plugin/
│   └── plugin.json                   # Minimal: name, version, description, author only
├── skills/
│   └── pm-challenger/
│       ├── SKILL.md                  # Challenger methodology (<200 lines, no XML tags)
│       └── references/
│           ├── scope-creep-patterns.md
│           ├── requirements-quality.md
│           ├── metrics-anti-patterns.md
│           └── gtm-gap-patterns.md
├── commands/
│   └── pm/
│       ├── brainstorm.md             # /pm:brainstorm
│       ├── prd.md                    # /pm:prd
│       ├── stories.md                # /pm:stories
│       ├── gtm.md                    # /pm:gtm
│       ├── analytics.md              # /pm:analytics
│       └── partner.md                # /pm:partner (orchestrator)
├── agents/
│   ├── prd-writer.md
│   ├── story-writer.md
│   ├── gtm-strategist.md
│   └── analytics-planner.md
├── docs/
│   ├── brainstorms/
│   ├── plans/
│   ├── prds/
│   ├── stories/
│   ├── gtm/
│   ├── analytics/
│   └── handoffs/
└── CLAUDE.md                         # Plugin documentation only
```

**Context persistence** lives in `product-context.yaml` at the **project root** (not inside this plugin). The plugin's `CLAUDE.md` contains only command documentation and a pointer to this file. If `product-context.yaml` does not exist, the first command run will prompt the PM to create it.

---

## Technical Approach

### Challenger Layer (Universal, Inline)

Every command applies the challenger as an **inline step** before invoking any specialist agent. The challenger behavior is defined in `skills/pm-challenger/SKILL.md` and loaded as process knowledge — commands do not invoke a separate challenger agent via `Task()`.

**Challenger output format** (prose, not YAML — the gate is conversational):

```
CHALLENGER REVIEW — PRD

[flag_prd_001] SCOPE CREEP
Issue: Payment integration overlaps with the existing billing feature launched Q4 2025.
Consequence: Engineering will build a parallel payment path that conflicts with billing's model.
Resolution path: Remove payment integration from this PRD and reference the billing feature, or create a separate integration PRD.

[flag_prd_002] WEAK RATIONALE
Issue: "Users want faster checkout" — no user evidence cited.
Consequence: We may build the wrong solution to a problem we don't fully understand.
Resolution path: Add a user research citation or validation data before proceeding.

Please address each flag. Type "override [flag_id] — [reason]" to override, or describe how you'll address it. Once all flags are resolved, I'll proceed.
```

**When status is clear:**
```
CHALLENGER REVIEW — PRD
No significant concerns. Ready to proceed.
```

**Flag ID convention:** `{domain}_{NNN}` where domain is `brainstorm`, `prd`, `stories`, `gtm`, or `analytics`, and NNN is zero-padded sequential within the run. Example: `prd_001`, `prd_002`.

**Maximum flags:** Surface the top 5 most significant flags. If more exist, note: "N additional minor concerns noted — ask to see them." Surfacing 12 flags in one pass is a brutal UX that trains PMs to skip the challenger.

**Four challenger categories (always check all four):**

| Category | What to Look For |
|---|---|
| Scope creep | Feature does more than one thing; "it would also be nice if"; features that could be a standalone roadmap item |
| Feature bloat | Complexity doesn't match frequency/severity of the problem |
| Weak rationale | "Our competitor has it"; "we think users want this"; no user evidence cited |
| Under-defined | Requirements using: "fast," "simple," "intuitive," "robust," "scalable," "better" without measurable definition |

**Mandatory challenger triggers (always flag):**
- Any requirement using unmeasurable language
- Any success metric with no baseline defined
- Any story missing the "so that [benefit]" clause
- GTM plan targeting "all users"
- Analytics plan with only vanity metrics
- PRD with an empty out-of-scope section
- Feature whose stated rationale is "our competitor has it"
- Feature that appears to solve more than one distinct user problem

**Gate:** Do not invoke any specialist agent until the PM has explicitly responded to each flag — either acknowledging they will address it or overriding it with `"override [flag_id] — [reason]"`. Silence does not count as resolution.

**When NOT to challenge:**
- PM has made an explicit decision with stated rationale
- Issue is cosmetic or subjective
- PM has domain context explaining the decision (committed customer deal, regulatory requirement)
- The "issue" is an established pattern in `product-context.yaml`

### Context Persistence (`product-context.yaml`)

All product context accumulates in `product-context.yaml` at the project root. This is a standalone YAML file, not embedded in markdown.

```yaml
# product-context.yaml
# Managed by pm commands — append only, do not delete entries manually
# Each entry is date-stamped with its source command

schema_version: 1
company: ""
personas: []
tech_stack: []
principles: []
recent_artifacts:
  # - date: 2026-02-22
  #   type: brainstorm
  #   topic: registration-flow
  #   path: docs/brainstorms/2026-02-22-registration-flow-brainstorm.md
```

**Rules:**
- Append only — never overwrite existing entries
- Date-stamp every addition with a comment: `# Added 2026-02-22 by /pm:prd`
- Tell the PM what was added in the output summary (no silent writes)
- Cap `recent_artifacts` at 20 entries; oldest entries are removed when the cap is reached
- In `/pm:partner`, only the orchestrator writes to `product-context.yaml` — individual subagents return context learnings as output for the orchestrator to write

**Read before ask:** Every command reads `product-context.yaml` before asking any clarifying questions. Ask only for fields that are (a) not present and (b) required for that specific command. Degrade gracefully if the file doesn't exist or can't be parsed: "Could not read product-context.yaml — proceeding without stored context. Please check the file format."

### Topic Slug Derivation

The topic slug drives filename consistency across all artifacts in a session. Derivation rules:

1. If `$ARGUMENTS` starts with `docs/` or ends with `.md` → extract the topic from the filename using the pattern `YYYY-MM-DD-{topic}-{type}.md` → topic is the middle segment
2. If `$ARGUMENTS` is a plain feature name → slugify it: lowercase, spaces to hyphens, strip special characters
3. If a brainstorm or PRD artifact is discovered from `docs/` → use the topic extracted from that filename
4. The topic slug must remain consistent across all five artifacts in a `/pm:partner` run — the orchestrator sets it at the start and passes it to every subagent

Example: "Registration Flow" → `registration-flow` → produces `docs/prds/2026-02-22-registration-flow-prd.md`

### $ARGUMENTS Disambiguation

Every command must disambiguate the input:

- If `$ARGUMENTS` starts with `docs/` or ends with `.md` → treat as an artifact path; read that file
- If `$ARGUMENTS` is empty → run artifact discovery (scan the relevant `docs/` subdirectory), then ask if nothing found
- Otherwise → treat as a feature name and run artifact discovery using it as a filter

### Artifact Discovery (Every Command)

Every command begins with artifact discovery before asking any questions:

```bash
ls -la docs/<relevant-dir>/*.md 2>/dev/null | sort -r | head -10
```

- Announce what was found: "Found PRD from 2026-02-22: registration-flow. Using as input."
- If multiple artifacts found: list all and ask the PM to confirm which to use (most recent is default)
- If nothing found: ask the PM to provide input or describe the feature

### compound-engineering Handoff

At the end of a `/pm:partner` full-lifecycle run:

**Prose summary** (displayed to PM):
```
PM Partner complete. Artifacts produced:

  Brainstorm:  docs/brainstorms/2026-02-22-<topic>-brainstorm.md
  PRD:         docs/prds/2026-02-22-<topic>-prd.md
  Stories:     docs/stories/2026-02-22-<topic>-stories.md
  GTM Plan:    docs/gtm/2026-02-22-<topic>-gtm.md
  Analytics:   docs/analytics/2026-02-22-<topic>-analytics.md

Ready to hand off to engineering.
Run `/workflows:plan` with the PRD to start implementation planning.
```

**Handoff file** (`docs/handoffs/YYYY-MM-DD-<topic>-handoff.yaml`) — minimal, for `/workflows:plan` auto-detection:
```yaml
schema_version: 1
date: 2026-02-22
feature: <topic>
prd: docs/prds/2026-02-22-<topic>-prd.md
stories: docs/stories/2026-02-22-<topic>-stories.md
brainstorm: docs/brainstorms/2026-02-22-<topic>-brainstorm.md
gtm: docs/gtm/2026-02-22-<topic>-gtm.md
analytics: docs/analytics/2026-02-22-<topic>-analytics.md
```

### /pm:partner Orchestration Pattern

All phases use **synchronous subagents** (no Teammate pattern, no `run_in_background`). Sequential from start to finish: brainstorm → prd → stories → gtm → analytics → handoff.

Each phase blocks until complete and returns the artifact path, which is passed as input to the next phase.

**Partial failure handling:**
- If a sequential phase returns an empty or non-existent artifact path, halt immediately
- Report: "The [phase] phase did not produce an artifact. Run `/pm:[phase]` directly to retry, or check the error above."
- Do not invoke downstream phases with a bad input
- Do not attempt to write the handoff file on partial failure

**End of run:**
- Write `product-context.yaml` with all context learnings from all phases (orchestrator writes once, not per-phase)
- Write handoff YAML to `docs/handoffs/`
- Display prose summary

### Output Locations

| Command | Output Path |
|---|---|
| `/pm:brainstorm` | `docs/brainstorms/YYYY-MM-DD-<topic>-brainstorm.md` |
| `/pm:prd` | `docs/prds/YYYY-MM-DD-<topic>-prd.md` |
| `/pm:stories` | `docs/stories/YYYY-MM-DD-<topic>-stories.md` |
| `/pm:gtm` | `docs/gtm/YYYY-MM-DD-<topic>-gtm.md` |
| `/pm:analytics` | `docs/analytics/YYYY-MM-DD-<topic>-analytics.md` |
| `/pm:partner` | All of the above + `docs/handoffs/YYYY-MM-DD-<topic>-handoff.yaml` |

---

## File Specifications

### `.claude-plugin/plugin.json`

```json
{
  "name": "product-skill",
  "version": "1.0.0",
  "description": "PM force multiplier for AI-native product and engineering teams",
  "author": "LeagueApps"
}
```

### `skills/pm-challenger/SKILL.md`

**Purpose:** Process knowledge for how to challenge PM decisions well. Under 200 lines. No XML tags. Imperative/infinitive writing style.

**Key sections:**
- Concrete example: one input (scope-creepy feature idea) → exact flag output
- The four categories with signal descriptions
- Mandatory trigger list
- Flag ID convention (`{domain}_{NNN}`)
- Max 5 flags per review policy
- Output format: prose with labeled flag blocks + clear terminal line
- When NOT to challenge
- Override protocol (accept without argument; log it; proceed)

**References subdirectory (load selectively by domain):**
- `scope-creep-patterns.md` — signals, compound requirements, "also nice if" patterns
- `requirements-quality.md` — measurability test, under-defined requirement patterns
- `metrics-anti-patterns.md` — vanity metrics catalog and the causal link test
- `gtm-gap-patterns.md` — missing rollout triggers, no internal comms, no rollback conditions

### `agents/prd-writer.md`

**Purpose:** Writes an agentic PRD for both human review and AI engineering agent consumption. Uses `model: inherit`. Includes `<example>` block in description.

**Always-include PRD sections (10 sections, 1,500–3,000 words total, 1-3 paragraphs per section):**
1. Problem statement (must reference user evidence, not assumptions)
2. User personas and jobs-to-be-done (from `product-context.yaml` or session)
3. Proposed solution (narrative, 2-3 paragraphs)
4. Functional requirements (numbered, atomic, each independently testable; acceptance criteria written as predicates, not prose)
5. **Explicit out-of-scope list** (as important as in-scope; stated as positive assertions: "This PRD does not include X")
6. Constraints (what existing behavior, schema, or API must not change — for engineering agents)
7. Assumptions (when proved wrong, signals which decisions to revisit)
8. Open questions (a PRD with no open questions is overconfident)
9. Success criteria (measurable: "completion rate > 72% at 30 days", not "users will love it")
10. Dependencies

**For agentic consumption:** Include at least one concrete example of the happy path per major requirement.

### `agents/story-writer.md`

**Purpose:** Converts a PRD into user stories. Groups by epic. Includes Story ID and structured metadata. Uses `model: inherit`. Includes `<example>` block.

**Decomposition strategy:** One epic per major feature area from the PRD. Decompose by user-facing capability, not implementation task. Expect 1–2 stories per functional requirement (8 requirements → 8–16 stories).

**Output format per story:**
```
**Story ID:** US-{NNN}
**Epic:** {epic-name}
**Priority:** P0 | P1 | P2
**Estimate:** S (1-2d) | M (3-5d) | L (6-10d)

As a [persona], I want [capability] so that [benefit].

Acceptance Criteria:
- Given [precondition], When [action], Then [expected outcome]
- Given [edge case], When [action], Then [expected behavior]
- [At least one negative/failure case per story]

Notes: [Implementation context — guidance, not instructions]
```

**Quality rules:** Every story needs all three parts. 3–5 ACs per story (more = split). Every story traces to a PRD requirement. Stories within an epic are independently deliverable.

### `agents/gtm-strategist.md`

**Purpose:** Produces a GTM plan. Uses `model: inherit`. Includes `<example>` block.

**Hard constraint in agent file:** "Do NOT write to `product-context.yaml`. Return any context learnings in your completion output for the orchestrator to write."

**Always-include GTM sections (7 sections):**
1. Launch goal + success definition (one sentence; what does winning look like at 90 days?)
2. Target audience + ICP (specific segment — not "all users")
3. Positioning + differentiation
4. Messaging (headline + 2-3 supporting points + one objection-handling statement)
5. Rollout phases with **exit criteria** (internal → beta → GA; each needs: who's included, trigger to advance, rollback trigger)
6. Stakeholder comms plan (internal teams before customers; always flag missing sales enablement)
7. Launch risks (what could go wrong; contingency per risk)

### `agents/analytics-planner.md`

**Purpose:** Produces a narrative analytics plan. Measurement defined at spec time. Uses `model: inherit`. Includes `<example>` block.

**Hard constraint in agent file:** "Do NOT write to `product-context.yaml`. Return any context learnings in your completion output for the orchestrator to write."

**Framework:** Ask the PM to choose AARRR or HEART at the start. If AARRR, organize supporting metrics around Acquisition/Activation/Retention/Revenue/Referral. If HEART, organize around Happiness/Engagement/Adoption/Retention/Task Success. The north star metric and failure signals apply regardless of framework.

**Always-include analytics sections (9 sections):**
1. North star metric (one metric that proxies the feature's core value)
2. Counter metrics (prevent gaming the NSM)
3. Leading indicators (days 1–14 early signals), organized by chosen framework
4. Lagging indicators (30/60/90-day confirmation), organized by chosen framework
5. **Baseline** (current state — required for every metric; flag any metric without a baseline)
6. Success thresholds at 30/60/90 days (specific numbers)
7. **Failure signals** (what triggers a rollback conversation)
8. Data sources and measurement approach (narrative)
9. **Metric ownership** (who owns the analysis)

### Command File Conventions (All Commands)

**Frontmatter:**
```yaml
---
name: pm:<command-name>
description: <one sentence — what it does and when to invoke it>
argument-hint: "[feature-name or artifact-path]"
allowed-tools: Read, Write, Grep, Bash, Task, AskUserQuestion
---
```

**Required elements in every command body (in order):**
1. `**Note: The current year is 2026.**`
2. `<feature_description> #$ARGUMENTS </feature_description>`
3. `**Process knowledge:** Load the pm-challenger skill for challenge methodology.`
4. `**If the input above is empty:** [command-specific question]`
5. **$ARGUMENTS disambiguation** — file path vs. feature name
6. **Artifact discovery** — scan `docs/<relevant-dir>/`; list multiple matches; default to most recent
7. **Read `product-context.yaml`** — extract all present fields; ask only for required missing context
8. **Challenger phase** — inline; max 5 flags; explicit gate before proceeding to specialist agent
9. **Specialist agent invocation** (not for `/pm:brainstorm` — no specialist agent)
10. **Output Summary** — artifact path(s), topic slug used, what was appended to `product-context.yaml`
11. **`product-context.yaml` update** — append only, date-stamped, announce what was added

**No "offer to proceed" chaining.** Commands may note the logical next step in the output summary as informational text only. Never ask "Would you like me to proceed with..." and never invoke a follow-on command.

### `commands/pm/brainstorm.md` — Output Format

Unlike other commands, `/pm:brainstorm` has no specialist agent. The challenger and brainstorm dialogue are both inline. The brainstorm document it produces must follow this structure:

```markdown
---
date: YYYY-MM-DD
topic: <kebab-case-topic>
---

# <Topic Title>

## Problem Hypothesis
[The problem we believe exists — with evidence or acknowledged assumption]

## Target Persona
[Who experiences this problem most acutely]

## Approaches Considered
### Approach A: [Name]
[Description, pros, cons, when best suited]

### Approach B: [Name]
[Description, pros, cons, when best suited]

## Chosen Approach
[Which approach and why]

## Key Assumptions
[What must be true for this to succeed]

## Pre-Mortem
["Imagine it's 6 months from now and this failed — what happened?"]

## Open Questions
[Unresolved questions for the PRD phase]

## Next Steps
→ Run `/pm:prd` to generate a Product Requirements Document
```

### Plugin `CLAUDE.md` (Documentation Only)

```markdown
# CLAUDE.md — product-skill plugin

This plugin provides PM tooling for AI-native product teams.

## Commands
- `/pm:brainstorm` — Explore and validate a feature idea
- `/pm:prd` — Generate a Product Requirements Document
- `/pm:stories` — Generate user stories from a PRD
- `/pm:gtm` — Generate a go-to-market plan
- `/pm:analytics` — Generate a narrative analytics plan
- `/pm:partner` — Full lifecycle orchestrator (runs all of the above)

## Setup
Product context is stored in `product-context.yaml` at your project root.
The first pm command you run will prompt you to create it if it doesn't exist.
```

---

## Implementation Phases

### Phase 1: Foundation

**Deliverables:**
- `.claude-plugin/plugin.json`
- `skills/pm-challenger/SKILL.md`
- `skills/pm-challenger/references/` (4 pattern catalog files)
- `CLAUDE.md` (documentation only)
- `product-context.yaml` template (at project root)
- All output directories: `docs/prds/`, `docs/stories/`, `docs/gtm/`, `docs/analytics/`, `docs/handoffs/`

**Success criteria:** Challenger skill loads and produces labeled flag output with correct IDs on a test feature idea. Outputs "No significant concerns. Ready to proceed." when input is clean.

### Phase 2: All Commands and Specialist Agents

**Deliverables (no ordering dependency between the first 5):**
- `commands/pm/brainstorm.md` (inline challenger + dialogue; no specialist agent)
- `agents/prd-writer.md` + `commands/pm/prd.md`
- `agents/story-writer.md` + `commands/pm/stories.md`
- `agents/gtm-strategist.md` + `commands/pm/gtm.md`
- `agents/analytics-planner.md` + `commands/pm/analytics.md`
- `commands/pm/partner.md` (built last — depends on all other commands)

**Success criteria:**
- Full lifecycle: `/pm:brainstorm` → `/pm:prd` → `/pm:stories` produces eng-ready artifacts
- `/pm:partner` runs all five phases sequentially and writes handoff YAML
- `product-context.yaml` updated with dated entries after each command
- Challenger flags are max 5; gate works; override with reason is accepted and not re-raised
- Partial failure halts the run with a specific error message

---

## Acceptance Criteria

### Functional Requirements

- [ ] `/pm:brainstorm` applies challenger inline, produces brainstorm doc in correct format (7 sections)
- [ ] `/pm:prd` produces 10-section agentic PRD (1,500–3,000 words) with explicit out-of-scope list
- [ ] `/pm:stories` produces stories with Story IDs, epics, priorities, estimates, Given/When/Then ACs
- [ ] `/pm:gtm` produces 7-section GTM plan with rollout exit criteria and rollback triggers
- [ ] `/pm:analytics` produces 9-section analytics plan with baselines, failure signals, metric ownership
- [ ] `/pm:partner` runs all five phases sequentially; writes handoff YAML; halts on phase failure
- [ ] All commands disambiguate `$ARGUMENTS` (file path vs. feature name)
- [ ] All commands derive topic slug consistently and use it across all artifact filenames
- [ ] All commands scan `docs/` before asking questions; handle multiple matches
- [ ] All commands read `product-context.yaml` before asking clarifying questions
- [ ] All commands append date-stamped context to `product-context.yaml` and announce what was added
- [ ] Challenger produces labeled flag output (max 5); clear state outputs "No significant concerns"
- [ ] Challenger gate is explicit — silence does not count as resolution
- [ ] PM override with `"override [flag_id] — [reason]"` is accepted and logged, never re-raised
- [ ] `/pm:partner` halts and reports the failed phase if a subagent returns no artifact
- [ ] Only the orchestrator writes to `product-context.yaml` during partner runs

### Quality Gates

- [ ] `plugin.json` contains only name, version, description, author
- [ ] `SKILL.md` is under 200 lines with no XML tags
- [ ] Agent files include `model: inherit` and `<example>` block in description
- [ ] `gtm-strategist.md` and `analytics-planner.md` explicitly prohibit writing to `product-context.yaml`
- [ ] Handoff YAML includes `schema_version: 1`
- [ ] No command auto-triggers another command without PM confirmation

---

## Dependencies & Prerequisites

- Claude Code with plugin support enabled
- compound-engineering plugin installed (for `/workflows:plan` handoff target)
- No external APIs, databases, or infrastructure required

---

## Future Considerations (Out of Scope)

- `/pm:ux-brief` — UX direction document for design handoff
- `/pm:legal` — Compliance and privacy checklist
- Integration with Linear or GitHub Issues for automatic story creation
- `/pm:retro` — Post-launch retrospective and learning capture
- Cross-plugin agent sharing (surfacing `pm-challenger` skill to compound-engineering's review pipeline)
- Stale context reconciliation in `product-context.yaml`

---

## References

- Brainstorm: `docs/brainstorms/2026-02-22-product-management-skill-brainstorm.md`
- compound-engineering plugin: `/Users/jeffclark/.claude/plugins/cache/every-marketplace/compound-engineering/2.30.0/`
- Brainstorming skill (structure reference): `skills/brainstorming/SKILL.md`
- Plan workflow (handoff target): `commands/workflows/plan.md`
- Agentic PRD: https://prodmoh.com/blog/agentic-prd
- Atlassian user stories: https://www.atlassian.com/agile/project-management/user-stories
- GTM planning: https://blog.hubspot.com/sales/gtm-strategy
- Product metrics: https://mixpanel.com/blog/product-metrics/
