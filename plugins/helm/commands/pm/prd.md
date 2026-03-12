# /pm:prd

**Note: The current year is 2026.**

Generate a Product Requirements Document from a feature description or existing brainstorm doc. The PRD is structured for both human review and direct consumption by AI engineering agents.

**Skill:** Load `pm-challenger` skill before beginning.

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Starts with `docs/` or ends with `.md` → file path to existing brainstorm or PRD doc
- Otherwise → feature name to generate PRD from scratch

If $ARGUMENTS is empty, check `docs/brainstorms/` for a recent brainstorm (last 14 days). If found, ask which to use. If none, ask: "What feature do you want to define requirements for?"

## Execution

### Pre-step: Template Detection

Check for `prd-template.md` in the project root:

- **File found and contains at least one `##` heading:**
  Print exactly: `Template found: prd-template.md — generating PRD using your template structure.`
  Read the full contents of `prd-template.md` into session context as `template_contents`.
  Set `template_mode: true`. Continue to Step 0.
- **File found but empty or contains no `##` headings:**
  Print exactly: `Warning: prd-template.md has no ## headings — falling back to default PRD structure.`
  Continue to Step 0 with `template_mode: false`.
- **File not found:** Continue to Step 0 silently. No message, no mention of templates.
  `template_mode: false`.

This step runs before research context is loaded and before any other input is processed.

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
- **File exists and populated** → extract company, personas, tech_stack, and principles
- **File exists but empty** → continue; note: "product-context.yaml has no data — output will be generic. Populate it for richer results."
- **File not found** → ask: "No product-context.yaml found. Create one now, or continue without project context?"

If $ARGUMENTS points to a brainstorm doc, read it and extract: What We're Building, chosen approach, key decisions, open questions.

If no brainstorm exists, ask 2–3 clarifying questions one at a time to establish: target user, core problem, and primary success metric.

### 2. Run challenger review

Load the `pm-challenger` skill. Review the requirements inputs before generating the PRD.

Domain for flag IDs: `prd`

If flags are raised, wait for the user to address or override each one. Do not proceed to generation until all flags are resolved or overridden.

### 3. Invoke prd-writer agent

**If `template_mode: true`:**
Spawn the `prd-writer` agent with all standard context (product-context.yaml, brainstorm doc,
clarifications, resolved challenger flags) PLUS:
- `template_mode: true`
- `template_contents`: the full text of `prd-template.md` read in the Pre-step

The agent will identify gap sections and run an interactive gap-filling dialogue before
writing the output file. Wait for the agent to complete the full gap dialogue and signal
that the output file has been written before proceeding to Step 4.

**If `template_mode: false`:**
Spawn the `prd-writer` agent with standard context only (current behavior unchanged).
Wait for the agent to return the draft PRD.

### 4. Present draft and iterate

Show the draft PRD to the user. Ask:

"Does this capture the requirements correctly? Any sections to adjust?"

Incorporate feedback and regenerate affected sections as needed. Do not re-run the challenger review on minor edits — only re-run if scope or success criteria materially change.

### 5. Write PRD

Derive topic slug from $ARGUMENTS or feature name: lowercase, spaces to hyphens, strip special characters.

Write to: `docs/prds/YYYY-MM-DD-<topic-slug>-prd.md`

### 6. Update product-context.yaml

Append to `recent_artifacts` (cap at 20, remove oldest if over):

```yaml
- date: YYYY-MM-DD
  type: prd
  topic: <topic-slug>
  path: docs/prds/YYYY-MM-DD-<topic-slug>-prd.md
  # Added YYYY-MM-DD by /pm:prd
```

### 7. Handoff

Use AskUserQuestion:

**Question:** "PRD written. What's next?"

**Options:**
1. Run `/pm:stories` — break into engineering-ready user stories
2. Run `/pm:gtm` — build the go-to-market plan
3. Run `/pm:partner` — run the full remaining lifecycle
4. Done for now — return later
