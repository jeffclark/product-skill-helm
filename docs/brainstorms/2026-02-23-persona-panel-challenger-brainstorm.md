---
title: Named Persona Panel for PM Challenger — Brainstorm
type: brainstorm
date: 2026-02-23
topic: persona-panel-challenger
status: complete
---

# Named Persona Panel for PM Challenger

## Problem

The current pm-challenger layer flags real issues but has no identity. Users don't know the frame they're being reviewed through. "SCOPE CREEP" from an anonymous voice carries less weight than the same challenge from Paul Graham or Marty Cagan. The challenger lacks gravitas.

Inspiration: Compound Engineering's code review skill, which assigns named engineers (e.g., DHH) to reviews so users immediately understand the philosophy and standards being applied.

## Solution

Upgrade the pm-challenger by assigning a named product leader persona to each lifecycle phase. Each persona brings a distinct philosophy and communication style. Before issuing a formal review, the persona asks targeted questions to surface gaps and challenge assumptions. The conversation is the intake; the review is the output.

## Panel Assignment

| Phase | Reviewer | Primary Lens |
|---|---|---|
| Brainstorm | **Paul Graham** | Is this worth building? Are you talking to users? Are you moving fast enough? |
| PRD | **Marty Cagan** | Are requirements clear? What's the riskiest assumption? Has discovery happened? |
| PRD (co-reviewer) | **Steve Jobs** | Is this simple? What would you cut? Are you saying ten things when you should say one? |
| Stories | **Marty Cagan** | Are engineers given problems to solve or features to build? Are acceptance criteria measurable? |
| GTM | **Paul Graham** | What's the scrappy, unscalable way to get first users? Is distribution part of the product? |
| Analytics | **Jeff Bezos** | What's the data? What's the smallest experiment? What's the mechanism that ensures this happens? |

### Jobs guardrail
Jobs' role at PRD is constrained to clarity and simplicity only. He asks what to cut, not whether the bar is high enough. He does not get to block on quality or perfectionism. If the PRD is simple and clear, he passes it.

## Interaction Model

**Changed from current behavior.** The current challenger issues a one-shot review immediately. The upgraded model:

1. Reviewer persona asks targeted questions (one at a time) to fill gaps and challenge assumptions
2. After Q&A, reviewer issues the formal review in their persona's voice
3. PM addresses flags or overrides with reasoning

The Q&A is genuine — not pro forma. The persona is trying to understand the context before judging it.

## Architecture

**Approach chosen: Persona reference files**

- `SKILL.md` stays lean: Q&A protocol, phase-to-persona mapping, review format, override rules
- Each persona lives in `skills/pm-challenger/references/`:
  - `persona-graham.md` — philosophy, signature questions, flag types, example language
  - `persona-cagan.md`
  - `persona-jobs.md` (with explicit scope constraints)
  - `persona-bezos.md`

Adding or swapping a persona = adding or editing one file. No changes to SKILL.md required.

This mirrors the existing `references/` pattern already established in the plugin (scope-creep-patterns.md, metrics-anti-patterns.md, etc.).

## Challenger Flags Raised

**[flag_brainstorm_001] UNDER-DEFINED REQUIREMENTS**
Resolved: Persona panel fully defined through brainstorm dialogue. Four personas assigned, all phases covered.

**[flag_brainstorm_002] SCOPE CREEP**
Resolved: One-shot review vs. conversational model clarified. The persona asks questions *before* issuing the review. This is a defined interaction model, not two separate features.

## Personas Considered and Rejected

| Persona | Reason not included |
|---|---|
| Shreyas Doshi | Strong on nuance and trade-offs; lower name recognition / gravitas |
| Reid Hoffman | Good for GTM / blitzscaling; Graham covers that ground more cleanly |
| Jason Fried | Aligned with user philosophy; too adjacent to DHH (already in code reviews) |

## Key Decisions

- Single persona per phase (except PRD which has two with non-overlapping scopes)
- Jobs is constrained — simplicity/clarity lens only, no perfectionism blocking
- The reviewer speaks in the persona's voice throughout — questions, flags, and all
- Flag format from current SKILL.md is preserved; tone and content change per persona
- Override protocol unchanged
