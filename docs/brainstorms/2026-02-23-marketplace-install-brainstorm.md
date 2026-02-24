---
title: Marketplace Install — Brainstorm
type: brainstorm
date: 2026-02-23
topic: marketplace-install
status: approved
---

# Marketplace Install — Brainstorm

## Problem

The current helm install process requires four manual steps:
1. `git clone` to a specific deeply-nested path
2. Edit `~/.claude/plugins/installed_plugins.json` with correct JSON and username substitution
3. Restart Claude Code
4. Copy `product-context.yaml` to the project root

This is error-prone, opaque, and adoption-blocking. The PM building helm has never successfully installed it.

## Target State

Install helm with the same two-command experience as compound-engineering:

```
/plugin marketplace add https://github.com/jeffclark/product-skill-helm.git
/plugin install helm
```

## Root Cause

The helm repo is structured as a standalone plugin, not as a Claude Code marketplace. The Claude Code plugin system requires a `plugins/<name>/` subdirectory in a marketplace repo. compound-engineering works because `https://github.com/EveryInc/compound-engineering-plugin.git` has a `plugins/compound-engineering/` directory — that's the shape Claude Code expects.

helm has no such structure. Its content lives at the repo root with no `plugins/` wrapper, so it can't be registered as a marketplace and installed via `/plugin install`.

## Chosen Approach

**Marketplace restructure** — reshape the helm repo so it is its own marketplace with a single plugin.

Add a `plugins/helm/` directory and move all plugin content into it:

```
plugins/
  helm/
    .claude-plugin/plugin.json
    CLAUDE.md
    commands/pm/
    skills/pm-challenger/
    agents/
README.md        ← stays at root
LICENSE          ← stays at root
product-context.yaml  ← stays at root (template/example)
```

No shell scripts. No JSON editing. No username substitution. The repo root stays clean. The install experience becomes identical to compound-engineering.

## What's Out of Scope

- Interactive `product-context.yaml` first-run setup (pm commands already detect and handle a missing file)
- Any changes to command behavior, skills, or agents

## Challenger Review

Reviewer: Paul Graham

One flag raised (`brainstorm_001`): `product-context.yaml` setup step not addressed.
Override accepted: product-context.yaml setup is out of scope; pm commands already handle missing file gracefully.

## Success Criteria

- `/plugin marketplace add https://github.com/jeffclark/product-skill-helm.git` completes without error
- `/plugin install helm` completes without error and `/pm:*` commands are available in Claude Code
- README install section is two commands, no manual file editing required
