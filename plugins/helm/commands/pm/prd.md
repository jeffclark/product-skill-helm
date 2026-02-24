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

Spawn a `prd-writer` agent with all context: product-context.yaml contents, brainstorm doc (if any), clarifications gathered, resolved challenger flags.

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
