---
title: Marketplace Install — PRD
type: prd
date: 2026-02-23
topic: marketplace-install
status: approved
---

# Marketplace Install

## Problem

Installing helm requires four manual steps: cloning to a specific path, hand-editing a JSON file with a username substitution, restarting Claude Code, and copying a YAML template. The process is error-prone enough that the plugin's own author has never successfully completed it. This will block adoption.

## Goal

Make helm installable with the same two-command experience as compound-engineering:

```
/plugin marketplace add https://github.com/jeffclark/product-skill-helm.git
/plugin install helm
```

After `/plugin install helm` completes, all `/pm:*` commands are immediately available with no additional steps.

## Non-Goals

- Interactive `product-context.yaml` first-run setup
- Changes to any command, skill, or agent behavior
- Version management or automated update mechanism
- Publishing to the `claude-plugins-official` or any third-party marketplace

## How Claude Code Marketplaces Work

Claude Code's plugin system discovers plugins inside a `plugins/<name>/` subdirectory of a marketplace repo. When a user runs `/plugin marketplace add <git-url>`, Claude Code clones the repo and indexes everything under `plugins/`. When they run `/plugin install <name>`, it copies `plugins/<name>/` to the plugin cache and activates it.

compound-engineering works because `https://github.com/EveryInc/compound-engineering-plugin.git` has a `plugins/compound-engineering/` directory. helm needs the same shape.

## Requirements

### 1. Add `plugins/helm/` directory

Create a `plugins/helm/` subdirectory at the repo root. This is the directory Claude Code will install from.

### 2. Move plugin content into `plugins/helm/`

The following must live inside `plugins/helm/`:

| Source (current) | Destination |
|---|---|
| `.claude-plugin/plugin.json` | `plugins/helm/.claude-plugin/plugin.json` |
| `CLAUDE.md` | `plugins/helm/CLAUDE.md` |
| `commands/` | `plugins/helm/commands/` |
| `skills/` | `plugins/helm/skills/` |
| `agents/` | `plugins/helm/agents/` |

### 3. Keep at repo root

These files remain at the repo root and are not part of the installed plugin:

- `README.md`
- `LICENSE`
- `product-context.yaml` (example/template for new users)
- `docs/` (development artifacts — brainstorms, PRDs, etc.)
- `.claude/` (development settings)

### 4. Update README install instructions

Replace the current 4-step manual install section with:

```bash
/plugin marketplace add https://github.com/jeffclark/product-skill-helm.git
/plugin install helm
```

The README should also note that the user needs to restart Claude Code after install (standard behavior for all Claude Code plugins).

## Acceptance Criteria

1. Running `/plugin marketplace add https://github.com/jeffclark/product-skill-helm.git` completes without error.
2. Running `/plugin install helm` completes without error.
3. After restarting Claude Code, all `/pm:*` commands (`/pm:brainstorm`, `/pm:prd`, `/pm:stories`, `/pm:gtm`, `/pm:analytics`, `/pm:partner`) are available in any project.
4. The README install section contains exactly two commands and no manual file editing instructions.
5. No command, skill, or agent behavior is changed.

## Challenger Review Summary

- Cagan (PRD): Clean pass
- Jobs (PRD): Clean pass
- Brainstorm flag `brainstorm_001` (product-context.yaml setup): Overridden — out of scope
