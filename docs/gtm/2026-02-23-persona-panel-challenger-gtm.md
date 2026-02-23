---
title: Named Persona Panel for PM Challenger — GTM
type: gtm
date: 2026-02-23
topic: persona-panel-challenger
status: draft
prd: docs/prds/2026-02-23-persona-panel-challenger-prd.md
---

# Named Persona Panel for PM Challenger — GTM

## Feature Summary

The PM Challenger now reviews your artifacts through named product leader personas — Paul Graham at brainstorm, Marty Cagan and Steve Jobs at PRD, Cagan again at stories, Graham at GTM, and Jeff Bezos at analytics. Each reviewer asks 2–3 questions in their own voice before issuing flags. The reviewer is a sparring partner, not a checklist.

---

## Positioning

**One-sentence value prop:** Helm takes you from idea to dev-ready artifacts in 30 minutes, with Paul Graham, Marty Cagan, Steve Jobs, and Jeff Bezos pushing back at every step.

**Differentiation from generic AI assistants:** ChatGPT and Copilot produce documents. Helm produces documents that have already been challenged. The distinction is not the output format — it is who pushed back on the content before you shipped it to engineering. Generic AI assistants have no identity and no philosophy. They cannot tell you that Jobs would have cut half your scope or that Graham thinks you have not talked to enough users. Helm can.

---

## Messaging

**Primary hook:** "0 to ready for dev in 30 minutes — with Graham, Cagan, Jobs, and Bezos in the room."

**Supporting proof points:**

- Every artifact phase has an assigned reviewer with a known philosophy. You know exactly what lens is being applied before you read the first flag.
- The reviewers ask questions before they flag. A Cagan review that starts with "What is the riskiest assumption you have not yet tested?" is different from a review that starts with a flag list. You get the thinking, not just the verdict.
- The skip flow is honest. If you skip the Q&A, Graham tells you in one paragraph what he was trying to surface and why it mattered — then steps aside. Skipping still costs you something.
- Adding a new persona is one file and one table row. The panel is extensible by design.

**What not to say:**

- Do not say "AI-powered product reviews" — it describes the mechanism, not the value, and every tool in the category can say it.
- Do not say "like having a board of advisors" — it overclaims the relationship and invites skepticism about accuracy.
- Do not say the personas are accurate simulations of how these people would actually respond — they are philosophical lenses, not impersonations. The value is the framework, not the fidelity.
- Do not lead with the five-command lifecycle. The persona panel is the differentiator this launch. The lifecycle is the context that makes it credible.

---

## Chapter 1 — Internal Launch

**Audience:** G, Guiv, Brittni — three internal PMs who will use this as their actual workflow, not as a test.

**Goal:** Learn whether the "30 minutes" hook is true, which persona interactions land, and where the workflow breaks under real use.

### Actions this week

1. **Send a direct Slack message to each of G, Guiv, and Brittni today.** Not a group message. Not a channel post. One message per person, written for them. The message should include: what Helm does, the "30 minutes" claim, and a specific ask — "use it on the next real thing you are working on and tell me what broke." Attach the plugin install path and one-sentence setup instructions.

2. **Include one worked example in the message.** Walk through a real artifact run — brainstorm through GTM — showing the Graham Q&A at brainstorm and the Cagan/Jobs dual review at PRD. This is not a demo video. It is a annotated transcript pasted inline so they can see what the interaction actually looks like before they try it.

3. **Set a five-business-day check-in.** Calendar invite with each person, 20 minutes, scheduled at the moment you send the Slack message. Do not wait for them to come back to you. The check-in is the forcing function.

4. **Create a single shared doc (Notion or Google Doc) for feedback.** Give each person a section with two prompts: "What broke or felt wrong" and "Did the 30-minute claim hold." Link it in the Slack message. This is the artifact capture mechanism for Chapter 1.

### What to learn

- Does the "30 minutes" claim hold in practice, or does it break at a specific phase? If it breaks, where?
- Which persona interaction generates the most engagement? Graham's bluntness, Cagan's discovery framing, Jobs's simplicity cuts, or Bezos's mechanism questions?
- Do PMs answer the pre-review questions, skip them, or get confused by the Q&A format?
- Is there a phase where the persona voice feels off — where the reviewer says something that would not fit that person's actual philosophy?
- Does the dual Cagan/Jobs PRD review feel valuable or feel like overhead?

### Success criteria for Chapter 1

- All three PMs have completed at least one full artifact run (brainstorm through GTM) within five business days of receiving the Slack message.
- At least two of three report that the 30-minute claim is plausible or true.
- At least one specific interaction (a persona question, a skip response, or a flag in a named voice) is called out by name as the moment the review felt different from a generic checker.
- No phase produces a broken or incoherent persona response that a PM flags as "this does not sound like that person at all."
- Feedback doc has entries from all three PMs.

### What Chapter 1 is not

Chapter 1 is not a test of whether people like it. It is a test of whether the "30 minutes" claim is true and whether the persona panel is the differentiator. Positive sentiment without evidence on those two questions does not unlock Chapter 2.

---

## Chapter 2 — External Expansion

**Audience:** PMs at growth-stage, AI-native companies who use Claude Code. The specific profile: individual contributor PMs or product leads at 50–500 person companies where Claude Code is already in the workflow and where the PM is the bottleneck — not engineering, not design, not prioritization, but artifact production itself.

**Goal:** Get Helm into the daily workflow of 10–20 PMs outside the company and establish a repeatable distribution path for subsequent launches.

### Criteria to move from Chapter 1 to Chapter 2

All of the following must be true before any external outreach begins:

- Chapter 1 success criteria are met (all three PMs completed a full run, 30-minute claim validated by at least two).
- At least one verbatim quote is in hand that names the persona panel specifically — not Helm generally. This quote is the anchor for all external messaging.
- No broken persona behavior has been reported and left unresolved. If a persona voice is broken and a fix is in flight but not deployed, Chapter 2 does not start.
- The install path works end-to-end without requiring any assistance. If any Chapter 1 user needed help installing, that is fixed first.

### Rollout sequence

| Phase | Cohort | Criteria to expand | Target date |
|-------|--------|--------------------|-------------|
| 1 — Internal | G, Guiv, Brittni | All three complete full run; 30-min claim validated; no broken persona behavior | Week of 2026-02-23 |
| 2 — Warm external | 5–10 PMs via direct outreach (LinkedIn, Slack DMs, personal network) — must be Claude Code users at growth-stage companies | 7 of 10 complete a full run; at least 3 unprompted shares or referrals; no new broken behavior | 2–3 weeks after Chapter 1 gate is met |
| 3 — Plugin ecosystem | Submit to Claude Code plugin directory or equivalent; post in relevant Slack/Discord communities (e.g., Lenny's, Mind the Product, AI-native PM forums) | Baseline install and run metrics established; support load is manageable without dedicated bandwidth | 2–3 weeks after Phase 2 gate is met |

**Pause/rollback trigger:** If any external user reports a persona voice that produces an incoherent or offensive response (not just "off" — actively wrong or harmful), all external promotion stops immediately. The plugin is not pulled from existing users, but no new outreach occurs until the persona file is corrected and validated by an internal full run.

### Channels — Chapter 2

**Direct outreach (Phase 2):** Personal network first. The ask is not "try this tool" — it is "I built something and I need 20 minutes of honest feedback on whether the '30 minutes' claim is real." That framing gets a higher response rate than a product pitch and produces more useful signal.

**Lenny's Slack and similar PM communities (Phase 3):** Post in the tools/AI channel with a worked example — not a feature list. Show a real Cagan PRD review interaction. Let the persona panel speak for itself. Include the verbatim quote from a Chapter 1 or Phase 2 user.

**Claude Code plugin ecosystem (Phase 3):** Submit for inclusion in any official or community-maintained plugin directory. The listing copy uses the same primary hook. Link to a 60-second worked example — the Graham brainstorm Q&A and the Cagan/Jobs dual PRD review are the two moments most likely to convert a reader into an installer.

**What not to do in Chapter 2:** Do not post in channels where Claude Code is not already in the workflow. The install assumption is real — a PM who has not adopted Claude Code is not the target and the conversion cost is too high for this stage.

---

## Launch Checklist

- [ ] All four persona files (`persona-graham.md`, `persona-cagan.md`, `persona-jobs.md`, `persona-bezos.md`) are complete and produce coherent, in-character reviews across all assigned phases
- [ ] SKILL.md phase-to-persona mapping table is finalized and matches the PRD panel assignment
- [ ] Skip flow tested for each persona — skip response is in character and does not read as a generic fallback
- [ ] Dual Cagan/Jobs PRD review tested — flag IDs are namespaced, scopes do not overlap, PM can address each reviewer independently
- [ ] Jobs scope constraint verified — Jobs does not raise flags outside the three specified categories even when genuine out-of-scope issues exist
- [ ] Install path tested end-to-end on a clean machine with no prior Helm setup
- [ ] Worked example (annotated full run transcript, brainstorm through GTM) prepared and ready to send with Chapter 1 outreach
- [ ] Shared feedback doc created with prompts for all three Chapter 1 PMs
- [ ] Five-business-day check-in calendar invites sent to G, Guiv, and Brittni at the same time as the Slack message
- [ ] Chapter 1 success criteria documented and agreed — the gate conditions are written down before the run begins, not evaluated after

---

## Risks and Mitigations

**Risk 1: The "30 minutes" claim fails in practice and there is no honest way to defend it.**

This is the highest-stakes risk because the hook is the entire external message. If Chapter 1 reveals that the actual time is 90 minutes, or that PMs get stuck at a specific phase, the external message must change before Chapter 2 begins. The mitigation is that Chapter 1 is explicitly structured to test this claim — all three PMs are asked specifically whether it held, and the check-in is scheduled before results are in. If the claim fails: find the phase where time breaks, fix it or reframe the message to reflect the actual value (e.g., "the hardest 30 minutes of product thinking compressed into 10"). Do not defend the original claim if the data says otherwise.

**Risk 2: The persona voices are philosophically off enough that PMs who know Cagan or Graham discount the tool entirely.**

The target user knows who these people are. A Cagan review that says something Cagan would never say is not a minor issue — it is the differentiating feature failing in front of the user who cares most. The mitigation is: before any external launch, at least one Chapter 1 PM who has read Cagan's work validates that the Cagan review reads as credible. "Credible" does not mean accurate to every published position. It means the questions and flags are consistent with the publicly known philosophy. If a Chapter 1 PM flags a specific Cagan interaction as wrong, it is fixed before Chapter 2. This risk does not have a workaround — it requires the persona files to be right.

**Risk 3: The dual PRD review (Cagan + Jobs) feels like overhead rather than value and reduces, not increases, engagement with the challenger.**

Two reviewers at the same phase is the one structural decision in this launch that has no precedent to validate against. If Chapter 1 PMs report that reading two review sections at PRD is too much — that they skim the second one or skip both — the dual review is a liability. The mitigation is to ask this question explicitly in the five-day check-in: "Did the dual Cagan/Jobs review at PRD feel like more value or more noise?" If two of three Chapter 1 PMs say noise, the Jobs scope at PRD is narrowed further (e.g., Jobs only flags when a PRD exceeds N requirements, as a complexity signal) or moved to a separate, optional review pass. This decision cannot be made without Chapter 1 data, but it must be made before Chapter 2 begins.

---

## Post-Launch Monitoring

**Watch for in the first 48 hours after Chapter 1 outreach:**

- Any report of a persona producing a response that is incoherent, out of character, or actively misleading. Threshold: one confirmed report stops external promotion until resolved.
- Any report of the install or first-run flow being broken. Threshold: one confirmed report triggers an immediate fix before the five-day check-in.
- Whether any Chapter 1 PM has run the tool unprompted (outside of the initial ask). This is an early signal on habit formation, not a gate criterion.

**First-week review (by 2026-03-02):**

Review the feedback doc and check-in notes against the Chapter 1 success criteria. The decisions this review drives:

1. Is the "30 minutes" claim defensible as written, needs qualification, or needs replacement? This sets the external message before any outreach begins.
2. Which persona interaction generated the most signal? This determines what the Phase 2 outreach message leads with.
3. Is the dual PRD review a keeper or a liability? This determines whether Chapter 2 goes out with the feature as shipped or with a modification.
4. Is the gate to Chapter 2 met? If yes, begin direct outreach to warm external contacts the following Monday. If not, document the specific gap and the fix needed.
