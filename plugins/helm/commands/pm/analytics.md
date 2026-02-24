# /pm:analytics

**Note: The current year is 2026.**

Define the analytics plan for a feature: primary metric, supporting metrics, measurement method, and success criteria. Uses AARRR for growth/acquisition features and HEART for UX/quality features.

**Skill:** Load `pm-challenger` skill before beginning.

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Starts with `docs/` or ends with `.md` → file path to an existing PRD or brainstorm doc
- Otherwise → feature name; will search `docs/prds/` then `docs/brainstorms/` for a match

If $ARGUMENTS is empty, ask: "Which feature's analytics plan should I build? Provide a path or feature name."

## Execution

### 0. Load research context (if present)

Check for `research-context.yaml` at the project root:

- **File absent** → continue silently. No warning, no mention.
- **File present but invalid YAML** → print exactly:
  `Warning: research-context.yaml could not be parsed — running without research context.`
  Then continue as if absent.
- **File present and valid** → read the full file contents into the command's execution
  context. This context is available to both artifact generation and challenger review
  steps in this session.
  - Check staleness: if `generated_at` is more than 7 days before current UTC datetime,
    print exactly:
    `Warning: research context for "[topic]" is N days old (generated YYYY-MM-DD). Consider re-running /pm:research to refresh findings.`
    where `[topic]` is `research_context.topic` from the file, and N is the whole
    number of days elapsed.
  - If `generated_at` cannot be parsed: print exactly:
    `Warning: research context for "[topic]" — generated date unknown. Consider re-running /pm:research to refresh findings.`
    If `topic` also cannot be read, omit the topic clause.

Staleness warnings do not block execution. Continue immediately after printing.

### 1. Load context

Read `product-context.yaml` at the project root:
- **File exists and populated** → extract company, personas, and principles
- **File exists but empty** → continue; note: "product-context.yaml has no data — output will be generic. Populate it for richer results."
- **File not found** → ask: "No product-context.yaml found. Create one now, or continue without project context?"

Read the source doc. Extract: feature goals, target user, any metrics or success criteria already defined.

### 2. Gather analytics inputs

Ask one at a time using AskUserQuestion:

1. Feature type — Is this primarily a growth/acquisition feature or a UX/quality improvement? (Determines AARRR vs HEART framework)
2. Primary metric — What is the single number that tells you if this worked?
3. Baseline — Do you have current data to compare against?
4. Instrumentation — Is there existing event tracking infrastructure, or does this need to be set up?

Skip questions already answered in the source doc.

### 3. Run challenger review

Load the `pm-challenger` skill. Review the analytics inputs against the metrics anti-patterns reference.

Domain for flag IDs: `analytics`

Common flags: vanity metric as primary, proxy metric without explanation, unmeasurable criteria, framework mismatch.

Do not proceed until all flags are resolved or overridden.

### 4. Invoke analytics-planner agent

Spawn an `analytics-planner` agent with: product-context.yaml contents, source doc, gathered analytics inputs, resolved challenger flags, confirmed framework choice (AARRR or HEART).

Wait for the agent to return the analytics plan draft.

### 5. Present and iterate

Show the draft. Ask: "Does this capture the right metrics and measurement approach? Any adjustments?"

Iterate on feedback. Do not re-run the challenger review on minor edits.

### 6. Write analytics plan

Derive topic slug from the source doc filename or feature name.

Write to: `docs/analytics/YYYY-MM-DD-<topic-slug>-analytics.md`

### 7. Update product-context.yaml

Append to `recent_artifacts` (cap at 20, remove oldest if over):

```yaml
- date: YYYY-MM-DD
  type: analytics
  topic: <topic-slug>
  path: docs/analytics/YYYY-MM-DD-<topic-slug>-analytics.md
  # Added YYYY-MM-DD by /pm:analytics
```

### 8. Handoff

Use AskUserQuestion:

**Question:** "Analytics plan written. What's next?"

**Options:**
1. Run `/pm:stories` — break into engineering stories
2. Run `/pm:gtm` — build the go-to-market plan
3. Done for now — return later
