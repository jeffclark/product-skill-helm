# CLAUDE.md — helm plugin

This file provides guidance to Claude Code when working with this repository.

This plugin provides PM tooling for AI-native product and engineering teams.

## Commands

- `/pm:research` — Gather market context: customer voice (Reforge Insights MCP) and
  competitive intelligence. Writes `research-context.yaml` for use by all other commands.
- `/pm:brainstorm` — Explore and validate a feature idea with the challenger layer
- `/pm:prd` — Generate a Product Requirements Document
- `/pm:stories` — Generate user stories from a PRD
- `/pm:gtm` — Generate a go-to-market plan
- `/pm:analytics` — Generate a narrative analytics plan
- `/pm:partner` — Full lifecycle orchestrator (runs all of the above sequentially,
  with research as phase 0)

## Plugin Architecture

```
.claude-plugin/plugin.json        # Plugin manifest
skills/pm-challenger/             # Challenger methodology skill
  SKILL.md                        # Core process knowledge (<200 lines)
  references/                     # Pattern catalogs loaded selectively
commands/pm/                      # User-facing commands
agents/                           # Specialist document-generation agents
```

## Setup

Product context for this plugin accumulates in `product-context.yaml` at your **project root**, not inside this plugin directory.

The first pm command you run will prompt you to create it if it doesn't exist.

Example structure:

```yaml
schema_version: 1
company: ""
personas: []
tech_stack: []
principles: []
recent_artifacts: []
```

Commands read from and append to this file automatically. Append-only — never overwrite existing entries.
