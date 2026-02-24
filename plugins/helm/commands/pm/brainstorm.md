# /pm:brainstorm

**Note: The current year is 2026.**

Explore a product idea through structured dialogue before committing to requirements. Surfaces scope, rationale, and key decisions. Produces a brainstorm doc that feeds `/pm:prd`.

**Skill:** Load `pm-challenger` skill before beginning.

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Starts with `docs/` or ends with `.md` → file path to an existing brainstorm doc (load and continue from it)
- Otherwise → feature or topic name to start a fresh brainstorm

If $ARGUMENTS is empty, ask: "What product idea or feature would you like to explore?"

## Execution

### 1. Load context

Read `product-context.yaml` at the project root:
- **File exists and populated** → extract company, personas, tech_stack, and principles
- **File exists but empty** → continue; note: "product-context.yaml has no data — output will be generic. Populate it for richer results."
- **File not found** → ask: "No product-context.yaml found. Create one now, or continue without project context?"

### 2. Understand the idea

Ask questions **one at a time** using AskUserQuestion. Prefer multiple choice when natural options exist.

Cover in order:
1. Problem — What's broken or missing today? Who feels it?
2. Users — Which persona? What's their context?
3. Constraints — Timeline, tech, dependencies?
4. Success — How will you know this worked?

Stop when the idea is clear or the user says "proceed."

### 3. Run challenger review

Load the `pm-challenger` skill. Run a challenger review on the idea as described.

Domain for flag IDs: `brainstorm`

Do not proceed until all flags are resolved or overridden.

### 4. Explore approaches

Propose 2–3 concrete approaches. For each:

```
### Approach A: [Name]
[2–3 sentence description]

Pros:
- ...

Cons:
- ...

Best when: [circumstance]
```

Lead with a recommendation. Apply YAGNI — prefer the simpler approach.

Use AskUserQuestion to confirm which approach the user prefers.

### 5. Capture the design

Derive the topic slug from $ARGUMENTS: lowercase, spaces to hyphens, strip special characters.
If $ARGUMENTS is empty, derive from the feature name agreed in step 2.

Write the brainstorm doc to: `docs/brainstorms/YYYY-MM-DD-<topic-slug>-brainstorm.md`

```markdown
---
date: YYYY-MM-DD
topic: <topic-slug>
---

# [Feature Title]

## What We're Building
[1–2 paragraphs]

## Why This Approach
[Approaches considered, why this one was chosen]

## Key Decisions
- [Decision]: [Rationale]

## Resolved Challenges
[Any challenger flags raised and how they were addressed]

## Open Questions
- [Unresolved questions for the PRD phase]

## Next Steps
→ Run `/pm:prd` to define requirements
```

### 6. Update product-context.yaml

Append to `recent_artifacts` (cap at 20, remove oldest if over):

```yaml
- date: YYYY-MM-DD
  type: brainstorm
  topic: <topic-slug>
  path: docs/brainstorms/YYYY-MM-DD-<topic-slug>-brainstorm.md
  # Added YYYY-MM-DD by /pm:brainstorm
```

### 7. Handoff

Use AskUserQuestion:

**Question:** "Brainstorm captured. What's next?"

**Options:**
1. Run `/pm:prd` — define requirements from this brainstorm
2. Refine further — keep exploring
3. Done for now — return later
