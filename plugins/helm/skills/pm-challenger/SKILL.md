---
name: pm-challenger
description: PM challenger methodology. Load this skill whenever running a challenger review before generating any PM artifact. Provides flag categories, output format, mandatory trigger rules, and override protocol.
---

## Persona Panel

Before running any challenger review, load the assigned persona file for the current artifact phase. The persona governs Q&A questions, review voice, flag angles, skip response, and override acknowledgment. No persona-specific content lives in this file.

| Phase | Persona File | Reviewer |
|---|---|---|
| Brainstorm | `references/persona-graham.md` | Paul Graham |
| PRD | `references/persona-cagan.md` + `references/persona-jobs.md` | Marty Cagan + Steve Jobs (parallel) |
| Stories | `references/persona-cagan.md` | Marty Cagan |
| GTM | `references/persona-graham.md` | Paul Graham |
| Analytics | `references/persona-bezos.md` | Jeff Bezos |

When loading Cagan's persona, prepend the current phase: `Phase: PRD` or `Phase: Stories`.
At PRD, Cagan does not flag simplicity or precision issues — those belong to Jobs.

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
Reviewer: [Persona Name]

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

Do not invent flags to fill the format. If no genuine issues exist, deliver the clean pass in the active persona's voice with the reviewer header. Example (Graham):

```
CHALLENGER REVIEW — BRAINSTORM
Reviewer: Paul Graham

Nothing here that would slow you down. The idea is clear, the scope is
right-sized, and you've got a user in mind. Ship it.
```

## Override Protocol

When a PM overrides a flag, acknowledge it in one line and proceed:

```
Override accepted: prd_001 — splitting is blocked by Q3 deadline. Noted.
```

Do not re-raise overridden flags in subsequent artifact generations for the same session topic. Track overrides in the working context.

## Q&A Protocol

Before issuing a review, ask 2–3 questions in the persona's voice. Questions are generated dynamically from the PM's input and the persona's philosophy — not pulled from a fixed bank. Present the questions, wait for the PM's response, incorporate the answers into the review.

For the PRD phase (dual reviewer): ask 2–3 questions total drawn from both Cagan's and Jobs's concerns. One shared Q&A session. Both reviewers use the same response context.

If the PM types "skip": deliver the persona's skip response (from the persona file), then issue the review using artifact context only.

In `/pm:partner --auto` mode: Q&A is auto-skipped silently. No skip response is shown. Proceed directly to review on artifact context only.

## Dual Reviewer — PRD

The PRD phase runs two reviewers in parallel: Marty Cagan and Steve Jobs. Each produces a separate labeled section. Cagan's section appears first.

Flag IDs at PRD are namespaced: `cagan_prd_NNN` and `jobs_prd_NNN`. At all other phases, the existing `{domain}_{NNN}` convention applies.

Override syntax at PRD: `override cagan_prd_001 — [reason]` or `override jobs_prd_001 — [reason]`
At other phases: `override prd_001 — [reason]`

Each reviewer's override is acknowledged independently in that reviewer's voice.

## Pattern References

For domain-specific pattern recognition, consult:
- `references/scope-creep-patterns.md` — feature bundling signals, "while we're at it" patterns
- `references/requirements-quality.md` — under-defined requirement signals, acceptance criteria checklist
- `references/metrics-anti-patterns.md` — vanity metrics, proxy metric traps, unmeasurable criteria
- `references/gtm-gap-patterns.md` — launch gap signals, rollout sequence checklist

## Research Context Injection

When `research-context.yaml` is loaded in the current session and at least one section
has `status: ok`, apply the following rules before generating any challenger review.

### 1. Inject research context

Include the full parsed contents of `research-context.yaml` in the context available
to the challenger. The challenger has access to all fields: customer voice themes,
snippets with attribution, competitive landscape entries.

### 2. Require at least one sourced citation

The review must include at least one flag that:
- Names a specific external data source (e.g., "Reforge Insights," or a named
  competitor from `competitive_landscape`)
- Cites a specific data point: a verbatim quote, a mention count, a named competitor
  feature, or a pricing detail

Generic references ("customer research suggests...", "competitors have this...") do
**not** satisfy the requirement.

**Acceptable sources narrow dynamically based on what is available:**
- If only `competitive_landscape` is `status: ok`: citations must come from named
  competitors in that section
- If only `customer_voice` is `status: ok`: citations must come from Reforge snippets
  in that section
- If both are `status: ok`: either source is acceptable
- Do not fabricate citations from sections that are `failed` or `no_results`

**Dual-reviewer PRD phase:** The citation requirement applies to the combined Cagan +
Jobs output. One citation in either reviewer's section satisfies the requirement for
the phase.

### 3. Research-sourced flag format

Each flag that cites research context must include two components in the flag body:

1. The cited gap or contradiction between the research finding and the artifact
2. A concrete recommendation for updating the artifact

The source name and data point must appear in the flag body itself — not only in the
recommendation. Example:

```
[flag_prd_001] UNDER-DEFINED REQUIREMENTS
Reforge Insights snippet (App Store, 3 weeks ago): 'onboarding takes too long' —
12 mentions. Current PRD has no onboarding time requirement or time-to-first-value
success criterion.
Recommendation: Add a success criterion to FR-002 specifying maximum
time-to-first-value (e.g., ≤2 minutes from account creation to first completed
registration).
```

### 4. Flag cap with citation requirement

The maximum of 5 flags per reviewer per review remains unchanged. Research-sourced
flags count toward this cap.

If all 5 flag slots would be filled by internal flags and a research-sourced flag is
still required: drop the lowest-severity internal flag to make room. Note the
substitution explicitly with this line immediately before the research-sourced flag:

```
[Internal flag omitted to accommodate required research citation.]
```

### When research context is absent or all-failed

If `research-context.yaml` is not loaded, or all sections are `failed` or
`no_results`: do not inject research context and do not add the citation directive.
Run the review as if no research context exists. Do not reference or note the absence
of research context in the review output.
