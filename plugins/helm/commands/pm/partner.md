# /pm:partner

**Note: The current year is 2026.**

Full PM lifecycle orchestrator. Runs the complete sequence: brainstorm → PRD → stories → GTM → analytics. Each phase runs after the previous one completes. The PM reviews and approves at each gate before the next phase begins.

**Skill:** Load `pm-challenger` skill before beginning.

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Starts with `docs/` or ends with `.md` → file path to an existing artifact (enter at the appropriate phase)
- Otherwise → feature name; starts from brainstorm phase

If $ARGUMENTS is empty, ask: "What feature would you like to take through the full PM lifecycle?"

## Automation Mode

If $ARGUMENTS contains `--auto` (e.g., `feature-name --auto` or `--auto feature-name`):
- Strip `--auto` to derive the feature name or file path
- Activate automation mode for the full run:
  - Phase gates auto-accept and log `"Gate auto-accepted (automation mode)"` instead of asking
  - Challenger reviews still run, but flags are appended as a non-blocking `## Challenger Flags — Pending Review` section in each output artifact rather than blocking the pipeline
- Announce: "Running in automation mode. All gates will auto-accept. Challenger flags will be recorded for post-run review."

## Phase Detection

If $ARGUMENTS points to an existing file, detect entry point from filename or frontmatter `type`:
- `brainstorm` → start at Phase 2 (PRD)
- `prd` → start at Phase 3 (Stories)
- `stories` → start at Phase 4 (GTM)
- `gtm` → start at Phase 5 (Analytics)
- Otherwise → start at Phase 1 (Brainstorm)

Announce: "Detected existing [type] for [topic]. Starting from [phase name]."

## Execution

### Phase 0: Research
<!-- Mirrors: commands/pm/research.md — update both if logic changes -->

Runs only when entering at Phase 1 (feature name argument, not file path).

1. Extract topic string from `$ARGUMENTS`, stripping `--auto` if present before use.
2. (Interactive mode only) Print: `Phase 0: Research — [topic]`
3. Execute research steps:
   a. Call the Monterey AI MCP server `search_snippets` tool with the topic string,
      30-second timeout. On success with results: `customer_voice.status: ok`. On
      timeout, connection failure, or missing tool: `customer_voice.status: failed`,
      populate `error` field. On empty result: `customer_voice.status: no_results`.
   b. Run a web search for `[topic] competitors features pricing positioning`. With ≥2
      named competitors: `competitive_landscape.status: ok`. With exactly 1 or no
      usable results: `status: no_results`, `competitors: []`.
   c. Check for existing `research-context.yaml` at project root. If present and valid
      YAML and `pm_notes` is non-empty: carry forward the value. If present but invalid
      YAML or absent: `pm_notes = ""`.
   d. Write `research-context.yaml` to project root:
      - `generated_by: "pm:partner"`
      - `generated_at`: current UTC datetime in ISO 8601
      - All other fields from steps a–c
      - Do NOT write `citation_impact`
      - If YAML serialization fails: print error, do not write file, continue to
        Phase 0 Review / Phase 1.
4. If both `customer_voice` and `competitive_landscape` are `status: failed` or
   `no_results`: print exactly:
   `Warning: research returned no results for "[topic]". Continuing without research context.`
   Then proceed to Phase 1 (auto mode) or Phase 0 Review (interactive mode).
5. **In `--auto` mode:** run silently. Suppress all output. Print only the dual-failure
   warning above if applicable. Proceed directly to Phase 1 Brainstorm.
6. **In interactive mode:** proceed to Phase 0 Review below.

**Phase 0 Review (interactive mode only):**

Skip this review if both research sections are `failed` or `no_results` (warning was
already printed in step 4 above). Proceed to Phase 1.

Display the human-readable research summary:
- Customer voice: themes with mention counts + up to 5 attributed quotes (or failure
  note if `status: failed` or `no_results`)
- Competitive landscape: each competitor as a 2–3 sentence summary (or failure note)

Then invoke AskUserQuestion:
- Question: `"Research complete. Add notes before continuing, or proceed?"`
- Options: (1) Add notes, (2) Proceed

If "Add notes" is selected:
- Invoke a second AskUserQuestion with an open text field (user enters notes via the
  "Other" free-text option).
- Read the current `research-context.yaml` into memory. Update only the `pm_notes`
  field: if empty, set to the PM's text; if it already has content, append with a
  single newline separator.
- Re-serialize the entire document from the parsed structure (do not use string surgery
  on the raw file). Use YAML block scalar (`|`) for `pm_notes` if it contains
  newlines. Write the full file back. Do not alter any other field.

After notes are submitted, or if "Proceed" is selected: continue to Phase 1.
Do not re-display the research summary.

---

### Phase 1: Brainstorm
<!-- Mirrors: commands/pm/brainstorm.md — update both if logic changes -->

Execute the brainstorm command flow inline:
- Load product-context.yaml
- Gather idea via dialogue (one question at a time)
- Run challenger review (domain: `brainstorm`)
- Explore 2–3 approaches, confirm choice
- Write `docs/brainstorms/YYYY-MM-DD-<topic-slug>-brainstorm.md`
- Update product-context.yaml

**Gate:** Show brainstorm summary. Ask:

"Brainstorm complete. Ready to move to PRD?"

Options: Yes, proceed / Refine brainstorm further / Stop here

Do not proceed to Phase 2 until the PM confirms.

---

### Phase 2: PRD
<!-- Mirrors: commands/pm/prd.md — update both if logic changes -->

Execute the PRD command flow inline:
- Use brainstorm output as input context
- Run challenger review on requirements inputs (domain: `prd`)
- Invoke `prd-writer` agent
- Present draft, iterate on feedback
- Write `docs/prds/YYYY-MM-DD-<topic-slug>-prd.md`
- Update product-context.yaml

**Gate:** Show PRD summary. Ask:

"PRD complete. Ready to generate user stories?"

Options: Yes, proceed / Revise PRD first / Stop here

---

### Phase 3: User Stories
<!-- Mirrors: commands/pm/stories.md — update both if logic changes -->

Execute the stories command flow inline:
- Use PRD as input context
- Run challenger review on story-generation readiness (domain: `story`)
- Invoke `story-writer` agent
- Present story set, iterate on feedback
- Write `docs/stories/YYYY-MM-DD-<topic-slug>-stories.md`
- Update product-context.yaml

**Gate:** Show story count and epic summary. Ask:

"Stories complete. Proceed to GTM plan?"

Options: Yes, proceed / Revise stories first / Stop here

---

### Phase 4: GTM Plan
<!-- Mirrors: commands/pm/gtm.md — update both if logic changes -->

Execute the GTM command flow inline:
- Use PRD and brainstorm as input context
- Gather GTM inputs not already in source docs (one at a time)
- Run challenger review (domain: `gtm`)
- Invoke `gtm-strategist` agent
- Present draft, iterate on feedback
- Write `docs/gtm/YYYY-MM-DD-<topic-slug>-gtm.md`
- Update product-context.yaml

**Gate:** Show GTM plan summary. Ask:

"GTM plan complete. Proceed to analytics plan?"

Options: Yes, proceed / Revise GTM first / Stop here

---

### Phase 5: Analytics Plan
<!-- Mirrors: commands/pm/analytics.md — update both if logic changes -->

Execute the analytics command flow inline:
- Use PRD as input context
- Gather analytics inputs not already in source docs, including confirming framework choice (AARRR or HEART)
- Run challenger review (domain: `analytics`)
- Invoke `analytics-planner` agent with confirmed framework choice (AARRR or HEART)
- Present draft, iterate on feedback
- Write `docs/analytics/YYYY-MM-DD-<topic-slug>-analytics.md`
- Update product-context.yaml

**Gate:** Show analytics plan summary. Ask:

"Analytics plan complete. Ready to finalize the lifecycle?"

Options: Yes, proceed / Revise analytics first / Stop here

---

### Final Handoff

Write the handoff manifest to: `docs/handoffs/YYYY-MM-DD-<topic-slug>-handoff.md`

```markdown
---
title: [Feature Name] — Handoff
type: handoff
date: YYYY-MM-DD
topic: <topic-slug>
status: ready-for-engineering
---

# [Feature Name] — PM Handoff

## Artifacts

| Type | Path |
|------|------|
| Brainstorm | docs/brainstorms/YYYY-MM-DD-<topic-slug>-brainstorm.md |
| PRD        | docs/prds/YYYY-MM-DD-<topic-slug>-prd.md |
| Stories    | docs/stories/YYYY-MM-DD-<topic-slug>-stories.md |
| GTM plan   | docs/gtm/YYYY-MM-DD-<topic-slug>-gtm.md |
| Analytics  | docs/analytics/YYYY-MM-DD-<topic-slug>-analytics.md |

## Challenger Review Summary

Flags raised across all phases: [total count]
Flags overridden: [count]

[For each overridden flag:]
- [flag_prd_001] SCOPE CREEP — Override: [PM's rationale]

## Engineering Entry Point

`/workflows:plan docs/stories/YYYY-MM-DD-<topic-slug>-stories.md`
```

Present a lifecycle summary:

```
PM LIFECYCLE COMPLETE — [Feature Name]

Artifacts produced:
- Brainstorm: docs/brainstorms/YYYY-MM-DD-<topic-slug>-brainstorm.md
- PRD:        docs/prds/YYYY-MM-DD-<topic-slug>-prd.md
- Stories:    docs/stories/YYYY-MM-DD-<topic-slug>-stories.md
- GTM plan:   docs/gtm/YYYY-MM-DD-<topic-slug>-gtm.md
- Analytics:  docs/analytics/YYYY-MM-DD-<topic-slug>-analytics.md

Challenger flags raised: [total count across all phases]
Flags overridden: [count and IDs]

Ready for engineering handoff.
```

Use AskUserQuestion:

**Question:** "Lifecycle complete. What's next?"

**Options:**
1. Hand off to engineering — run `/workflows:plan` with the stories doc
2. Review a specific artifact — ask which one
3. Done — all artifacts are saved
