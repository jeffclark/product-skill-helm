---
title: "feat: Add market research phase (/pm:research)"
type: feat
date: 2026-02-24
stories: docs/stories/2026-02-24-market-research-phase-stories.md
prd: docs/prds/2026-02-24-market-research-phase-prd.md
---

# feat: Add market research phase (`/pm:research`)

## Overview

Add a `/pm:research` command to Helm that gathers external market signal — customer
feedback via Reforge Insights MCP and competitive intelligence via web search — and
makes it available to all PM artifact commands and challenger reviews. This is the
first feature that makes Helm's challengers market-aware rather than logic-aware.

**6 stories across 5 epics. Implement in dependency order.**

> **Implementation note:** STORY-004 and STORY-005 modify the same block in the same
> file with a single `if --auto / else interactive` branch. They may be implemented as
> one story if preferred. The dependency order is unchanged either way.

## Problem Statement

Helm artifacts are generated from internal assumptions with no external market signal.
Challengers can push back on logic but not on market reality — they have no access to
customer voice data or competitive positioning. The result: PRDs and brainstorms that
are internally coherent but externally uninformed.

## Proposed Solution

A new `/pm:research [topic]` command that:

1. Calls Monterey AI MCP `search_snippets` for customer voice data (Reforge Insights)
2. Runs competitive web research for feature sets, positioning, pricing
3. Writes findings to `research-context.yaml` at the project root
4. All existing commands load this file at startup; challengers inject it and require
   at least one sourced citation per review

`/pm:partner` runs research automatically as phase 0 before the brainstorm phase.

## Architecture

This is a **Claude Code plugin** — all commands are markdown instruction files that
Claude executes as skills. There is no traditional code (Ruby, Python, JS).

Implementation means creating and modifying `.md` files. Manual validation is the test
harness for prompting stories. "Acceptance" means the instruction produces the correct
Claude behavior, verified by running the command.

### Files to Create

| File | Story | Purpose |
|---|---|---|
| `plugins/helm/commands/pm/research.md` | STORY-001 | New `/pm:research` slash command |
| `docs/research-context-schema.md` | STORY-002 | Canonical v1 schema reference |
| `docs/fixtures/research-context-sample.yaml` | STORY-006 | Manual validation fixture |

### Files to Modify

| File | Story | Change |
|---|---|---|
| `plugins/helm/commands/pm/brainstorm.md` | STORY-003 | Add research load + staleness step |
| `plugins/helm/commands/pm/prd.md` | STORY-003 | Add research load + staleness step |
| `plugins/helm/commands/pm/stories.md` | STORY-003 | Add research load + staleness step |
| `plugins/helm/commands/pm/gtm.md` | STORY-003 | Add research load + staleness step |
| `plugins/helm/commands/pm/analytics.md` | STORY-003 | Add research load + staleness step |
| `plugins/helm/commands/pm/partner.md` | STORY-004 + 005 | Add phase 0 block |
| `plugins/helm/skills/pm-challenger/SKILL.md` | STORY-006 | Add research injection section |
| `plugins/helm/CLAUDE.md` | — | Add `/pm:research` to commands list |
| `README.md` | — | Add `/pm:research` to commands table and output structure |

---

## Implementation Phases

### Phase 1 — Schema (STORY-002)

**Create `docs/research-context-schema.md`**

This is the unblocking foundation. Every subsequent story references field names,
status enums, and constraints from this document. No other files are modified.

The document must define:

- All top-level fields: `topic`, `generated_at`, `generated_by`, `customer_voice`,
  `competitive_landscape`, `pm_notes`
- `customer_voice` sub-fields: `status` enum (`ok | failed | no_results`), `error`
  (present only when `status: failed`), `source`, `query`, `snippet_count`, `themes`
  (with `label`, `summary`, `mention_count`), `snippets` (with `text`, `source` enum,
  `date`, `sentiment` enum)
- `competitive_landscape` sub-fields: `status` enum, `error`, `competitors` (with
  `name`, `features`, `positioning`, `pricing`, `source_url`)
- `pm_notes`: always present as a string; empty string if no notes. When multiple
  notes have been appended, they are separated by a single newline within a YAML
  block scalar (use `|` literal block style).
- Structural constraints:
  - `generated_at` required + valid ISO 8601 UTC
  - `competitors` must have ≥1 entry when `status: ok`
  - `snippets` may be empty list when `status: no_results`
  - **1-competitor rule:** exactly 1 named competitor from web search → `status: no_results`,
    `competitors: []`. Competitive positioning requires ≥2 named competitors. This resolves
    the conflict between the ≥2 threshold in the command spec and the ≥1 schema constraint.
  - `research-context.yaml` lives at the project root — same directory as
    `product-context.yaml`.
- `generated_by` enum: valid values for v1 are `"pm:research"` and `"pm:partner"`.
  No other values are valid. No current downstream behavior branches on this field;
  it exists for Phase 2 provenance. Treat as reserved metadata.
- **`citation_impact` is deferred to Phase 2 and must not be written in v1.**
- Complete annotated YAML example with both sections at `status: ok`

---

### Phase 2 — `/pm:research` Command (STORY-001)

**Create `plugins/helm/commands/pm/research.md`**

Follow the universal command header pattern from all existing `commands/pm/*.md` files:

```markdown
# /pm:research

**Note: The current year is 2026.**

[Description]

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Otherwise → topic to research
```

**Execution steps:**

**1. Parse topic**
- If `$ARGUMENTS` is empty: AskUserQuestion "What topic should I research?"
- Use `$ARGUMENTS` as the topic string as-is. Do not handle or strip `--auto` here —
  that flag belongs to `/pm:partner` and is stripped before the topic reaches this command.

**2. Call Reforge Insights MCP**
```
Call the Monterey AI MCP server search_snippets tool with the topic string as
the semantic search query. Apply a 30-second timeout.

- Success with results → customer_voice.status: ok; populate snippets, themes,
  snippet_count
- Timeout or connection failure → customer_voice.status: failed; populate error
  field with failure message; continue to step 3
- Tool not available in environment → customer_voice.status: failed; error:
  "Monterey AI MCP tool not available — verify MCP configuration"; continue
- Empty result → customer_voice.status: no_results; continue
```
Treat timeout, connection failure, and missing tool as the same `status: failed` path.
Do not halt on MCP failure.

**3. Run competitive web search**
```
Run a web search for: [topic] competitors features pricing positioning

- ≥2 named competitors found → competitive_landscape.status: ok; populate
  competitors list (name, features 2–4 sentences, positioning 1–2 sentences,
  pricing or null, source_url or null)
- Exactly 1 named competitor found → competitive_landscape.status: no_results;
  competitors: [] (insufficient for competitive positioning)
- No usable results or web search error → competitive_landscape.status: no_results;
  competitors: []
```

**4. Carry forward `pm_notes`**
```
Before writing, check for existing research-context.yaml at project root:
- If present and valid YAML and contains non-empty pm_notes → capture the value
- If present but invalid YAML → treat as absent; pm_notes = ""
- If absent or pm_notes is empty → pm_notes = ""
```

**5. Write `research-context.yaml`**
```
Construct the YAML document per docs/research-context-schema.md:
- topic: [topic string]
- generated_at: [current UTC datetime in ISO 8601]
- generated_by: "pm:research"
- customer_voice: [from step 2]
- competitive_landscape: [from step 3]
- pm_notes: [from step 4; use YAML block scalar (|) if multi-line]

Write research-context.yaml to the project root, overwriting any existing file.
Do NOT write citation_impact.

If YAML serialization fails: print the error, print the human-readable summary
(step 6) from in-memory data only, and do not write the file. Never write a
partial file.
```

**6. Print human-readable summary**
```
## Research Summary — [topic]

### Customer Voice (Reforge Insights)
[If status: ok]
Key themes:
- [theme label]: [theme summary] (N mentions)
...

Top quotes (up to 5):
- "[snippet text]" — [source], [date]
...

[If status: failed] Customer voice research failed: [error message]
[If status: no_results] No customer voice data found for this topic.

### Competitive Landscape
[If status: ok]
[Competitor name]: [features summary] [positioning summary] [pricing if available]
...

[If status: no_results] No competitive data found for this topic.
```

**Important constraints:**
- Do NOT run a challenger review (this is a data-gathering command, not artifact-generating)
- Do NOT update `product-context.yaml` (research is not an artifact type in that file)
- Do NOT write `citation_impact` field (Phase 2)

---

### Phase 3 — Existing Command Integration + Partner Phase 0 Silent (Parallel)

#### STORY-003: Research load + staleness check (5 command files)

**Modify:** `brainstorm.md`, `prd.md`, `stories.md`, `gtm.md`, `analytics.md`

Insert the following as a new `### 0. Load research context` step before the existing
`### 1. Load context` step in each file. Use `### 0.` numbering — do not renumber the
existing steps.

```markdown
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
    where `[topic]` is `research_context.topic` from the file, and N is the whole number
    of days elapsed.
  - If `generated_at` cannot be parsed: print exactly:
    `Warning: research context for "[topic]" — generated date unknown. Consider re-running /pm:research to refresh findings.`
    If `topic` also cannot be read, omit the topic clause.

Staleness warnings do not block execution. Continue immediately after printing.
```

This step is identical across all five files.

#### STORY-004: Partner phase 0 — silent execution

**Modify:** `plugins/helm/commands/pm/partner.md`

Insert the following block **before** the existing `### Phase 1: Brainstorm` block.
The phase 0 block only runs when `$ARGUMENTS` is a feature name — the existing Phase
Detection section handles file-path entry points, which skip phase 0.

```markdown
### Phase 0: Research
<!-- Mirrors: commands/pm/research.md — update both if logic changes -->

Runs only when entering at Phase 1 (feature name argument, not file path).

1. Extract topic string from `$ARGUMENTS`, stripping `--auto` if present before use.
2. (Interactive mode only) Print: `Phase 0: Research — [topic]`
3. Execute research steps (identical to /pm:research steps 2–5):
   - Call Monterey AI MCP search_snippets with topic string, 30-second timeout
   - Run competitive web search (apply same 1-competitor threshold: ≥2 required for ok)
   - Carry forward pm_notes from existing research-context.yaml if present and valid
     YAML; treat as absent if invalid YAML; treat as absent if file missing
   - Write research-context.yaml to project root with generated_by: "pm:partner"
4. If both customer_voice and competitive_landscape are status: failed or no_results:
   print: `Warning: research returned no results for "[topic]". Continuing without
   research context.` Then proceed to Phase 0 Review (interactive) or Phase 1 (auto).
5. In --auto mode: run silently. Suppress all output. Print only the dual-failure
   warning above if applicable. Proceed directly to Phase 1 Brainstorm.
6. In interactive mode: proceed to Phase 0 Review (below).
```

---

### Phase 4 — Partner Interactive Review (STORY-005)

**Modify:** `plugins/helm/commands/pm/partner.md`

Add the Phase 0 Review block immediately after the Phase 0 execution block above
(still within Phase 0, before Phase 1):

```markdown
**Phase 0 Review (interactive mode only):**

If both research sections are failed/no_results: skip this review. The warning
was already printed in step 4 above. Proceed to Phase 1.

Display the human-readable research summary (same format as /pm:research standalone
terminal output: customer voice themes + up to 5 attributed quotes, competitor summaries,
failure notes inline for any failed section).

Then invoke AskUserQuestion:
- Question: "Research complete. Add notes before continuing, or proceed?"
- Options: (1) Add notes, (2) Proceed

If "Add notes" is selected:
  Invoke a second AskUserQuestion with an open text field (no options — use the
  "Other" free-text input).
  Read the current research-context.yaml into memory. Update only the pm_notes field:
  - If pm_notes is empty: set to the PM's text
  - If pm_notes has content: append PM's text with a single newline separator
  Re-serialize the entire document from the parsed structure using the schema in
  docs/research-context-schema.md. Write the full file back. Do not alter any other
  field. Use YAML block scalar (|) for pm_notes if it contains newlines.

After notes are submitted, or if "Proceed" is selected:
  Continue to Phase 1 (Brainstorm). Do not re-display the research summary.
```

---

### Phase 5 — Challenger Research Context Injection (STORY-006)

**Modify:** `plugins/helm/skills/pm-challenger/SKILL.md`

Add a new section after the existing `## Pattern References` section (end of file).
The new section governs conditional behavior — it does not change existing rules.

```markdown
## Research Context Injection

When `research-context.yaml` is loaded in the current session and at least one
section has `status: ok`, apply the following rules before generating any challenger
review:

**1. Inject research context into the review**
Include the full parsed contents of `research-context.yaml` in the context available
to the challenger. The challenger has access to all fields: customer voice themes,
snippets with attribution, competitive landscape entries.

**2. Require at least one sourced citation**
The review must include at least one flag that:
- Names a specific external data source (e.g., "Reforge Insights," a named competitor)
- Cites a specific data point (a verbatim quote, a mention count, a named competitor
  feature or pricing detail)

Generic references ("customer research suggests...", "competitors have this...") do
not satisfy the requirement.

Acceptable sources narrow dynamically:
- If only `competitive_landscape` is `ok`: citations must come from named competitors
- If only `customer_voice` is `ok`: citations must come from Reforge snippets
- If both are `ok`: either source is acceptable

In dual-reviewer phases (Cagan + Jobs in PRD review): the citation requirement is
satisfied by the combined output. One citation in either reviewer's section satisfies
the requirement for the phase. Jobs is not independently required to produce a research
citation given his narrow scope mandate.

**3. Research-sourced flag format**
Each flag that cites research context must include two components in the flag body:
(1) the cited gap or contradiction between the research finding and the artifact
(2) a concrete recommendation for updating the artifact

The source name and data point must appear in the flag body itself — not only in the
recommendation section. Example:

```
[flag_prd_001] SCOPE CREEP
Reforge Insights snippet (App Store, 3 weeks ago): 'onboarding takes too long' —
12 mentions. Current PRD has no onboarding time requirement.
Recommendation: Add a success criterion to FR-002 specifying maximum
time-to-first-value.
```

**4. Caps and limits**
The citation requirement does not increase the maximum flag count. The existing cap
of 5 flags per reviewer per review remains. Research-sourced flags count toward
this cap.

If all 5 flag slots would be filled by internal flags and a research-sourced flag is
required: drop the lowest-severity internal flag to make room. Note the substitution
explicitly: `[Internal flag dropped to accommodate required research citation.]`

**When research context is absent or all-failed:**
If `research-context.yaml` is not loaded, or all sections are `failed` or
`no_results`, do not inject research context and do not add the citation directive.
Run the review as if no research context exists. Do not reference or note the
absence of research context in the review output.
```

**Create `docs/fixtures/research-context-sample.yaml`** — a realistic sample file
for manual validation. Must have both sections at `status: ok` with:
- 2-3 customer voice themes, 3-5 snippets with realistic attribution (App Store dates,
  Gong labels), mix of sentiment
- 2-3 named competitors with feature summaries, positioning, and pricing

**Manual validation (acceptance gate for STORY-006):**

Run 3 challenger reviews using the fixture (at least 1 PRD review, 1 brainstorm review).

**Pass criteria (all 3 runs must pass):**
- Each run includes ≥1 flag that names an external source and cites a specific data point
- Each research-sourced flag includes a concrete artifact recommendation

**Required test case — cap substitution:**
In at least 1 run, configure the review to produce 5 natural internal flags (use a
deliberately weak artifact). Verify that the citation requirement causes the
lowest-severity internal flag to be dropped, the research-sourced flag takes its place,
and the substitution note appears. Document which flag was dropped and the note text.

Document all 3 run results before closing STORY-006.

---

### Housekeeping (after all stories)

**Update `plugins/helm/CLAUDE.md`** — add `/pm:research` to the Commands list:
```markdown
- `/pm:research` — Gather market context: customer voice (Reforge) and competitive
  intelligence. Writes `research-context.yaml` for use by all other commands.
```

**Update `README.md`** — add to the commands table:
```markdown
| `/pm:research` | Gather customer voice (Reforge Insights MCP) and competitive
  intelligence. Writes `research-context.yaml` to project root. |
```

Add `research-context.yaml` to the Output Structure section. Include a note:
`research-context.yaml` is a project-generated file. Add it to `.gitignore` if you
do not want to commit research findings to source control.

---

## Key Constraints

| Constraint | Detail |
|---|---|
| `citation_impact` must NOT be written | Phase 2. Documented as deferred in `docs/research-context-schema.md`. |
| No challenger review in `/pm:research` | It's a data-gathering command, not artifact-generating. |
| No `product-context.yaml` update in `/pm:research` | Research is not an artifact type tracked in that file. |
| `research-context.yaml` is overwrite-on-run | Only `pm_notes` is preserved across re-runs. `product-context.yaml` is append-only — different contract. |
| Phase 0 only runs for feature name entry | File-path arguments to `/pm:partner` skip phase 0 (established by existing Phase Detection logic). |
| Mirror comment convention in `partner.md` | Phase 0 block carries `<!-- Mirrors: commands/pm/research.md — update both if logic changes -->` |
| STORY-006 is a prompting story | Deliverable is SKILL.md modification + fixture + 3 documented test runs including cap substitution case. |
| `research.md` does not handle `--auto` | The `--auto` flag is a `/pm:partner` concept. Partner strips it before passing the topic. |

---

## Acceptance Criteria

### Functional
- [ ] `/pm:research [topic]` is invocable and writes a valid `research-context.yaml`
- [ ] MCP failure does not halt the command — competitive research continues independently
- [ ] Both-sections failure produces the correct warning; file is still written with failure status
- [ ] `pm_notes` is carried forward when `/pm:research` re-runs over an existing file
- [ ] `pm_notes` carry-forward treats a present-but-unparseable existing file as absent
- [ ] All 5 existing commands load `research-context.yaml` silently when absent
- [ ] Invalid YAML in `research-context.yaml` prints exactly the specified warning and continues
- [ ] Staleness warning includes the topic name and prints when `generated_at` > 7 days; does not block execution
- [ ] `/pm:partner` runs phase 0 before brainstorm for feature-name arguments
- [ ] `--auto` flag is stripped from topic in partner phase 0 before research executes
- [ ] `--auto` mode suppresses all phase 0 output except the dual-failure warning
- [ ] Interactive phase 0 review surfaces the research summary and offers notes input
- [ ] PM notes are appended to `pm_notes` via read-modify-write; other fields unchanged after write
- [ ] Challenger cites ≥1 named external source per review when research context is present with ≥1 `ok` section (manual validation, 3 runs, all passing)
- [ ] Cap substitution tested: research citation replaces lowest-severity internal flag when cap is reached
- [ ] Challenger runs cleanly when research context is absent or all-failed, with no mention of missing context

### Non-Functional
- [ ] `research-context.yaml` is valid YAML — if serialization fails, write is aborted with error message and no partial file is written
- [ ] `docs/research-context-schema.md` exists and is the canonical reference
- [ ] `docs/fixtures/research-context-sample.yaml` exists and is schema-valid
- [ ] `plugins/helm/CLAUDE.md` lists `/pm:research`
- [ ] `README.md` commands table includes `/pm:research` and `.gitignore` note

---

## SpecFlow Analysis — Key Resolutions

These gaps were identified by automated flow analysis and review, and resolved here.
Implementers should treat these as additive constraints on the stories.

**competitive_landscape `failed` vs `no_results`** — Treat web search connection/tool
errors as `status: failed` with `error` field populated. Treat web search success with
<2 named competitors (including exactly 1) as `status: no_results`. Mirrors the MCP
pattern. (Resolves Gap 1)

**1-competitor threshold** — Exactly 1 named competitor = `status: no_results`,
`competitors: []`. Competitive positioning requires ≥2 named entries. This resolves
the conflict between the ≥2 threshold in the command spec and the ≥1 schema constraint
for `ok` state. The schema constraint (`competitors` must have ≥1 entry when `status:
ok`) is correct — it just means you need ≥2 to reach `ok`. (New resolution)

**pm_notes carry-forward with corrupted file** — If the existing `research-context.yaml`
is present but cannot be parsed as valid YAML, treat it as absent: carry forward an
empty string. This mirrors the STORY-003 load-step behavior for consistency. (New resolution)

**pm_notes YAML encoding** — Use YAML block scalar (`|`) for `pm_notes` when it
contains newlines. On append, the full `pm_notes` value (original + newline + new text)
is re-serialized using block scalar style. Plain empty string stays as `pm_notes: ""`.
(New resolution)

**pm_notes read-modify-write in STORY-005** — Re-serialize the entire document from
the parsed structure. Do not attempt string surgery on the raw file. Any field not
recognized in the v1 schema should be passed through unchanged. The write is a full
file overwrite after in-memory modification. (New resolution)

**Atomic write** — There is no atomic write requirement. Claude has no syscall-level
rename primitive. The actual protection is: if YAML serialization fails, print the
error and do not write the file. Never write a partial file. This is the enforceable
contract. (Resolves Gap 3)

**pm_notes carry-forward in partner phase 0** — YES. Phase 0 in `/pm:partner` carries
forward `pm_notes` identically to `/pm:research`, including the invalid-YAML-treat-as-absent
rule. (Resolves Gap 4)

**Inline phase execution in partner** — When partner executes phases 1–5 inline, those
command blocks do NOT re-run the STORY-003 load step independently. Partner's phase 0
satisfies the research context load requirement for all subsequent inline phases.
Research context loaded in phase 0 is held in the session and passed to all downstream
challenger reviews. (Resolves Gap 7 and Gap 8)

**Multi-session load** — The STORY-003 load step must instruct Claude to read the full
file contents into the command's execution context — not merely verify the file exists.
Claude Code sessions are independent; the file is not automatically in context from a
prior session. The instruction uses "read the full file contents into the command's
execution context" for this reason. (New resolution)

**`generated_by` when partner writes the file** — Use `"pm:partner"` (not `"pm:research"`)
when `/pm:partner` phase 0 generates the file. The `generated_by` field has no current
behavioral consumer; it exists for Phase 2 provenance. (Resolves Gap 22)

**Staleness warning format** — The canonical format includes the topic name:
`Warning: research context for "[topic]" is N days old (generated YYYY-MM-DD). Consider re-running /pm:research to refresh findings.`
This applies consistently across all 5 command files. The SpecFlow resolution
supersedes the shorter format in the original STORY-003 acceptance criteria. (New resolution)

**Citation requirement in dual-reviewer PRD phase** — "At least one citation" means
across the combined Cagan + Jobs output. One citation in Cagan's section satisfies
the requirement for the phase. (Resolves Gap 19)

**Citation directive when only one section is `ok`** — Acceptable sources narrow
dynamically. If only `competitive_landscape` is `ok`, citations must come from named
competitors. If only `customer_voice` is `ok`, citations must come from Reforge
snippets. Challenger does not fabricate cross-section citations. (Resolves Gap 17)

**Flag cap (5) vs citation requirement** — Citation requirement takes precedence. If
5 internal flag slots are full and a research-sourced flag is required, drop the
lowest-severity internal flag to make room. The challenger notes this substitution
with a specific line in the review output. (Resolves Gap 18)

**AskUserQuestion freeform text** — The "Other" option in AskUserQuestion supports
free text input. STORY-005's pm_notes collection uses this mechanism. (Resolves Gap 14)

**Phase 0 freshness check** — Phase 0 always re-runs research unconditionally. No
freshness check. Deliberate design choice: partner is the canonical entry point for
fresh lifecycle runs. (Resolves Gap 4 — conscious decision)

**Topic mismatch when loading** — No topic mismatch check. Global file is loaded
without topic validation (per spec: no per-topic scoping). The staleness warning now
includes the loaded topic name so the PM can see at a glance whether the loaded
research matches the current feature. (Resolves Gap 6 and Gap 11)

---

## Dependencies & Risks

**Dependency: Monterey AI MCP server must be pre-configured.** Implementation does not
include auth/credential setup. If unconfigured, customer voice sections fail with
`status: failed` and `error: "Monterey AI MCP tool not available — verify MCP
configuration"` — expected behavior, not a bug. Verify MCP configuration in the
target environment before testing STORY-001.

**Risk: Challenger citation compliance is non-deterministic.** The citation requirement
is enforced via prompting, not code. Manual validation with 3 runs establishes a
baseline. Sustained compliance requires prompt iteration if initial results are poor.
3 runs are sufficient to confirm the behavior is achievable; they are not a statistical
guarantee.

**Risk: `pm_notes` read-modify-write in STORY-005.** This is the plan's highest-risk
operation. Claude re-serializes the entire file from parsed structure, not via string
surgery. Risks: different quoting style on re-serialization, inconsistent multi-line
string encoding, `generated_at` losing UTC notation. Mitigations in the instruction:
explicit re-serialize-from-structure requirement, YAML block scalar rule for multi-line
`pm_notes`, schema doc as the serialization reference. Acceptance: a mandatory manual
test must start with a pre-populated `pm_notes` and a non-trivial `research-context.yaml`,
run the append, and verify all other fields are unchanged.

---

## Success Metrics

**Primary:** ≥1 challenger flag per review session cites a named external source by
name when `research-context.yaml` is present with `status: ok`. Verifiable by
inspecting challenger output for named citations.

**Secondary (Phase 2):** `citation_impact` logging tracks whether research-sourced
flags result in plan updates.

---

## References

### Artifacts
- Brainstorm: `docs/brainstorms/2026-02-24-market-research-phase-brainstorm.md`
- PRD: `docs/prds/2026-02-24-market-research-phase-prd.md`
- Stories: `docs/stories/2026-02-24-market-research-phase-stories.md`

### Key File Reference Points
- Existing command pattern: `plugins/helm/commands/pm/brainstorm.md`
- Partner orchestrator + phase structure: `plugins/helm/commands/pm/partner.md:41`
- Challenger skill + Pattern References section: `plugins/helm/skills/pm-challenger/SKILL.md:122`
- Plugin manifest: `plugins/helm/.claude-plugin/plugin.json`
- CLAUDE.md commands list: `plugins/helm/CLAUDE.md:9`

### External
- Monterey AI MCP server docs: https://docs.monterey.ai/knowledge-hub/mcp-server
- Available MCP tools: `search_snippets`, `generate_document`, `list_workspaces`

### Phase 2 (Deferred)
- `citation_impact` logging schema and write-back mechanism
- Zendesk integration
- Notion integration
