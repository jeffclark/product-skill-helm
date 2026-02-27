---
title: STORY-006 Manual Validation Results
date: 2026-02-24
story: STORY-006
feature: market-research-phase
status: pending-mcp-environment
---

# STORY-006 Manual Validation Results

**Status: PENDING — Real MCP environment required**

The runs below validated the **prompting instruction structure** (SKILL.md Research
Context Injection section) using a synthetic fixture. They did **not** validate the
end-to-end pipeline. STORY-006 acceptance is not met until real-environment runs are
completed (see "Required real-environment validation" below).

---

**Date:** 2026-02-24
**Validator:** Claude Code (inline simulation against synthetic fixture)
**Fixture:** `docs/fixtures/research-context-sample.yaml` — fictional data authored by Claude Code. Not real Reforge Insights output. Not real web search output.
**Limitation:** The challenger reviews below were simulated in-context against the same fixture data that Claude Code wrote. The Monterey AI MCP `search_snippets` tool was not called. No real `research-context.yaml` was produced by `/pm:research`. This is a circular test — the same model that authored the fixture data also ran the reviews against it.

**What these runs do establish:**
- The Research Context Injection prompting instructions in SKILL.md produce research-sourced flags when fixture data is present in the expected schema format
- The flag format (source name + data point in body + concrete recommendation) is achievable
- Cap substitution logic fires correctly when 5 internal flags are present

**What these runs do NOT establish:**
- That `/pm:research` successfully calls the Monterey AI MCP server
- That the load step in the 5 existing commands reads a real file into execution context
- That the challenger receives real MCP-returned data and generates citations from it
- End-to-end compliance with the acceptance criteria in STORY-006

---

**Pass criteria (for real-environment runs):** All 3 runs must include ≥1 flag naming an external source + citing a specific data point + concrete artifact recommendation.
**Required test case:** Cap substitution must fire in at least 1 run with substitution note documented.

---

## Run 1 — PRD Challenger Review

**Artifact:** PRD for "Streamlined Mobile Registration Checkout"
- FR-001: Reduce checkout to ≤4 steps
- FR-002: Progress indicator
- FR-003: Improved payment form layout
- Out of scope: Payment method changes (Apple Pay, Google Pay)
- Success: 10% increase in mobile completion rate

**Reviewer:** Marty Cagan (PRD)

**Flags raised:**

| Flag ID | Category | Research-sourced |
|---|---|---|
| `flag_prd_001` | UNDER-DEFINED REQUIREMENTS | No — step count and definition of "step" unspecified |
| `flag_prd_002` | METRIC RISK | No — completion rate lacks baseline and measurement method |
| `flag_prd_003` | UNDER-DEFINED REQUIREMENTS | **Yes** |

**Research-sourced flag (`flag_prd_003`):**
> Reforge Insights data (16 snippets, topic: youth sports mobile registration checkout): "payment method gaps" is the second-highest theme with 7 mentions, including explicit requests for Apple Pay and Google Pay. The PRD scopes out payment method changes entirely. Meanwhile, TeamSnap — the primary competitor — already supports Apple Pay and Google Pay at checkout, and SportsEngine supports ACH and installment plans. Reducing step count and improving form layout will not address the reason 7 of 16 users mentioned mobile checkout: they cannot pay the way their phone expects them to.
> Recommendation: Revise the out-of-scope decision for payment methods. Either add an FR explicitly addressing mobile wallet support (Apple Pay, Google Pay), or document a reasoned decision for why payment method parity with TeamSnap is deferred with a target date.

**Verification:**
- [x] Named external source: "Reforge Insights data"
- [x] Specific data point: "second-highest theme with 7 mentions," "7 of 16 users," TeamSnap named with specific feature (Apple Pay, Google Pay)
- [x] Source and data point appear in flag body (not only in recommendation)
- [x] Concrete artifact recommendation: "add an FR explicitly addressing mobile wallet support or document the deferral decision"
- [x] Flag count: 3 (under cap)

**Result: PASS**

---

## Run 2 — Brainstorm Challenger Review

**Artifact:** Brainstorm for "One-tap Re-enrollment for Returning Families"
- Idea: Single-confirmation re-registration for returning families
- Rationale: "We hear this complaint every season" (no named user, no quote)
- No mention of competitor feature sets

**Reviewer:** Paul Graham (Brainstorm)

**Flags raised:**

| Flag ID | Category | Research-sourced |
|---|---|---|
| `flag_brainstorm_001` | WEAK RATIONALE | No — no named user, no quote, pattern inferred not observed |
| `flag_brainstorm_002` | SCOPE CREEP | **Yes** |

**Research-sourced flag (`flag_brainstorm_002`):**
> Reforge Insights data (topic: youth sports mobile registration checkout): "repeat data entry" has 6 mentions, but the two higher-volume complaints are checkout step count (9 mentions) and payment method gaps (7 mentions). If you build one-tap re-enrollment and nothing else, you will have shipped something for 6 of 16 users while leaving the top two pain points untouched. Additionally, TeamSnap already ships one-tap re-enrollment as a baseline feature — you would be closing a parity gap, not creating a differentiator.
> Recommendation: Before committing to one-tap re-enrollment as the brainstorm focus, test whether reducing the step count for first-time registrations (9 mentions) or adding Apple Pay/Google Pay support (7 mentions) would move the needle for more parents. One-tap re-enrollment may be the right second step, not the first.

**Verification:**
- [x] Named external source: "Reforge Insights data"
- [x] Specific data points: "6 mentions," "9 mentions," "7 mentions," "7 of 16 users," "16 users" (denominator visible)
- [x] Named competitor: TeamSnap ("already ships one-tap re-enrollment as a baseline feature")
- [x] Source and data point appear in flag body
- [x] Concrete artifact recommendation: "test whether step count reduction or Apple Pay/Google Pay would move the needle more"
- [x] Flag count: 2 (under cap)

**Result: PASS**

---

## Run 3 — Cap Substitution Test (Weak PRD)

**Artifact:** PRD for "Mobile Registration Overhaul" — deliberately weak with 5 natural internal issues
- Scope: Bundles checkout redesign + push notifications + coach roster access + saved payment methods + parent profiles
- Rationale: "users want a better mobile experience"
- FR-002: Push notifications for registration reminders (no permission flow specified)
- FR-003: Coach roster visibility without separate login (security model unspecified)
- FR-004: Saved payment methods (payment types unspecified)
- Success: "user satisfaction improves"

**Reviewer:** Marty Cagan (PRD)

**Internal flags before citation requirement applied (5):**
1. `flag_prd_001` SCOPE CREEP — 5 unrelated systems bundled (HIGH severity)
2. `flag_prd_002` WEAK RATIONALE — "better mobile experience" no specificity (HIGH severity)
3. `flag_prd_003` UNDER-DEFINED REQUIREMENTS — push notification permission flow missing (LOW severity — narrowest scope, most recoverable)
4. `flag_prd_004` UNDER-DEFINED REQUIREMENTS — coach roster security model unspecified (HIGH severity — security risk)
5. `flag_prd_005` METRIC RISK — "user satisfaction improves" unmeasurable (HIGH severity)

**Cap substitution decision:** Citation requirement fills slot 4. `flag_prd_003` (push notification permission flow) is the lowest-severity internal flag — narrower scope, most recoverable in sprint, does not block delivery the way flags 1, 2, 4, 5 do. It is dropped.

**Substitution note produced:**
> `[Internal flag omitted to accommodate required research citation.]`

**Research-sourced flag inserted as `flag_prd_004`:**
> Reforge Insights data (topic: youth sports mobile registration checkout): "payment method gaps" is the second-highest theme with 7 explicit mentions, including direct requests for Apple Pay and Google Pay. FR-004 says "add saved payment methods" but does not specify which payment methods are in scope. Saved credit card on file is a materially different engineering effort from Apple Pay, which requires no card entry and relies on device biometrics. TeamSnap already supports Apple Pay, Google Pay, and ACH — shipping FR-004 as "saved credit card" will close the data-entry friction but leave the payment method gap against TeamSnap entirely open.
> Recommendation: Revise FR-004 to explicitly enumerate the payment methods in scope for v1. Separate "mobile wallet support (Apple Pay, Google Pay)" from "saved card on file" as distinct requirements with distinct acceptance criteria. If mobile wallet support is deferred, document the decision and target date.

**Final flag output (5 flags, cap maintained):**
1. `flag_prd_001` SCOPE CREEP
2. `flag_prd_002` WEAK RATIONALE
3. `flag_prd_003` UNDER-DEFINED REQUIREMENTS (coach roster security — renumbered after drop)
4. `flag_prd_004` UNDER-DEFINED REQUIREMENTS (research-sourced — payment methods)
5. `flag_prd_005` METRIC RISK

**Verification:**
- [x] Substitution note: `[Internal flag omitted to accommodate required research citation.]` — appears immediately before research-sourced flag
- [x] Dropped flag documented: `flag_prd_003` — push notification permission flow (lowest severity, narrowest scope)
- [x] Named external source: "Reforge Insights data"
- [x] Specific data points: "7 explicit mentions," named TeamSnap with specific payment methods (Apple Pay, Google Pay, ACH)
- [x] Source and data point in flag body
- [x] Concrete artifact recommendation: "Revise FR-004 to enumerate payment methods; separate mobile wallet from saved card"
- [x] Flag count: 5 (cap maintained exactly)
- [x] Citation requirement satisfied without exceeding cap

**Result: PASS**

---

## Summary

| Run | Artifact | Reviewer | Research flag | Citation valid | Result |
|---|---|---|---|---|---|
| 1 | PRD — Checkout Improvements | Cagan | `flag_prd_003` | Yes | **PASS** |
| 2 | Brainstorm — Re-enrollment | Graham | `flag_brainstorm_002` | Yes | **PASS** |
| 3 | PRD (weak) — Overhaul | Cagan | `flag_prd_004` (cap substitution) | Yes | **PASS** |

**All 3 runs: PASS (synthetic fixture only — see status at top)**

**STORY-006 acceptance gate: NOT MET — pending real-environment runs**

### Observations

1. The citation directive reliably surfaces the payment method gap (Reforge 7 mentions + TeamSnap competitive comparison) as the highest-value research flag across all artifact types that touch checkout. This is the correct behavior — the research data is clear enough that the most compelling flag writes itself.

2. In Run 2 (brainstorm), the research flag correctly reframes the scope of the brainstorm rather than just adding a requirement. Graham's voice naturally fits this use — the research data becomes evidence that the brainstorm may be solving the wrong problem first.

3. Cap substitution (Run 3) correctly identifies the narrowest/most recoverable internal flag as the one to drop. The substitution note format is unambiguous. The dropped flag (push notification permission flow) is real and should be addressed in sprint — but it is the right call to defer it for the research citation given the severity ordering.

4. **Prompt compliance observation:** The Research Context Injection section in SKILL.md produces consistent citation behavior across both Paul Graham (brainstorm) and Marty Cagan (PRD) reviewer voices. The source-naming and data-point requirements are specific enough that there is no ambiguity about what satisfies them. Generic references ("customers mentioned...") were not produced in any run.

---

## Required real-environment validation

To close STORY-006, run the following with Monterey AI MCP configured in your environment:

**Prerequisites:**
- Monterey AI MCP server configured in Claude Code (verify with `/mcp` or check `~/.claude/mcp.json`)
- Delete any existing `research-context.yaml` at project root before starting

**Step 1 — Run `/pm:research` on a real topic**
```
/pm:research [your topic]
```
Verify: `research-context.yaml` is written at project root with real snippets in `customer_voice` and real competitors in `competitive_landscape`. Both sections should be `status: ok` for the citation directive to activate.

**Step 2 — Run a PRD challenger review**
Run `/pm:prd` on any feature in your project. At the challenger review step, verify:
- [ ] At least one flag names "Reforge Insights" or a named competitor by name
- [ ] The cited data point is from the actual `research-context.yaml` written in Step 1 (not generic)
- [ ] The flag includes a concrete artifact recommendation

**Step 3 — Run a brainstorm challenger review**
Run `/pm:brainstorm` on any topic. Same verification as Step 2.

**Step 4 — Cap substitution run**
Use a deliberately weak artifact (bundle 5+ unrelated requirements, vague rationale, no metrics). Verify:
- [ ] `[Internal flag omitted to accommodate required research citation.]` appears in the output
- [ ] Total flags = 5 (cap maintained)
- [ ] Research-sourced flag cites a named source and data point

Document results and update this file's `status` frontmatter from `pending-mcp-environment` to `passed`.

---

### Follow-up for Phase 2

When `citation_impact` logging is implemented, the three research-sourced flags from these runs provide realistic test data:
- Run 1, `flag_prd_003`: source = Reforge Insights + TeamSnap, data point = payment method gap (7 mentions)
- Run 2, `flag_brainstorm_002`: source = Reforge Insights + TeamSnap, data point = re-enrollment parity + higher-priority themes
- Run 3, `flag_prd_004`: source = Reforge Insights + TeamSnap, data point = payment method gap (7 mentions, Apple Pay/Google Pay)
