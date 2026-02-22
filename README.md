# helm

Your AI coding agent won't tell you when you're building the wrong thing. Helm will. A Claude Code plugin that challenges your product decisions before they become engineering work.

Pairs with [compound-engineering](https://github.com/EveryInc/compound-engineering-plugin) for end-to-end product-to-engineering handoffs.

## Installation

**1. Clone the repo:**

```bash
git clone https://github.com/jeffclark/product-skill-helm \
  ~/.claude/plugins/cache/local/helm/1.0.0
```

**2. Register the plugin** by adding this entry to `~/.claude/plugins/installed_plugins.json` inside the `"plugins"` object:

```json
"helm@local": [
  {
    "scope": "user",
    "installPath": "/Users/YOUR_USERNAME/.claude/plugins/cache/local/helm/1.0.0",
    "version": "1.0.0"
  }
]
```

Replace `YOUR_USERNAME` with your macOS username (`echo $USER` if unsure). If `installed_plugins.json` doesn't exist yet, create it:

```json
{
  "version": 2,
  "plugins": {
    "helm@local": [
      {
        "scope": "user",
        "installPath": "/Users/YOUR_USERNAME/.claude/plugins/cache/local/helm/1.0.0",
        "version": "1.0.0"
      }
    ]
  }
}
```

**3. Restart Claude Code.** The `/pm:*` commands will be available in any project.

## Setup

Copy `product-context.yaml` to your project root and fill in your context:

```bash
cp ~/.claude/plugins/cache/local/helm/1.0.0/product-context.yaml ./product-context.yaml
```

```yaml
schema_version: 1
company: "Acme Corp"
personas:
  - name: "League Admin"
    description: "Manages team rosters and schedules for recreational leagues"
tech_stack:
  - Ruby on Rails
  - React
principles:
  - Ship small, iterate fast
  - Measure before optimizing
recent_artifacts: []
```

All `/pm:*` commands read from this file automatically and update `recent_artifacts` as artifacts are produced.

## Commands

| Command | What it does |
|---------|-------------|
| `/pm:brainstorm` | Explore a feature idea through structured dialogue. Produces a brainstorm doc. |
| `/pm:prd` | Generate a PRD from a brainstorm doc or feature description. |
| `/pm:stories` | Break a PRD into engineering-ready user stories. |
| `/pm:gtm` | Build a go-to-market plan with rollout sequencing and stakeholder comms. |
| `/pm:analytics` | Define the analytics plan using AARRR (growth) or HEART (UX/quality). |
| `/pm:partner` | Full lifecycle orchestrator. Runs all of the above in sequence with PM approval gates. |

Each command accepts a feature name or a file path:

```
/pm:prd team-roster-import
/pm:prd docs/brainstorms/2026-02-22-team-roster-import-brainstorm.md
/pm:partner --auto team-roster-import
```

## The Challenger Layer

Every command runs a challenger review before generating its artifact. The challenger flags:

- **SCOPE CREEP** — feature is doing more than one job
- **WEAK RATIONALE** — the "why" is missing or circular
- **UNDER-DEFINED REQUIREMENTS** — critical behavior is unspecified
- **METRIC RISK** — success criteria won't tell you if the feature worked

Flags are labeled (`[flag_prd_001]`) with an explanation, consequence, and resolution path. You resolve each one or override it with a reason before the artifact is generated.

```
CHALLENGER REVIEW — PRD

[flag_prd_001] SCOPE CREEP
This feature bundles messaging, comments, and notification preferences into one
deliverable. These are three independent systems. Consequence: any delay on one
blocks the full release.
Resolution path: Scope to messaging only. Create separate definitions for comments
and notification preferences.

---
1 flag. Address it before proceeding, or override with a reason.
To override: "override prd_001 — splitting blocked by Q3 deadline"
```

## Engineering Handoff

After `/pm:partner` completes, it writes a handoff manifest to `docs/handoffs/`. Pass the stories doc to compound-engineering to begin implementation:

```
/workflows:plan docs/stories/2026-02-22-team-roster-import-stories.md
```

## Automation Mode

Run `/pm:partner --auto` to bypass all approval gates. Useful when driving the lifecycle from an upstream orchestrator. Challenger flags are appended to each artifact for post-run review rather than blocking the pipeline.

## Output Structure

```
docs/
  brainstorms/   YYYY-MM-DD-<topic>-brainstorm.md
  prds/          YYYY-MM-DD-<topic>-prd.md
  stories/       YYYY-MM-DD-<topic>-stories.md
  gtm/           YYYY-MM-DD-<topic>-gtm.md
  analytics/     YYYY-MM-DD-<topic>-analytics.md
  handoffs/      YYYY-MM-DD-<topic>-handoff.md
product-context.yaml
```

## License

MIT
