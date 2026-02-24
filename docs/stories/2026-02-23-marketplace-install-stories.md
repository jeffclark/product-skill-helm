---
title: Marketplace Install — User Stories
type: stories
date: 2026-02-23
topic: marketplace-install
status: approved
source-prd: docs/prds/2026-02-23-marketplace-install-prd.md
---

# Marketplace Install — User Stories

## Story Count: 1
## Epic: Install Experience

---

## Story 1 — Two-Command Install

**As a PM who wants to use helm,** I can install it with two commands so that I never have to manually edit JSON files, substitute my username, or clone to a specific path.

```
/plugin marketplace add https://github.com/jeffclark/product-skill-helm.git
/plugin install helm
```

### Context

Today, installing helm requires four manual steps including hand-editing `~/.claude/plugins/installed_plugins.json` with a hardcoded username path. The process is brittle enough that the plugin's own author has never completed it successfully. The fix is to restructure the repo so Claude Code's native marketplace system can handle install — the same mechanism that makes compound-engineering installable in two commands.

### What needs to change

**Repo structure:** Move all plugin content into a `plugins/helm/` subdirectory. Claude Code discovers plugins by looking for `plugins/<name>/` in a registered marketplace repo.

| Move from (current root) | Move to |
|---|---|
| `.claude-plugin/plugin.json` | `plugins/helm/.claude-plugin/plugin.json` |
| `CLAUDE.md` | `plugins/helm/CLAUDE.md` |
| `commands/` | `plugins/helm/commands/` |
| `skills/` | `plugins/helm/skills/` |
| `agents/` | `plugins/helm/agents/` |

**Stay at repo root** (not part of the installed plugin):
- `README.md`
- `LICENSE`
- `product-context.yaml`
- `docs/`
- `.claude/`

**README:** Replace the current 4-step install section with the two commands above. Retain the restart-required note (standard for all Claude Code plugins).

### Acceptance Criteria

- [ ] `plugins/helm/` exists in the repo and contains `.claude-plugin/plugin.json`, `CLAUDE.md`, `commands/`, `skills/`, and `agents/`
- [ ] `/plugin marketplace add https://github.com/jeffclark/product-skill-helm.git` completes without error
- [ ] `/plugin install helm` completes without error
- [ ] After restarting Claude Code, `/pm:brainstorm`, `/pm:prd`, `/pm:stories`, `/pm:gtm`, `/pm:analytics`, and `/pm:partner` are all available in any project
- [ ] README install section contains exactly two commands with no manual file editing instructions
- [ ] No command, skill, or agent behavior is changed by this restructure

### Out of Scope

- Interactive `product-context.yaml` first-run setup
- Any changes to `/pm:*` command behavior
