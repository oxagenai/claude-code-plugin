---
description: Generate a full behavioural description of a function — what it calls, reads, writes, throws, and returns — grounded in the workspace ontology.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__ontology_explain_function, mcp__oxagen__ontology_get_node
---

# /oxagen:explain — behavioural description of a function

Produces a structured, ontology-grounded explanation of a function: its
purpose, inputs, outputs, side effects, error paths, and graph-visible
callers. Useful before editing a function you're unfamiliar with, or as
a sanity-check after refactoring.

## Steps

### 1 — Resolve the symbol

The argument is either:
- `<qualified-name>` — e.g. `oxagen.domains.billing.service.charge_credits`
- `<file>:<line>` — e.g. `packages/oxagen/oxagen/domains/billing/service.py:88`

Resolve to a node ID:

```
mcp__oxagen__ontology_get_node({
  identifier: "<argument>"
})
```

If no match:

> No node found for `<argument>`. Try `/oxagen:search "<symbol>"` to
> confirm the exact name or check that the workspace has been ingested.

### 2 — Call `ontology.explain_function`

```
mcp__oxagen__ontology_explain_function({
  node_id: "<resolved-node-id>"
})
```

Response fields:
- `summary` — one-paragraph behavioural description
- `inputs` — typed parameter list with descriptions
- `outputs` — return type + description
- `calls` — list of functions called
- `reads` — external state read (DB tables, env vars, config keys)
- `writes` — external state mutated
- `raises` — exception types and conditions
- `callers` — immediate callers from the graph (up to 10)

### 3 — Format the explanation

```
## <qualified-name>
Defined: <file:line>

<summary>

### Inputs
  <name>  <type>  — <description>
  ...

### Returns
  <type> — <description>

### Calls
  <file:line>  <qualified-name>
  ...

### Reads
  <resource>  (<type>)
  ...

### Writes
  <resource>  (<type>)
  ...

### Raises
  <ExceptionType>  when <condition>
  ...

### Called by (<N> immediate callers)
  <file:line>  <caller-name>
  ...
  [Run `/oxagen:impact <qualified-name>` for full blast radius]
```

Omit any section that returns empty data.

### 4 — Suggest follow-ups

- "Run `/oxagen:impact <name>` to see the full caller tree."
- "Run `/oxagen:search "tests for <name>"` to find test coverage."
- If `writes` is non-empty: "This function mutates external state — review
  callers before changing its contract."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `ontology.explain_function` errors | Symbol is ingested but explain data not yet computed — re-ingest or use `/oxagen:impact` for raw graph data |
| `ontology.get_node` returns null | Misspelled name or not yet ingested; try `/oxagen:search` |
| `calls` / `reads` / `writes` all empty | Ingest may be incomplete for this file; check connection status at `app.oxagen.ai` |
