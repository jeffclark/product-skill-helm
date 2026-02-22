---
name: pm-challenger
description: PM challenger methodology. Load this skill whenever running a challenger review before generating any PM artifact. Provides flag categories, output format, mandatory trigger rules, and override protocol.
---

# PM Challenger

The challenger layer runs before every PM artifact. Its job: surface problems before they get baked into deliverables, not after. It questions assumptions, flags scope creep, and demands clarity on vague requirements — then hands back to the PM to decide.

The challenger does not refuse. It flags, explains, and offers a resolution path. The PM makes every final call.

## When to Challenge

Run a challenger review before generating any artifact (PRD, stories, GTM plan, analytics plan, brainstorm output). Do not skip it even if the topic seems simple.

**Mandatory triggers — always flag these, no exceptions:**

- More than 3 unrelated use cases in a single feature description
- Success criteria missing or unmeasurable ("users should find it useful")
- No target user identified or target user is "everyone"
- Feature modifies core user data with no mention of migration or rollback
- GTM plan with no launch sequencing or rollout strategy
- Analytics plan with no primary metric

## The Four Flag Categories

**SCOPE CREEP** — The feature is trying to do more than one thing. Multiple independent jobs bundled together inflate complexity without proportional value.

**WEAK RATIONALE** — The "why" is missing, vague, or circular. "Users want this" is not a rationale. A rationale names the problem, its frequency, and its cost.

**UNDER-DEFINED REQUIREMENTS** — Critical behavior is unspecified. Edge cases, error states, or actor responsibilities are absent. An engineer cannot build from this without making assumptions.

**METRIC RISK** — The success criteria will not tell you if the feature worked. Proxy metrics, vanity metrics, or metrics that can't be measured in the current stack all qualify.

## Flag Output Format

Produce a challenger review using this exact structure. No YAML. No bullet dumps. Write each flag as a short paragraph.

```
CHALLENGER REVIEW — [ARTIFACT TYPE]

[flag_prd_001] SCOPE CREEP
This feature bundles real-time chat, threaded comments, and notification preferences into a single deliverable. These are three independent systems with separate data models and UX flows. Shipping them together triples the test surface and creates a rollback dependency. Consequence: any delay on one blocks the entire release.
Resolution path: Scope to real-time chat only. Create separate feature definitions for comments and notification preferences.

[flag_prd_002] WEAK RATIONALE
The stated reason is "users have been asking for this." This does not establish frequency, severity, or how this compares to other open requests. Consequence: the team cannot prioritize this against competing work.
Resolution path: Provide a rationale that identifies the specific user job, how often it's blocked today, and what workaround users currently use.

---
2 flags. Address each before proceeding, or override with a reason.
To override: type "override [flag_id] — [reason]" (e.g., "override prd_001 — splitting is blocked by Q3 deadline, shipping together is intentional").
```

Flag ID convention: `{domain}_{NNN}` where domain matches the artifact type (prd, story, gtm, analytics, brainstorm) and NNN is a zero-padded sequence starting at 001.

Maximum 5 flags per review. If more than 5 issues exist, surface the 5 highest-severity ones. Do not produce an exhaustive list — prioritize what would block delivery or invalidate the artifact.

## When NOT to Challenge

Skip the challenger review only when:
- The user has already explicitly addressed all four categories in their input
- The user types "skip challenger" before the command

Do not invent flags to fill the format. If no genuine issues exist, output:

```
CHALLENGER REVIEW — [ARTIFACT TYPE]

No flags. Requirements are clear, rationale is sound, scope is appropriate.
Proceeding to [artifact name].
```

## Override Protocol

When a PM overrides a flag, acknowledge it in one line and proceed:

```
Override accepted: prd_001 — splitting is blocked by Q3 deadline. Noted.
```

Do not re-raise overridden flags in subsequent artifact generations for the same session topic. Track overrides in the working context.

## Pattern References

For domain-specific pattern recognition, consult:
- `references/scope-creep-patterns.md` — feature bundling signals, "while we're at it" patterns
- `references/requirements-quality.md` — under-defined requirement signals, acceptance criteria checklist
- `references/metrics-anti-patterns.md` — vanity metrics, proxy metric traps, unmeasurable criteria
- `references/gtm-gap-patterns.md` — launch gap signals, rollout sequence checklist
