# /pm:gtm

**Note: The current year is 2026.**

Build a go-to-market plan for a feature. Covers messaging, rollout sequencing, stakeholder communication, and launch criteria.

**Skill:** Load `pm-challenger` skill before beginning.

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Starts with `docs/` or ends with `.md` → file path to an existing PRD or brainstorm doc
- Otherwise → feature name; will search `docs/prds/` then `docs/brainstorms/` for a match

If $ARGUMENTS is empty, ask: "Which feature's GTM plan should I build? Provide a path or feature name."

## Execution

### 1. Load context

Read `product-context.yaml` at the project root:
- **File exists and populated** → extract company, personas, and principles
- **File exists but empty** → continue; note: "product-context.yaml has no data — output will be generic. Populate it for richer results."
- **File not found** → ask: "No product-context.yaml found. Create one now, or continue without project context?"

Read the source doc (PRD or brainstorm). Extract: feature summary, target user, launch timeline if specified.

### 2. Gather GTM inputs

Ask questions one at a time using AskUserQuestion to fill gaps not in the source doc:

1. Launch scope — full release, beta first, or percentage rollout?
2. Timeline — target launch date or phase dates?
3. Stakeholders — which teams need communication (sales, support, success, legal)?
4. Messaging anchor — what is the single most important thing the user should understand?

Skip questions already answered in the source doc.

### 3. Run challenger review

Load the `pm-challenger` skill. Review the GTM inputs against the GTM gap patterns reference.

Domain for flag IDs: `gtm`

Common flags at this stage: missing rollout sequence, missing stakeholder comms, no rollback trigger defined, undifferentiated messaging.

Do not proceed until all flags are resolved or overridden.

### 4. Invoke gtm-strategist agent

Spawn a `gtm-strategist` agent with: product-context.yaml contents, source doc, gathered GTM inputs, resolved challenger flags.

Wait for the agent to return the GTM plan draft.

### 5. Present and iterate

Show the draft. Ask: "Does this cover the launch plan? Any sections to adjust?"

Iterate on feedback. Do not re-run the challenger review on minor edits.

### 6. Write GTM plan

Derive topic slug from the source doc filename or feature name.

Write to: `docs/gtm/YYYY-MM-DD-<topic-slug>-gtm.md`

### 7. Update product-context.yaml

Append to `recent_artifacts` (cap at 20, remove oldest if over):

```yaml
- date: YYYY-MM-DD
  type: gtm
  topic: <topic-slug>
  path: docs/gtm/YYYY-MM-DD-<topic-slug>-gtm.md
  # Added YYYY-MM-DD by /pm:gtm
```

### 8. Handoff

Use AskUserQuestion:

**Question:** "GTM plan written. What's next?"

**Options:**
1. Run `/pm:analytics` — define the analytics plan
2. Run `/pm:stories` — generate engineering stories
3. Done for now — return later
