# /pm:stories

**Note: The current year is 2026.**

Break a PRD into engineering-ready user stories. Stories are structured for direct consumption by compound-engineering agents — each story is independently implementable and includes explicit acceptance criteria.

**Skill:** Load `pm-challenger` skill before beginning.

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Starts with `docs/` or ends with `.md` → file path to an existing PRD doc
- Otherwise → feature name; will search `docs/prds/` for a matching PRD

If $ARGUMENTS is empty, check `docs/prds/` for a recent PRD (last 14 days). If found, ask which to use. If none, ask: "Which PRD should I break into stories? Provide the path or feature name."

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
- **File exists and populated** → extract personas and tech_stack
- **File exists but empty** → continue; note: "product-context.yaml has no data — output will be generic."
- **File not found** → continue without project context

Read the PRD. Extract: user types, functional requirements, acceptance criteria, constraints, out-of-scope items.

### 2. Run challenger review

Load the `pm-challenger` skill. Review the PRD for story-generation readiness.

Domain for flag IDs: `story`

Common flags at this stage:
- PRD acceptance criteria are not independently testable → UNDER-DEFINED REQUIREMENTS
- Stories would require shared state between two features not yet built → SCOPE CREEP
- No definition of "done" for the epic → UNDER-DEFINED REQUIREMENTS

Do not proceed until all flags are resolved or overridden.

### 3. Invoke story-writer agent

Spawn a `story-writer` agent with: PRD content, product-context.yaml contents, resolved challenger flags.

Wait for the agent to return the story set.

### 4. Present and iterate

Show the stories. Ask: "Do these cover the requirements? Any missing stories or adjustments?"

Incorporate feedback. Do not regenerate stories that weren't changed.

### 5. Write stories

Derive topic slug from the PRD filename or $ARGUMENTS.

Write to: `docs/stories/YYYY-MM-DD-<topic-slug>-stories.md`

### 6. Update product-context.yaml

Append to `recent_artifacts` (cap at 20, remove oldest if over):

```yaml
- date: YYYY-MM-DD
  type: stories
  topic: <topic-slug>
  path: docs/stories/YYYY-MM-DD-<topic-slug>-stories.md
  # Added YYYY-MM-DD by /pm:stories
```

### 7. Handoff

Use AskUserQuestion:

**Question:** "Stories written. What's next?"

**Options:**
1. Hand off to engineering — run `/workflows:plan` with the stories doc
2. Run `/pm:gtm` — build the go-to-market plan
3. Run `/pm:analytics` — define the analytics plan
4. Done for now — return later
