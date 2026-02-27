---
title: research-context.yaml — v1 Schema Reference
type: schema
date: 2026-02-24
---

# `research-context.yaml` — v1 Schema Reference

This document is the canonical reference for the `research-context.yaml` file written
by `/pm:research` and `/pm:partner` phase 0. All components that read or write this
file must conform to this schema.

The file lives at the **project root** — the same directory as `product-context.yaml`.

---

## Top-Level Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `research_context.topic` | string | yes | Free-text topic string passed to the command |
| `research_context.generated_at` | string | yes | ISO 8601 UTC datetime (e.g. `2026-02-24T14:30:00Z`) |
| `research_context.generated_by` | string | yes | See enum below |
| `research_context.customer_voice` | object | yes | See section below |
| `research_context.competitive_landscape` | object | yes | See section below |
| `research_context.pm_notes` | string | yes | Always present; empty string `""` if no notes |

### `generated_by` enum

Valid values for v1:

| Value | Written by |
|---|---|
| `"pm:research"` | `/pm:research` command |
| `"pm:partner"` | `/pm:partner` phase 0 |

No other values are valid in v1. This field exists for Phase 2 provenance; no current
downstream behavior branches on it.

---

## `customer_voice`

| Field | Type | Notes |
|---|---|---|
| `status` | enum | `ok \| failed \| no_results` |
| `error` | string | Present **only** when `status: failed`; omit otherwise |
| `source` | string | `"Monterey AI / Reforge Insights"` |
| `query` | string | The search query sent to `search_snippets` |
| `snippet_count` | integer | Count of snippets returned; 0 when `status: no_results` |
| `themes` | list | Present when `status: ok`; may be empty list |
| `snippets` | list | May be empty list when `status: no_results` |

### `themes` items

| Field | Type | Notes |
|---|---|---|
| `label` | string | Theme name |
| `summary` | string | One-sentence summary |
| `mention_count` | integer | Omit if unavailable |

### `snippets` items

| Field | Type | Notes |
|---|---|---|
| `text` | string | Verbatim quote |
| `source` | enum | `App Store \| Gong \| Survey \| unknown` |
| `date` | string | ISO 8601 date or `null` |
| `sentiment` | enum | `positive \| negative \| neutral \| unknown` |

---

## `competitive_landscape`

| Field | Type | Notes |
|---|---|---|
| `status` | enum | `ok \| failed \| no_results` |
| `error` | string | Present **only** when `status: failed`; omit otherwise |
| `competitors` | list | Must have ≥1 entry when `status: ok`; empty list when `status: no_results` |

### `competitors` items

| Field | Type | Notes |
|---|---|---|
| `name` | string | Competitor name |
| `features` | string | 2–4 sentence summary of relevant feature set |
| `positioning` | string | 1–2 sentence positioning description |
| `pricing` | string | Pricing summary or `null` |
| `source_url` | string | URL of primary source or `null` |

---

## `pm_notes`

Always present as a string. Empty string `""` if no notes have been added.

**YAML encoding rule:** When `pm_notes` contains content, use a YAML block scalar
(`|` literal style). When multiple notes have been appended across sessions, they are
separated by a single newline within the block scalar.

```yaml
# Empty notes
pm_notes: ""

# Single note (block scalar)
pm_notes: |
  Research looks solid. Competitor pricing data may be out of date — verify Acme's
  new pricing tier before PRD.

# Multiple appended notes (newline-separated within block scalar)
pm_notes: |
  Research looks solid. Competitor pricing data may be out of date — verify Acme's
  new pricing tier before PRD.
  Customer voice data is from 6 weeks ago — mostly App Store. Gong data thin.
```

---

## Structural Constraints

- `generated_at` must be present and valid ISO 8601 UTC on every write. Commands use
  this field for staleness calculation.
- `competitive_landscape.competitors` must contain ≥1 entry when `status: ok`.
- `customer_voice.snippets` may be an empty list when `status: no_results`.
- **1-competitor threshold:** Exactly 1 named competitor from web search → `status:
  no_results`, `competitors: []`. Competitive positioning requires ≥2 named competitors
  to reach `status: ok`.
- `pm_notes` is always present. Writers must include it even when empty.
- The file must be valid YAML. If serialization produces invalid YAML, the write must
  be aborted and an error surfaced to the terminal. Never write a partial file.

---

## Deferred Fields (Phase 2)

**`citation_impact` must NOT be written in v1.** It is reserved for Phase 2.

When Phase 2 is implemented, it will track whether research-sourced challenger flags
led to artifact updates. Do not include this field in any v1 read or write logic.

---

## Complete Annotated Example

```yaml
research_context:
  topic: "youth sports registration mobile experience"
  generated_at: "2026-02-24T14:30:00Z"
  generated_by: "pm:research"

  customer_voice:
    status: ok
    source: "Monterey AI / Reforge Insights"
    query: "youth sports registration mobile experience"
    snippet_count: 14

    themes:
      - label: "Checkout friction"
        summary: "Parents report too many steps to complete registration on mobile."
        mention_count: 8
      - label: "Payment confusion"
        summary: "Unclear what fees cover and why totals change at checkout."
        mention_count: 5
      - label: "Roster visibility"
        summary: "Coaches want to see who has registered without logging in separately."
        mention_count: 4

    snippets:
      - text: "I had to enter my kid's info three separate times. This is absurd."
        source: App Store
        date: "2026-01-15"
        sentiment: negative
      - text: "Finally got the app to work but it took me 20 minutes to register."
        source: App Store
        date: "2026-02-01"
        sentiment: negative
      - text: "Love that I can pay in installments now. Makes it so much easier."
        source: Survey
        date: "2026-01-28"
        sentiment: positive
      - text: "Wish I could save payment info. I register 4 kids every season."
        source: Gong
        date: "2026-02-10"
        sentiment: neutral
      - text: "Registration confirmation came through immediately. Very smooth."
        source: App Store
        date: "2026-02-18"
        sentiment: positive

  competitive_landscape:
    status: ok
    competitors:
      - name: "TeamSnap"
        features: "Mobile-first registration with saved payment profiles, one-tap re-enrollment for returning families, and in-app roster management. Supports Apple Pay and Google Pay at checkout."
        positioning: "Positioned as the all-in-one team management platform; registration is a conversion entry point into their broader comms and scheduling tools."
        pricing: "$12/month per team for basic; custom pricing for leagues"
        source_url: "https://www.teamsnap.com/features"
      - name: "SportsEngine"
        features: "Unified registration across multiple sports and age groups per family. Bulk discount application, split-payment plans, and integrated background checks for coaches."
        positioning: "Targets league administrators and multi-sport clubs; emphasizes compliance and background check integration as differentiators."
        pricing: "Per-transaction fee model; contact for league pricing"
        source_url: "https://www.sportsengine.com/solutions/registration"
      - name: "Sportably"
        features: "Lightweight mobile registration with QR-code check-in for walk-up registrations. No saved profiles; optimized for low-friction single-event signups."
        positioning: "Targets single-sport recreational leagues that want minimal setup; not designed for multi-team organizations."
        pricing: "Free up to 50 registrants; $0.99 per registration above that"
        source_url: null

  pm_notes: |
    TeamSnap Apple Pay integration is worth flagging to the PRD — if we don't support
    it we'll keep losing parents who registered on mobile and abandoned at payment.
```
