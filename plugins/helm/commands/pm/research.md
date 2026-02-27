# /pm:research

**Note: The current year is 2026.**

Gather external market signal for a feature or topic: customer voice via Reforge
Insights MCP and competitive intelligence via web search. Writes findings to
`research-context.yaml` at the project root and prints a human-readable summary.

All other Helm commands load this file automatically when present. Run this command
before or alongside brainstorm and PRD work to ground challengers in real market data.

## Input

<input> #$ARGUMENTS </input>

**$ARGUMENTS disambiguation:**
- Otherwise → topic to research (free-text, e.g. "mobile registration checkout")

If $ARGUMENTS is empty, ask: "What topic should I research?"

## Execution

### 1. Parse topic

Use `$ARGUMENTS` as the topic string as-is.

### 2. Call Reforge Insights MCP

Call the Monterey AI MCP server `search_snippets` tool with the topic string as the
semantic search query. Apply a 30-second timeout.

- **Success with results** → `customer_voice.status: ok`; populate `snippets`,
  `themes`, `snippet_count` from the returned data
- **Timeout or connection failure** → `customer_voice.status: failed`; set
  `customer_voice.error` to the failure message; continue to step 3
- **Tool not available in environment** → `customer_voice.status: failed`; set
  `customer_voice.error` to `"Monterey AI MCP tool not available — verify MCP
  configuration"`; continue to step 3
- **Empty result set** → `customer_voice.status: no_results`; continue to step 3

Treat timeout, connection failure, and missing tool as the same `status: failed` path.
Do not halt execution on MCP failure — continue to competitive research.

### 3. Run competitive web search

Run a web search for: `[topic] competitors features pricing positioning`

- **≥2 named competitors found** → `competitive_landscape.status: ok`; populate
  `competitors` list. For each competitor include: `name`, `features` (2–4 sentence
  summary of relevant feature set), `positioning` (1–2 sentences), `pricing` (or
  `null` if unavailable), `source_url` (or `null` if unavailable)
- **Exactly 1 named competitor found** → `competitive_landscape.status: no_results`;
  `competitors: []`. One entry is insufficient for competitive positioning.
- **No usable results or web search error** → `competitive_landscape.status:
  no_results`; `competitors: []`

### 4. Carry forward `pm_notes`

Before writing, check for an existing `research-context.yaml` at the project root:

- **File present and valid YAML and `pm_notes` is non-empty** → capture the value
- **File present but invalid YAML** → treat as absent; `pm_notes = ""`
- **File absent or `pm_notes` is empty** → `pm_notes = ""`

### 5. Write `research-context.yaml`

Construct the YAML document conforming to `docs/research-context-schema.md`:

```yaml
research_context:
  topic: [topic string]
  generated_at: [current UTC datetime in ISO 8601, e.g. 2026-02-24T14:30:00Z]
  generated_by: "pm:research"
  customer_voice: [from step 2]
  competitive_landscape: [from step 3]
  pm_notes: [from step 4; use YAML block scalar (|) if multi-line]
```

Write `research-context.yaml` to the project root, overwriting any existing file.

**Do NOT write `citation_impact`** — that field is reserved for Phase 2.

If YAML serialization fails: print the error message, then print the human-readable
summary (step 6) from in-memory data only. Do not write the file. Never write a
partial file.

### 6. Print human-readable summary

```
## Research Summary — [topic]

### Customer Voice (Reforge Insights)
```

**If `customer_voice.status: ok`:**
```
Key themes:
- [theme label]: [theme summary] ([mention_count] mentions)
[repeat for each theme]

Top quotes (up to 5):
- "[snippet text]" — [source], [date]
[repeat for each quote]
```

**If `customer_voice.status: failed`:**
```
Customer voice research failed: [error message]
```

**If `customer_voice.status: no_results`:**
```
No customer voice data found for this topic.
```

Then:
```
### Competitive Landscape
```

**If `competitive_landscape.status: ok`:**
```
[Competitor name]: [features summary] [positioning summary] Pricing: [pricing or "not publicly available"].
[repeat for each competitor]
```

**If `competitive_landscape.status: no_results`:**
```
No competitive data found for this topic.
```

## Important Constraints

- **No challenger review** — This is a data-gathering command, not artifact-generating.
  Do not run or invoke the pm-challenger skill.
- **No `product-context.yaml` update** — Research is not an artifact type tracked in
  that file.
- **No `citation_impact` field** — Do not write it. Phase 2 only.
