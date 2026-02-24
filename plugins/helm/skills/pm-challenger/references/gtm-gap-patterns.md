# GTM Gap Patterns

Reference catalog for the pm-challenger skill. Use these signals to identify SCOPE CREEP and UNDER-DEFINED REQUIREMENTS flags in GTM plans.

## Launch Gap Signals

**No rollout sequence**
A GTM plan that describes messaging but not sequencing is incomplete. Who hears about this first? Internal teams, beta users, or all users simultaneously? Missing sequence means the team will improvise at launch.

**No stakeholder communication plan**
Features that touch other teams (sales, support, success, finance) require internal communication before external communication. If the GTM plan only addresses external messaging, flag it.

**No support readiness signal**
Any customer-facing feature needs a support readiness step: FAQ, known issues, escalation path. If support is not named in the GTM plan, they will be caught off guard.

**No rollback or pause trigger**
What causes the team to pause or roll back the launch? Missing criteria means the team will debate in real time during an incident. A GTM plan should name the signal that triggers a rollback decision.

**Messaging without differentiation**
Messaging that could apply to any feature from any company is undifferentiated. It will not cut through. Flag messaging that lacks: specific problem named, specific user named, or specific outcome claimed.

## Rollout Sequence Checklist

A complete rollout sequence answers:

- [ ] Who is in the initial cohort (internal, beta, % of users, specific segments)?
- [ ] What is the hold criteria before expanding rollout?
- [ ] What is the full rollout timeline (not just launch date — phases)?
- [ ] Which teams need to be notified before external announcement?
- [ ] What is the pause/rollback trigger?

If more than 2 of these are missing, flag it.

## Stakeholder Communication Gaps

**Sales** — need to know before customers ask. If the feature affects pricing or packaging, sales needs a talk track.

**Support / Success** — need documentation, known issues, and an escalation path. Never launch without them.

**Legal / Compliance** — any feature touching personal data, billing, or regulated industries requires a legal review signal in the GTM plan.

**Internal users** — if your company uses your own product, internal users should be in the first rollout cohort.

## Messaging Anti-Patterns

- Feature-led messaging ("We added X") instead of benefit-led ("Now you can Y")
- Audience mismatch: technical feature description sent to non-technical users
- Over-promising: "never worry about X again" for a feature that reduces but doesn't eliminate X
- Missing call to action: announcement with no clear next step for the user

## Resolution Paths

- Add a rollout phases table: Cohort / Criteria to expand / Timeline
- Name each stakeholder team and the specific artifact they need (doc, email, training)
- Rewrite messaging in the format: "For [user], [feature] means [outcome]."
- Define the rollback trigger as a specific, observable signal (not "if things go wrong")
