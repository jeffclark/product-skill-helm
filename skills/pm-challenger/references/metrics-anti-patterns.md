# Metrics Anti-Patterns

Reference catalog for the pm-challenger skill. Use these signals to identify METRIC RISK flags.

## Vanity Metrics

Metrics that go up but don't tell you if the feature is working.

- **Page views / impressions** — measures exposure, not value
- **Registered users** — measures top-of-funnel, not activation
- **Feature opens / visits** — measures discoverability, not utility
- **"Time on page"** — ambiguous: could mean engaging or confused
- **Total items created** — measures volume, not whether items are used

Vanity metrics are fine as secondary signals. They are a METRIC RISK when used as the primary success criterion.

## Proxy Metric Traps

A proxy metric correlates with the real outcome but doesn't measure it directly. They break when behavior changes or when the proxy is optimized for its own sake.

**Common proxy traps:**
- Using NPS as the sole measure of product quality (NPS measures sentiment, not behavior)
- Using email open rate as a measure of feature awareness (opens ≠ action)
- Using support ticket volume as a measure of usability (not all confusion generates tickets)
- Using retention as the primary metric for a feature with no re-engagement mechanic

## Unmeasurable Criteria

Success criteria that cannot be instrumented with the current stack or available data.

**Signals:**
- Requires user-reported data (surveys, interviews) as the primary source of truth
- Requires comparing against a baseline that doesn't exist yet
- Measures something in a third-party system the team doesn't have access to
- Uses cohort analysis on a user base too small to reach statistical significance

## Framework Mismatch

Using the wrong measurement framework for the feature type.

**AARRR (Pirate Metrics)** — best for growth features, acquisition funnels, referral loops
- Acquisition: new users from specific channel
- Activation: first "aha moment"
- Retention: return usage
- Referral: invites / shares
- Revenue: conversion, upsell

**HEART** — best for user experience features, quality improvements, UX redesigns
- Happiness: satisfaction (surveys, ratings)
- Engagement: depth and frequency of use
- Adoption: % of eligible users using the feature
- Retention: return rate
- Task success: completion rate, error rate, time-on-task

Flag METRIC RISK when: a growth feature uses only HEART metrics, or a UX improvement feature uses only AARRR metrics without an engagement or task success component.

## Resolution Paths

- Replace vanity metrics with behavioral metrics that measure the feature's job
- Add a leading indicator (early signal) + a lagging indicator (outcome) pair
- Define the measurement method alongside the metric (what event fires, what query produces this number)
- For proxy metrics: name the real outcome being proxied and explain why direct measurement isn't feasible
