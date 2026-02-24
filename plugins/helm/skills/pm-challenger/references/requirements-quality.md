# Requirements Quality Patterns

Reference catalog for the pm-challenger skill. Use these signals to identify UNDER-DEFINED REQUIREMENTS flags.

## Under-Defined Requirement Signals

**Missing actor**
"The user can do X" — which user? Roles matter when permissions, data access, or UI vary by actor.

**Missing trigger**
"Show a confirmation dialog" — when? On what action? Under what conditions? Triggers define the start of every flow.

**Missing failure state**
"Submit the form" — what happens if submission fails? Retry? Rollback? Silent failure? Missing failure states are bugs waiting to happen.

**Missing edge case for core data operations**
Any feature that writes, deletes, or migrates user data must specify: what happens to existing data? What happens to in-flight operations? What's the rollback path?

**Unmeasurable acceptance criteria**
- "Users should find it easy" → not testable
- "Should be fast" → specify a threshold
- "Handles large datasets" → define "large"
- "Works on mobile" → specify breakpoints or device targets

**Ambiguous ownership**
"The system will notify the user" — which system? Which team owns the notification infrastructure? If it crosses a service boundary, the boundary must be named.

## Acceptance Criteria Checklist

A well-defined requirement answers all of:

- [ ] Who is the actor (role, permission level)?
- [ ] What triggers this behavior?
- [ ] What is the expected output / state change?
- [ ] What happens when the trigger condition is not met?
- [ ] What is the error state and recovery path?
- [ ] Is the success condition measurable (has a threshold or can be pass/fail tested)?

If more than 2 of these are missing for a core requirement, flag UNDER-DEFINED REQUIREMENTS.

## Common Resolution Paths

- Rewrite acceptance criteria in Given/When/Then format to surface missing conditions
- Add explicit error states for every write operation
- Replace subjective qualifiers ("fast," "easy," "clean") with measurable thresholds
- Name the actor explicitly in each criterion
- For data operations: add a "data contract" section specifying input format, validation rules, and persistence expectations
