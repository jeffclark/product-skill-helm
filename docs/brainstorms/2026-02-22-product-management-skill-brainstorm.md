---
date: 2026-02-22
topic: product-management-skill
---

# Product Management Skill for Claude Code

## What We're Building

A Claude Code skill that acts as a force multiplier for AI-native product and engineering teams. It combines modular PM deliverable sub-skills (each with a built-in challenger layer) with an optional PM Partner orchestrator for end-to-end lifecycle runs. The skill produces structured PM artifacts — PRDs, user stories, GTM plans, analytics specs — while actively pushing back on feature bloat, under-defined requirements, and bad ideas.

This is not PM automation. It's a senior PM co-pilot that compresses the PM lifecycle through parallel agents and produces outputs designed to feed directly into compound-engineering workflows.

## Why This Approach

**Blend B+C: Modular sub-skills + optional orchestrator**

- **Modular sub-skills** give PMs the flexibility to drop in mid-lifecycle. A PM who already has a PRD can jump straight to `/pm:stories` without going through the full flow.
- **PM Partner orchestrator** (`/pm:partner`) handles full end-to-end runs — spinning up parallel specialist agents and sequencing the lifecycle intelligently.
- **Challenger layer is baked into every sub-skill**, not just the orchestrator. Whether you invoke directly or via the Partner, you always get the pushback.

Other approaches considered:
- Pure pipeline (A): Too rigid, doesn't match how PMs actually work
- Pure hub (C): No escape hatch for mid-lifecycle entry points

## Key Decisions

- **Challenger layer is universal**: Every sub-skill (not just the orchestrator) actively flags feature bloat, scope creep, under-defined requirements, and weak rationale. It questions before it produces.
- **AI-native output formats**: All deliverables are structured for direct consumption by compound-engineering agents — PRDs feed `/workflows:plan`, user stories are formatted for engineering agents, analytics specs include event schemas.
- **Parallel agent architecture**: The PM Partner orchestrator runs specialist agents in parallel (e.g., GTM + analytics can run simultaneously once requirements are locked), compressing multi-day PM work into minutes.
- **Force multiplier framing**: The PM makes every final call. Claude's role is to compress research, draft artifacts, surface risks, and challenge decisions — not replace PM judgment.
- **Core deliverables** (phase 1 scope): PRD, user stories, GTM plan, analytics plan. UX brief and legal/compliance as secondary scope.

## Sub-Skills

| Sub-Skill | Deliverable | Notes |
|---|---|---|
| `/pm:brainstorm` | Feature exploration doc | Pushes back on bad ideas early |
| `/pm:prd` | Product Requirements Document | Gates user story generation |
| `/pm:stories` | User stories (eng-ready) | Structured for compound-engineering |
| `/pm:gtm` | Go-to-market plan | Messaging, rollout, stakeholder comms |
| `/pm:analytics` | Analytics plan + event spec | Metrics, success criteria, tracking |
| `/pm:partner` | Full lifecycle orchestrator | Invokes the above; optional entry point |

## Resolved Decisions

- **Challenger tone**: Always explains the pushback with rationale, then offers a resolution path — e.g., "This looks like scope creep because X. Do you want to cut it or create a separate feature?"
- **compound-engineering handoff**: PM Partner surfaces a structured "ready to hand off" summary and the PM manually triggers `/workflows:plan`. No auto-trigger.
- **Analytics format**: Narrative only — written plan describing metrics and success criteria. No machine-readable event schemas.
- **Context sourcing**: CLAUDE.md as baseline (company, users, tech stack, product principles), skill asks follow-up questions only for what's missing. Critically: **the skill updates CLAUDE.md with new context learned during a session**, compounding knowledge for future runs.

## Next Steps

→ Run `/workflows:plan` when ready to implement the skill structure and individual sub-skills.
