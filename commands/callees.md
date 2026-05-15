---
description: List functions and methods that a symbol calls (its outgoing CALLS edges) ordered by PageRank.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_callees_of, mcp__oxagen__ontology_get_node
---

# /oxagen:callees — outgoing calls from a symbol

Wraps `code.callees_of` (M9, SPEC §7.2). Inverse of
`/oxagen:callers` — returns what the symbol itself calls.

## Steps

### 1 — Resolve the symbol

Argument is either `<file>:<line>` or a qualified name.

### 2 — Call `code.callees_of`

```
mcp__oxagen__code_callees_of({
  name: "<symbol>",
  depth: 1,
  limit: 50
})
```

Returns the standard SPEC §7.9 envelope (`results`, `evidence`,
`tokens_used_estimate`, `counterfactual_estimate_tokens`,
`counterfactual_method`, `cursor`, `tenant_scoped_at`).

### 3 — Format

```
Callees of <symbol> (<N>)

  <file:line>  <callee_qualified_name>  (<callee_type>)
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:explain <symbol>` for the behavioural breakdown."
- "Run `/oxagen:path <a> <b>` to find a directed call chain."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | The symbol may be a leaf or ingest is incomplete |
| `ontology_get_node` null | Run `/oxagen:search` to confirm the name |
