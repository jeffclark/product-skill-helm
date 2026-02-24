---
title: Market Research Phase (`/pm:research`) — User Stories
type: stories
date: 2026-02-24
topic: market-research-phase
prd: docs/prds/2026-02-24-market-research-phase-prd.md
---

# Market Research Phase (`/pm:research`) — User Stories

## Epic Summary

This story set delivers the full market research layer for Helm: a new `/pm:research` command, the `research-context.yaml` file schema and write/read contract, integration of that file across all existing commands, `/pm:partner` phase 0 orchestration, and challenger prompting rules that require sourced citations when research is present.

**Recommended implementation order:** STORY-002 (schema) → STORY-001 (command) → STORY-003 and STORY-004 in parallel → STORY-005 → STORY-006.

---

## Implementation Order

1. **STORY-002** — Define the schema first. Every other story references field names, status enums, and structural constraints from this document.
2. **STORY-001** — Implement `/pm:research`. Establishes the file write contract and produces a real `research-context.yaml` downstream stories develop against.
3. **STORY-003 + STORY-004** — Parallel. STORY-003 (load + staleness across existing commands) can be developed against the schema spec without waiting for STORY-001 to be complete. STORY-004 (partner phase 0, silent) requires STORY-001.
4. **STORY-005** — Add the interactive phase 0 review prompt. Requires STORY-004.
5. **STORY-006** — Challenger injection and citation requirement. Requires STORY-002 (fixture) and STORY-003 (context load). Can run in parallel with STORY-004 and STORY-005, but manual validation requires a real file from STORY-001.

---

## Stories

---

### [EPIC-2] STORY-002: Define and document the `research-context.yaml` v1 schema

**As an** engineering agent implementing any part of the research feature,
**I want** a single canonical reference for the `research-context.yaml` schema,
**so that** every component that reads or writes this file uses the same field names, status enums, and structural constraints without needing to infer them from command prose.

**Acceptance Criteria:**
- [ ] A schema reference document exists at `docs/research-context-schema.md` that defines the complete v1 schema.
- [ ] The document specifies all top-level fields: `research_context.topic`, `research_context.generated_at`, `research_context.generated_by`, `research_context.customer_voice`, `research_context.competitive_landscape`, `research_context.pm_notes`.
- [ ] For `customer_voice`: `status` enum (`ok`, `failed`, `no_results`), `error` field (present only when `status: failed`), `source`, `query`, `snippet_count`, `themes` (with `label`, `summary`, `mention_count`), and `snippets` (with `text`, `source` enum, `date`, `sentiment` enum).
- [ ] For `competitive_landscape`: `status` enum (`ok`, `failed`, `no_results`), `error` field (present only when `status: failed`), `competitors` (with `name`, `features`, `positioning`, `pricing`, `source_url`).
- [ ] Document explicitly notes that `citation_impact` is deferred to Phase 2 and must not be written in v1.
- [ ] Structural constraints documented: `generated_at` is required, valid ISO 8601 UTC; `competitive_landscape.competitors` must have ≥1 entry when `status: ok`; `customer_voice.snippets` may be empty list when `status: no_results`; `pm_notes` is always present as a string (empty string if no notes).
- [ ] Document includes a complete annotated YAML example showing a fully-populated `ok` state for both sections.
- [ ] A reader can write a conformant `research-context.yaml` without consulting any command file.

**Notes:**
Documentation artifact only — not a runtime file. Governs all read/write logic in STORY-001, STORY-003, STORY-004, STORY-005, and STORY-006.

**Dependencies:** None

---

### [EPIC-1] STORY-001: Implement the `/pm:research` command

**As a** PM who wants to ground Helm artifacts in external market signal,
**I want to** run `/pm:research [topic]` and receive a filled `research-context.yaml` plus a human-readable summary,
**so that** my challengers and artifact agents have access to real customer voice data and named competitive intelligence.

**Acceptance Criteria:**
- [ ] Command registered at `commands/pm/research.md`, invocable as `/pm:research [topic]`.
- [ ] If `[topic]` is not provided, the command asks: "What topic should I research?" via AskUserQuestion and waits before proceeding.
- [ ] Command calls the Monterey AI MCP server `search_snippets` tool with the topic string as query, with a 30-second timeout.
- [ ] If MCP call succeeds: `customer_voice.status` is `ok`, `snippets` contains returned data.
- [ ] If MCP call fails for any reason (timeout, connection refused, tool error): `customer_voice.status` is `failed`, `customer_voice.error` is populated, execution continues to competitive research without surfacing a hard error. If result is empty: `status: no_results`.
- [ ] Command runs a web search for competitive intelligence using the topic string, querying for feature sets, positioning, and pricing of relevant competitors.
- [ ] If web search returns results for ≥2 named competitors: `competitive_landscape.status` is `ok`, each entry includes `name`, `features`, `positioning`, `pricing` (null if unavailable), `source_url` (null if unavailable).
- [ ] If web search returns no usable results: `competitive_landscape.status` is `no_results`, `competitors` is empty list.
- [ ] Before writing, command reads any existing `research-context.yaml`. If `pm_notes` is non-empty, that value is carried forward. If file absent or `pm_notes` empty, `pm_notes` is written as empty string.
- [ ] Command overwrites `research-context.yaml` at project root. No backup created.
- [ ] `generated_at` is valid ISO 8601 UTC. `generated_by` is `"pm:research"`. `topic` is the topic string.
- [ ] If serialization produces invalid YAML, write is aborted and error printed. No partial file written.
- [ ] On completion, command prints human-readable summary: (1) customer voice themes + up to 5 attributed quotes with source label and date, or failure note; (2) each competitor as 2–3 sentence summary, or failure note.

**Notes:**
Timeout and connection failure for MCP are treated as the same error path (`status: failed`). Uses Monterey AI MCP `search_snippets` (pre-configured — no auth flow). Competitive research uses existing web search capability. `citation_impact` field must NOT be written (Phase 2).

**Dependencies:** STORY-002

---

### [EPIC-3] STORY-003: Add research context load and staleness check to all existing commands

**As a** PM running any Helm command after having run `/pm:research`,
**I want** the command to automatically load and use my research context,
**so that** the context I gathered is available to artifact generation and challenger review without any additional action.

**Acceptance Criteria:**
- [ ] Each of the following commands loads `research-context.yaml` at startup, before any other execution step: `/pm:brainstorm`, `/pm:prd`, `/pm:stories`, `/pm:gtm`, `/pm:analytics`.
- [ ] If `research-context.yaml` is absent, the command proceeds with no error, no warning, no mention of missing context.
- [ ] If the file exists but cannot be parsed as valid YAML, the command prints exactly: `Warning: research-context.yaml could not be parsed — running without research context.` and continues as if absent.
- [ ] If the file exists and parses successfully, its contents are held in the command's execution context and are available to both artifact generation and challenger review steps.
- [ ] If the file exists, parses successfully, and `generated_at` is >7 days before current UTC datetime, the command prints exactly: `Warning: research context is N days old (generated YYYY-MM-DD). Consider re-running /pm:research to refresh findings.` (N = whole days elapsed). Then proceeds immediately.
- [ ] If `generated_at` cannot be parsed, staleness warning is printed with `generated date unknown` in place of the date.
- [ ] Staleness warnings do not block execution. No PM confirmation required.
- [ ] Research context load is additive — does not replace or interfere with `product-context.yaml` loading.

**Notes:**
Affects `commands/pm/brainstorm.md`, `commands/pm/prd.md`, `commands/pm/stories.md`, `commands/pm/gtm.md`, `commands/pm/analytics.md`. Each file receives an identical new load step at the top of the Execution section, before step 1. Staleness threshold is exactly 7 days.

**Dependencies:** STORY-002. STORY-001 not required to write the load logic — only required for end-to-end testing.

---

### [EPIC-4] STORY-004: Add research phase 0 to `/pm:partner` — silent execution and failure handling

**As a** PM running `/pm:partner [topic]`,
**I want** research to run automatically before the brainstorm phase,
**so that** challengers and artifact agents have external market signal without requiring a separate `/pm:research` call.

**Acceptance Criteria:**
- [ ] When `/pm:partner [topic]` is invoked with a feature name (not a file path), phase 0 executes before Phase 1 (Brainstorm). The research query is the topic string.
- [ ] Phase 0 executes all research logic from FR-001 through FR-004: MCP call, web search, `pm_notes` carry-forward, overwrite of `research-context.yaml`.
- [ ] Phase 0 header displayed before research begins (interactive mode only): `Phase 0: Research — [topic]`.
- [ ] If both sections are `failed` or `no_results`: command prints exactly `Warning: research returned no results for "[topic]". Continuing without research context.` then proceeds to Phase 1.
- [ ] If at least one section is `status: ok`: execution continues to Phase 1 without warning. Research summary is NOT printed here (handled by STORY-005 in interactive mode).
- [ ] Phase 0 failures do not block continuation. Command proceeds to Phase 1 in all failure cases.
- [ ] In `--auto` mode: phase 0 runs silently. No phase header, no summary. Only the both-sections-failed warning is printed if applicable. Then proceeds directly to Phase 1 with auto-accept behavior.
- [ ] When `/pm:partner` is invoked with a file path argument (mid-lifecycle entry), phase 0 does not run.

**Notes:**
Affects `commands/pm/partner.md`. Phase 0 inserted before Phase 1. Phase 0 reuses research logic from `/pm:research` at the prompting level (not shared code). `--auto` branch suppresses all output except the dual-failure warning.

**Dependencies:** STORY-001, STORY-002

---

### [EPIC-4] STORY-005: Add interactive phase 0 research review to `/pm:partner`

**As a** PM running `/pm:partner [topic]` in interactive mode,
**I want** to see the research summary after phase 0 and have the option to add notes before continuing,
**so that** I can flag context or caveats that should influence how my challengers and agents use the research findings.

**Acceptance Criteria:**
- [ ] After phase 0 completes in interactive mode (no `--auto`), the command displays the human-readable research summary: customer voice themes + up to 5 attributed quotes (or failure note), then competitor summaries (or failure note). Format is identical to `/pm:research` standalone output.
- [ ] After the summary, command invokes AskUserQuestion: `"Research complete. Add notes before continuing, or proceed?"` with options: (1) Add notes, (2) Proceed.
- [ ] If PM selects "Add notes": a second AskUserQuestion with open text field. PM's response is appended to `pm_notes` in `research-context.yaml`. If `pm_notes` was empty, it is set to the text. If it had existing content, new text is appended with a newline separator.
- [ ] After notes are submitted (or if PM selects "Proceed"), command continues to Phase 1 without re-displaying the summary.
- [ ] If both research sections are `failed`/`no_results`: AskUserQuestion prompt is not shown. Command prints the dual-failure warning (from STORY-004) and proceeds to Phase 1 directly.
- [ ] In `--auto` mode: this entire interactive step is skipped. No AskUserQuestion invoked, no summary printed.

**Notes:**
Affects `commands/pm/partner.md`. Extends phase 0 block added in STORY-004. `pm_notes` append is a read-modify-write on `research-context.yaml` — only `pm_notes` field changes, not a full overwrite.

**Dependencies:** STORY-004, STORY-001

---

### [EPIC-5] STORY-006: Inject research context into challenger prompts and require sourced citations

**As a** PM receiving a challenger review when research context is present,
**I want** the challenger to cite named external sources with specific data points,
**so that** at least one flag per review is grounded in real market signal and gives me a concrete artifact recommendation.

**Acceptance Criteria:**
- [ ] When `research-context.yaml` is loaded and ≥1 section has `status: ok`, the challenger prompt receives the full parsed contents of `research-context.yaml` as injected context before generating questions or flags.
- [ ] When research context is present with ≥1 section at `status: ok`, the challenger instruction set includes an explicit directive: the review must include at least one flag citing a named external source and a specific data point. Acceptable sources: a Reforge Insights snippet with source attribution, a named competitor from `competitive_landscape`, a specific mention count or quote from `customer_voice`. Generic references ("customer research suggests...") do not satisfy the requirement.
- [ ] Each research-sourced flag includes two components: (1) the cited gap or contradiction between the research finding and the artifact, and (2) a concrete artifact recommendation (e.g., "Add a requirement addressing X to FR-003," "Revise positioning to differentiate from Competitor Y's Z feature").
- [ ] Research-sourced flag body explicitly labels the cited source name and data point (not only in the recommendation). Example format: `"Reforge Insights snippet (App Store, 3 weeks ago): 'onboarding takes too long' — 12 mentions. Current PRD has no onboarding time requirement. Recommendation: Add a success criterion to FR-002 specifying maximum time-to-first-value."`
- [ ] If all research sections are `failed` or `no_results`: no research context is injected, no citation directive is added. Challenger runs as if no `research-context.yaml` exists. Challenger does not reference or note the absence.
- [ ] The citation requirement does not increase the maximum flag count. Existing cap of 5 flags per reviewer per review remains.
- [ ] **Manual validation:** Using the sample fixture at `docs/fixtures/research-context-sample.yaml`, run three challenger reviews across different artifact types (at least one PRD review and one brainstorm review). In all three runs, at least one flag cites a named source and includes a concrete artifact recommendation. Results documented in acceptance notes.

**Notes:**
This is a prompting story — the deliverable is modifications to `skills/pm-challenger/SKILL.md`. A sample fixture must be created at `docs/fixtures/research-context-sample.yaml` for manual validation (both sections at `status: ok`, realistic-looking data). Manual validation results (3 test runs) are the acceptance gate, not automated tests. `citation_impact` logging is NOT part of this story (Phase 2).

**Dependencies:** STORY-002 (fixture must be schema-valid), STORY-003 (context must be loaded into command context before it can be injected into challenger)
