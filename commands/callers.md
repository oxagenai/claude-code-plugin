---
description: List callers of a function or method — direct callsites in the workspace ontology, ordered by PageRank.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_find_callers, mcp__oxagen__ontology_get_node
---

# /oxagen:callers — direct callers of a symbol

Wraps `code.find_callers` (alias `code.callers_of`). Use when you only
need the upstream caller list and don't want the full
`/oxagen:impact` blast-radius report.

## Steps

### 1 — Resolve the symbol

The argument is either a qualified name
(`oxagen.domains.billing.service.charge_credits`) or a
`<file>:<line>` locator.

If the input looks like a name, optionally pre-resolve it:

```
mcp__oxagen__ontology_get_node({ identifier: "<argument>" })
```

### 2 — Call `code.find_callers`

```
mcp__oxagen__code_find_callers({
  name: "<symbol>",   // or pass node_id if you resolved one
  depth: 1,
  limit: 50
})
```

The response uses the SPEC §7.9 envelope: `results`, `evidence`,
`tokens_used_estimate`, `counterfactual_estimate_tokens`,
`counterfactual_method`, `cursor`, `tenant_scoped_at`.

### 3 — Format

```
Callers of <symbol> (<N>)

  <file:line>  <caller_qualified_name>  (<caller_type>)
  ...

  +M more — pass `depth: 2` to widen, or run `/oxagen:impact` for the
  full blast-radius report.
```

### 4 — Suggest follow-ups

- "Run `/oxagen:impact <symbol>` for the full caller + dependency
  + path report."
- "Run `/oxagen:callees <symbol>` to flip direction."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `ontology_get_node` returns null | Run `/oxagen:search` to confirm the name |
| Empty result on a known-public symbol | Ingest may be incomplete; check `app.oxagen.ai/workspaces/.../connections` |
