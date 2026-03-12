---
title: PRD Template Support — GTM Plan
type: gtm
date: 2026-03-04
topic: prd-template-support
prd: docs/prds/2026-03-04-prd-template-support-prd.md
---

# PRD Template Support — GTM Plan

## Feature Summary

PMs can place a `prd-template.md` file in their project root. When `/pm:prd` runs and detects the file, the prd-writer agent generates content for every section of the company's template, asks the PM to fill gaps it cannot resolve, and produces a PRD that conforms to the organizational structure — no reformatting required.

---

## Target Audience

**Segment 1 — Existing Product Skill Helm users blocked by template mismatch**

PMs already running `/pm:prd` who then spend 10–20 minutes reformatting output before sharing internally. They have adopted the tool for content quality but are absorbing a friction cost on every PRD. This feature removes that cost entirely. They need to know this exists so they can set it up on their next run — not at a future migration point.

**Segment 2 — Orgs evaluating PM AI tools where template compliance is a blocker**

Product leads or heads of product at companies with enforced PRD standards who have looked at ChatPRD, Craft.io, or Notion AI and found that template conformance alone is table stakes — they need content quality and review, not just a structure skeleton. This is the audience that dismissed category tools for producing well-formatted filler. The differentiator here is template conformance + challenger review in a single run. No competitor delivers both.

**First users (defined):** PMs in the author's personal network at companies with documented PRD standards — known individuals, approachable via direct message, likely already aware of Helm.

---

## Positioning

**Against ChatPRD and Craft.io:** Both offer template support. Neither has a challenger review layer. Helm is the only tool that fills your template *and* pushes back on whether the content inside each section is actually sound.

**Against Notion AI:** Relies on community templates with no structured content generation per section. Helm generates content for every section and surfaces gaps explicitly — the PM does not have to audit the output for blanks.

**The differentiator to protect:** Template conformance + challenger review is the combination. Either alone is not the pitch. Messaging that leads with "template support" without the challenger is a weaker story and undersells the release.

---

## Messaging

**Core message:** Your company's PRD template, filled in by AI — and challenged before you share it.

**Supporting points:**

- Drop a `prd-template.md` in your project root. Every heading your company requires shows up in the output, in order, with content or an explicit gap marker. Zero reformatting.
- When the agent cannot fill a section — because context is missing, not because it guessed — it asks. You answer or skip. No silent blanks, no low-confidence filler passed off as content.
- The challenger review layer still runs. A Cagan-style review that questions whether a template section is under-specified is more useful than a tool that faithfully fills in the boxes. You get conformance and scrutiny in a single pass.
- No schema, no special syntax, no admin setup. Plain markdown committed to your project root. If your template already exists as a doc, it takes two minutes to adapt.

**What not to say:**

- Do not say "template-aware AI" — it describes the mechanism and is easy to dismiss as a minor feature.
- Do not lead with the file setup steps. The setup is trivial; the outcome (shareable PRD without reformatting) is the message.
- Do not claim the agent will match company-specific terminology perfectly. Vocabulary harmonization is explicitly deferred to v2. Overpromising on terminology matching will generate immediate friction with enterprise evaluators.
- Do not position this as "just like ChatPRD but with review." That framing concedes the primary feature to a competitor. Lead with what no competitor has: the combination.

---

## Rollout Sequence

This launch is a plugin version bump and changelog entry. Staged rollout gates are set by signal, not by timeline. Both challenger flags for a formal staged rollout (gtm_001 and gtm_002) are overridden — changelog-only launch is intentional for this release.

| Phase | Cohort | Criteria to expand | Target date |
|-------|--------|--------------------|-------------|
| 1 — Internal validation | Author only | Prototype spike findings document complete; agent tested against 2–3 real template structures; no silent filler behavior observed | Before version bump ships |
| 2 — First users | 3–5 PMs from author's network at companies with defined PRD standards | At least 3 of 5 complete a full `/pm:prd` run using their company template with no reformatting reported; no P1 behavior issues (silent filler, wrong section order, crash on detection) open for more than 24 hours | Within 1 week of version bump |
| 3 — General availability | All Plugin users via changelog | Phase 2 success criteria met; no open P1 issues; changelog entry and setup instructions published | After Phase 2 gate |

**Pause/rollback trigger:** If any user reports the agent generating content under the wrong template heading — placing content in a section other than the one it was generated for — the version is pulled from changelog promotion and Phase 2 outreach stops. One confirmed report of incorrect section placement is sufficient to pause. The trigger is observable in the output file without any tooling.

---

## Stakeholder Communication

| Team | What they need | When | Owner |
|------|----------------|------|-------|
| Phase 2 first users | Direct message with: what changed, a one-paragraph setup instruction (drop `prd-template.md` in project root), and a specific ask — "run it on your next real PRD and tell me if the output is shareable as-is" | Day of version bump | Author |
| PM community (Lenny's Slack, AI PM forums) | A worked example post showing a before/after: reformatting friction before, zero-reformat output after. Include a real template structure and the output it produced. Link to the changelog. | After Phase 2 gate is met | Author |
| Plugin ecosystem / Claude Code directory | Changelog entry with feature description. One-sentence setup instruction in the plugin README. | With version bump | Author |

No sales team. No customer success team. No support queue. This is a solo-distributed plugin. The "support" channel is direct message. Escalation path for any reported issue is: author receives report, confirms behavior, patches or responds within 24 hours.

---

## Launch Checklist

- [ ] Prototype spike complete: agent tested against 2–3 real template structures, findings document written, silent filler conditions identified
- [ ] FR-001 detection message verified: exact string `Template found: prd-template.md — generating PRD using your template structure.` appears on detection
- [ ] FR-002 section order verified: output headings match template heading sequence exactly on a 10-section template
- [ ] FR-003 gap dialogue verified: agent surfaces unfillable sections before writing the file, not after
- [ ] FR-004 regression verified: project without `prd-template.md` produces identical output to current behavior
- [ ] Version bump committed and changelog entry written
- [ ] README or SKILL.md updated with one-paragraph setup instruction for `prd-template.md`
- [ ] Direct outreach messages drafted for Phase 2 first users (3–5 individuals, written per person)
- [ ] Rollback procedure documented: which version to revert to, how to notify Phase 2 users if pulled

---

## Post-Launch Monitoring

**Watch for in the first 48 hours:**

- Section placement errors: any report of content appearing under the wrong heading. Threshold: one confirmed report triggers pause. Check output files directly — no tooling required.
- Silent filler: any report of the agent generating plausible-sounding but context-free content for a section rather than surfacing a gap question. Threshold: one confirmed report with a reproducible template structure triggers a patch before Phase 2 expands.
- Detection failures: any report of the template file not being detected (PM placed `prd-template.md` in root, no detection message appeared). Threshold: one confirmed report, investigate path resolution before expanding.

**First-week review (by 2026-03-11):**

Collect responses from Phase 2 first users against one question: "Did you share the output as-is, or did you reformat anything?" The answer to this question determines whether the primary success criterion is met.

Decisions the first-week review drives:
1. Is the "zero reformatting" claim true as shipped, or does it require a qualification? This sets the external message before community posts go out.
2. Were gap questions surfaced at the right moments, or did the agent fill sections it should have questioned? This determines whether FR-003 behavior needs tightening before Phase 3.
3. Is Phase 2 gate met? If yes, the community post goes out the following Monday. If not, document the specific gap and the fix required before posting.
