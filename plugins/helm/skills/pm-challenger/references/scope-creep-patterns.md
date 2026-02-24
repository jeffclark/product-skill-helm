# Scope Creep Patterns

Reference catalog for the pm-challenger skill. Use these signals to identify SCOPE CREEP flags.

## Feature Bundling Signals

**"While we're at it" additions**
A feature description that includes "and also," "while we're at it," or "at the same time" almost always bundles unrelated work. Each "and also" is a candidate for a separate feature.

**Multiple actor types in one feature**
If the feature serves both admins and end users with different flows, it is likely two features. Different actors have different success criteria and different failure modes.

**Multiple data model changes**
A single feature that requires new columns on 3+ unrelated tables is a bundling signal. Count the distinct schema changes — each one represents an independent concern.

**"Phase 2 is already defined"**
If the description includes a phase 2 ("in a later iteration, we'll add X"), that's a sign the current scope has already been pre-expanded. Challenge whether phase 2 should influence phase 1 architecture at all.

## "While We're at It" Anti-Patterns

- Adding a notification system to a feature whose core job is data display
- Including a settings/preferences panel in a feature that's really about CRUD
- Adding bulk operations to a feature spec for individual record editing
- Adding export/import to a feature spec focused on filtering/search
- Adding an audit log to a feature that hasn't established a core write path yet

## Scope Creep Severity Scale

**High** — Independent systems bundled with separate data models (e.g., messaging + feed + notifications)
**Medium** — Related but separable concerns that share a UI surface (e.g., inline editing + drag-to-reorder)
**Low** — Minor additions that share the same data model and actor (e.g., adding sort options to a list view that already has filtering)

Flag HIGH and MEDIUM. Use judgment on LOW — if it materially increases test surface or rollback risk, flag it.

## Resolution Patterns

- Narrow to a single actor, single job
- Define a "walking skeleton" — the minimum that delivers the core value
- Move bundled items to a separate feature doc and link to it
- If bundling is intentional (deadline, dependency), the PM should state this explicitly in the rationale
