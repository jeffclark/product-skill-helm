---
title: Market Research Phase (`/pm:research`)
type: prd
date: 2026-02-24
topic: market-research-phase
status: draft
---

# PRD: Market Research Phase (`/pm:research`)

## Problem

Helm artifacts — brainstorms, PRDs, GTM plans — are generated from internal assumptions with no external market signal. PMs either skip research entirely or conduct it manually outside the tool, producing findings that never integrate into the artifact pipeline. Challengers can interrogate logic but cannot surface contradictions with market reality: they have no access to customer voice data or competitive positioning. The result is artifacts that are internally coherent but externally uninformed.

## Users & Context

**Primary user:** PMs using Helm on AI-native product teams. They run the full Helm lifecycle (`/pm:partner`) or individual commands. Research habits vary widely — some run thorough external research before starting; many do not.

**Usage pattern:** Research is run once per topic before artifact generation begins, either explicitly via `/pm:research` or automatically as phase 0 of `/pm:partner`. The PM may re-run research on the same topic if they suspect findings are stale.

**Frequency:** Per feature or initiative, not per session. A PM working on a topic for a week may run research once and rely on it across multiple artifact iterations.

## Goals

- Enable challengers to cite named external sources (Reforge, named competitors) in at least one flag per review session.
- Surface customer voice and competitive intelligence automatically within the Helm workflow, requiring no manual export or copy-paste by the PM.
- Make research context non-breaking: all existing commands continue to function identically when `research-context.yaml` is absent.
- Integrate research output into `/pm:partner` phase 0 without extending the interactive workflow beyond one additional review step.

## Non-Goals

- Zendesk ticket ingestion (Phase 2).
- Notion document parsing (Phase 2).
- Incremental or diff-based research updates — each run overwrites the global file.
- Per-topic or per-project scoping of research files — one global file at project root.
- Sentiment scoring, trend analysis, or quantitative synthesis beyond what `search_snippets` returns.
- PM authentication or credential management for the Monterey AI MCP server — assumed to be configured in the environment.

## Requirements

### `/pm:research` command

**FR-001:** The system shall expose a `/pm:research [topic]` command where `[topic]` is a required free-text argument describing the feature or problem area to research.

- Actor: PM
- Trigger: PM runs `/pm:research [topic]` in Claude Code
- Expected outcome: The command executes customer voice research and competitive research, then writes findings to `research-context.yaml` at the project root and prints a human-readable summary to the terminal.
- Error state: If the Monterey AI MCP server is unreachable or returns an error, the command logs the failure reason, skips the customer voice section, and continues to competitive research. The written file notes the customer voice section as `status: failed` with the error message. The command does not exit with a hard failure.

**FR-002:** The system shall call the Monterey AI MCP server `search_snippets` tool with the topic string as the semantic search query.

- Actor: System (Helm plugin)
- Trigger: `/pm:research` execution, after topic is parsed
- Expected outcome: The system retrieves feedback snippets (app store reviews, Gong-sourced quotes, user feedback) and includes them in the research file under `customer_voice`.
- Error state: See FR-001 error state. Partial results are written as-is without padding or interpolation.

**FR-003:** The system shall conduct competitive landscape research via web search, querying for feature sets, positioning, and pricing of relevant competitors identified from the topic string.

- Actor: System (Helm plugin)
- Trigger: `/pm:research` execution, after MCP call completes or times out
- Expected outcome: The system identifies at least two named competitors and writes their feature summary, positioning, and pricing (if publicly available) to `research-context.yaml` under `competitive_landscape`.
- Error state: If web search returns no usable competitive results, `competitive_landscape` is written with `status: no_results` and the command continues. The PM summary notes that competitive data was unavailable.

**FR-004:** The system shall overwrite `research-context.yaml` at the project root on each `/pm:research` run, regardless of whether a prior file exists. No backup is created.

**FR-005:** The system shall print a human-readable research summary to the terminal on completion:
1. Customer voice section — key themes and up to five representative quotes with source attribution (e.g., "App Store, 4 weeks ago" or "Gong call transcript")
2. Competitive landscape section — each competitor with a two-to-three sentence summary

If a section failed, the summary notes the failure inline rather than omitting the section silently.

---

### `research-context.yaml` schema

```yaml
research_context:
  topic: "[free-text topic string]"
  generated_at: "[ISO 8601 datetime, UTC]"
  generated_by: "pm:research"

  customer_voice:
    status: "ok | failed | no_results"
    error: "[error message if status: failed; omit otherwise]"
    source: "Monterey AI / Reforge Insights"
    query: "[search query sent to search_snippets]"
    snippet_count: 0
    themes:
      - label: "[theme name]"
        summary: "[one sentence]"
        mention_count: 0    # omit if unavailable
    snippets:
      - text: "[verbatim quote]"
        source: "App Store | Gong | Survey | unknown"
        date: "[ISO 8601 date or null]"
        sentiment: "positive | negative | neutral | unknown"

  competitive_landscape:
    status: "ok | failed | no_results"
    error: "[error message if status: failed; omit otherwise]"
    competitors:
      - name: "[competitor name]"
        features: "[2–4 sentence summary of relevant feature set]"
        positioning: "[1–2 sentence positioning description]"
        pricing: "[pricing summary or null]"
        source_url: "[URL of primary source or null]"

  # Appended optionally during /pm:partner interactive phase 0 review:
  pm_notes: ""

  # Appended during challenger review sessions (append-only within session):
  citation_impact:
    - flag_id: "[challenger flag identifier]"
      source_cited: "[Reforge Insights | competitor name]"
      data_point: "[quote or theme cited]"
      pm_action: "accepted | overridden | deferred"
      artifact: "prd | brainstorm | gtm | stories | analytics"
      timestamp: "[ISO 8601 datetime]"
```

**Schema constraints:**
- `generated_at` must be present and valid on every write. Commands use this field for staleness calculation.
- `customer_voice.snippets` may be empty list if `status: no_results`.
- `competitive_landscape.competitors` must contain ≥1 entry if `status: ok`.
- File must be valid YAML. If serialization produces invalid YAML, write is aborted and error is surfaced to terminal.

---

### Integration with existing commands

**FR-006:** Each Helm command (`/pm:brainstorm`, `/pm:prd`, `/pm:stories`, `/pm:gtm`, `/pm:analytics`) shall load `research-context.yaml` at startup if the file exists at the project root.

- Expected outcome: Contents are parsed and held in the command's execution context, available to both artifact generation and challenger review.
- Error state: If file exists but is invalid YAML or fails schema validation, log warning ("research-context.yaml could not be parsed — running without research context") and continue.

**FR-007:** All commands shall operate identically when `research-context.yaml` is absent. No error, no warning, no mention of missing context.

---

### Staleness handling

**FR-008:** Each command that loads `research-context.yaml` shall compare `generated_at` to the current datetime. If older than 7 days, print one warning line before proceeding:

```
Warning: research context is N days old (generated YYYY-MM-DD). Consider re-running /pm:research to refresh findings.
```

If `generated_at` cannot be parsed, treat as stale and emit warning with "generated date unknown."

**FR-009:** Staleness warnings do not block execution. No PM confirmation required.

---

### `/pm:partner` phase 0

**FR-010:** When `/pm:partner [topic]` is invoked, the system shall execute research as phase 0 before the brainstorm phase, using the provided topic as the research query. Writes `research-context.yaml`. Research failures do not block continuation.

**FR-011 (interactive mode):** After phase 0 completes, display the human-readable research summary and prompt:

```
Research complete. Press enter to continue to brainstorm, or type notes to carry forward.
```

If the PM types notes, append to `research-context.yaml` under `pm_notes` before proceeding. Ctrl-C exits cleanly without beginning brainstorm.

**FR-012 (`--auto` mode):** Research runs silently. No summary printed, no prompt. If both sections fail, print one warning line, then continue.

---

### Challenger behavior with research context

**FR-013:** When `customer_voice.status: ok` or `competitive_landscape.status: ok`, at least one challenger flag per review session must cite a named external source with a specific data point — quote, mention count, or named competitor feature. Generic references ("customer research suggests...") do not satisfy this requirement.

**FR-014:** Every research-sourced challenger flag must include two components:
1. The cited gap or contradiction with the artifact
2. A concrete recommendation for how to update the artifact (e.g., "Add a requirement addressing X to FR-003," "Revise positioning to differentiate from Competitor Y's Z feature")

**FR-015:** If all research sections are `failed` or `no_results`, challengers run as if no research context is loaded. They do not note or reference the absence of research context.

---

## Success Metrics

**Primary:** At least one challenger flag per review session cites an external data source by name when `research-context.yaml` is present with `status: ok` in at least one section. Verifiable by inspecting challenger output for named citations.

**Secondary:** When a PM accepts, overrides, or defers a research-sourced challenger flag, Helm appends to `citation_impact` in `research-context.yaml` — recording the flag ID, source cited, data point, PM action, and artifact. This log tracks whether research signal is moving the work.

---

## Out of Scope (Phase 2)

- Zendesk ticket ingestion
- Notion document parsing
- Per-topic research file scoping
- Research diffing or change detection across runs
- Scheduled or background research refresh
- Additional MCP tools beyond `search_snippets`

---

## Open Questions

1. **MCP timeout behavior:** Should timeout be a distinct error state from connection failure? Owner: engineering, resolve before implementation.
2. **`pm_notes` on re-run:** Currently specced as overwrite (field lost when research re-runs). Confirm this is acceptable.
3. **`citation_impact` across sessions:** Cumulative append or reset per session? Not resolved.

---

## Agent Consumption Notes

- **New file:** `research-context.yaml` at project root. Written by `/pm:research` and `/pm:partner` phase 0. Read by all existing commands at startup.
- **New command:** `/pm:research [topic]` — standalone, no subcommands.
- **Modified commands:** `/pm:partner` gains phase 0 (interactive + `--auto` branches). All existing commands gain file-load + staleness check at startup.
- **New MCP calls:** Monterey AI MCP server, `search_snippets` tool. Pre-configured in environment — no auth flow to implement.
- **Web search:** Competitive research uses existing web search capability. No new integration if web search is available in plugin environment.
- **Terminal output:** Research summary printout, staleness warning, interactive phase 0 prompt in partner. No GUI changes.
- **Failure modes:** MCP unreachable, MCP partial results, web search no results, YAML write failure, YAML parse failure on load, `generated_at` parse failure for staleness check.
