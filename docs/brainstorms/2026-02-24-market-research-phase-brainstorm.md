---
date: 2026-02-24
topic: market-research-phase
---

# Market Research Phase

## What We're Building

A research phase for Helm that gathers external market context — customer
feedback and competitive intelligence — and makes it available to every
subsequent PM artifact and challenger review. Today, PMs either skip research
entirely or do it manually outside Helm, with no established process. The result
is artifacts that reflect internal thinking without external signal: challengers
can only push back on logic, not on market reality.

The research phase surfaces two categories of signal: customer voice (via
Reforge Insights MCP — app store reviews, Gong-sourced feedback, user quotes)
and competitive landscape (via web research — feature sets, positioning,
pricing). These are written to `research-context.yaml` at the project root and
loaded automatically by brainstorm, PRD, GTM, and challenger reviews.

## Why This Approach

Three approaches were evaluated:

**A. `/pm:research` command + `research-context.yaml`** — Dedicated opt-in
command writes findings to a persistent file. All other commands load it
automatically when present.

**B. `--research` flag on existing commands** — Inline research per command,
not persisted across the feature lifecycle.

**C. Research as phase 0 in `/pm:partner`** — Research baked into the full
lifecycle orchestrator.

**Chosen: A + C.** A standalone `/pm:research` command gives PMs explicit
control — they can run it when needed, review and edit the findings before any
artifact is generated, and reuse the context across the full feature lifecycle.
`/pm:partner` additionally runs it automatically as phase 0, so full-lifecycle
runs are covered without a separate invocation. Individual commands remain
non-breaking: they work identically if `research-context.yaml` is absent.

## Key Decisions

- **Reforge via Monterey AI MCP server**: `search_snippets` for semantic search
  over customer feedback; competitive analysis via web research. The MCP server
  is the integration mechanism — no file exports or custom API work required.
- **Opt-in architecture**: Research is a named step, not automatic. This
  prevents every Helm session from incurring research latency when it isn't
  needed.
- **Persistent artifact**: `research-context.yaml` persists at project root,
  scoped to a topic, with a timestamp. Commands treat stale research as a
  warning, not a blocker.
- **Phase 2 sources**: Notion and Zendesk are explicitly deferred. Reforge
  proves the thesis; competitive web research covers the second highest-value
  source. Expansion to internal sources follows if phase 1 succeeds.

## Resolved Challenges

**[flag_brainstorm_001] SCOPE CREEP** — Four sources (Zendesk, Notion, Reforge,
competitive web) bundled into a single deliverable. Resolved by scoping phase 1
to Reforge + competitive web research only. Zendesk and Notion are phase 2,
contingent on phase 1 proving value.

**[flag_brainstorm_002] UNDER-DEFINED REQUIREMENTS** — Reforge access mechanism
and artifact type were unspecified. Resolved: Reforge is accessed via the
Monterey AI MCP server (`search_snippets` tool). Competitive research uses web
search. Both flow into `research-context.yaml` before reaching the challenger.

**[flag_brainstorm_003] METRIC RISK** — "Challenger gets sharper" was
unmeasurable. Resolved: success is defined as at least one flag per challenger
review citing an external data source by name. Binary and observable.

## Open Questions

- What is the staleness threshold for `research-context.yaml`? When should Helm
  prompt the PM to refresh it?
- Should competitive research be structured (named competitors, feature matrix)
  or freeform synthesis? What format is most useful to the challenger?
- Does `/pm:research` run silently in `--auto` mode within `/pm:partner`, or
  does it surface findings for PM review before proceeding?
- Should `research-context.yaml` be topic-scoped (one file per feature) or
  global (one file per project, overwritten on each run)?

## Next Steps
→ Run `/pm:prd` to define requirements
