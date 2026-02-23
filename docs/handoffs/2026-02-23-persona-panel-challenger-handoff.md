---
title: Named Persona Panel for PM Challenger — Handoff
type: handoff
date: 2026-02-23
topic: persona-panel-challenger
status: ready-for-engineering
---

# Named Persona Panel for PM Challenger — PM Handoff

## Artifacts

| Type | Path |
|------|------|
| Brainstorm | docs/brainstorms/2026-02-23-persona-panel-challenger-brainstorm.md |
| PRD        | docs/prds/2026-02-23-persona-panel-challenger-prd.md |
| Stories    | docs/stories/2026-02-23-persona-panel-challenger-stories.md |
| GTM plan   | docs/gtm/2026-02-23-persona-panel-challenger-gtm.md |

## Challenger Review Summary

Flags raised across all phases: 5
Flags overridden: 1

**Overridden flags:**
- [flag_prd_001] METRIC RISK — Override: qualitative engagement is the bar; no instrumentation required for a pre-launch plugin with a single-digit user base. Noted.

## Key Decisions for Engineering

- **Architecture:** 4 new persona files in `skills/pm-challenger/references/` + updated `SKILL.md`. No other files touched.
- **Questions:** Dynamic (LLM-generated from PM input in persona's voice), not a fixed bank.
- **PRD dual-review:** Shared Q&A, parallel review sections, namespaced flag IDs (`cagan_prd_NNN`, `jobs_prd_NNN`).
- **Cagan:** One file serves both PRD and Stories phases; phase context passed at invocation.
- **Jobs scope:** Hard-constrained to complexity masquerading as sophistication, bundled scope, and fuzzy language only. Must be explicit enough that the model stays in lane.
- **Skip flow:** In-character response (3–5 sentences, first person, direct address) before issuing review on available context.
- **All 9 stories are parallelizable.** STORY-002 (persona file template) is a soft dependency — recommended first to avoid structural rework on persona files.

## Engineering Entry Point

`/workflows:plan docs/stories/2026-02-23-persona-panel-challenger-stories.md`
