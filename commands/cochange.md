---
description: List symbols that historically change together with the target — derived from git commit co-occurrence in the workspace ontology.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_co_changes_with, mcp__oxagen__ontology_get_node
---

# /oxagen:cochange — symbols that change together

Wraps `code.co_changes_with` (M9, SPEC §7.2). Surfaces hidden
coupling that the call graph alone misses — files or symbols that
keep showing up in the same commits.

Useful before a refactor: even if A doesn't call B, if they always
ship together, changing A probably needs B updated too.

## Steps

### 1 — Resolve the symbol

Argument is `<file>:<line>` or a qualified name.

### 2 — Call `code.co_changes_with`

```
mcp__oxagen__code_co_changes_with({
  name: "<symbol>",
  limit: 25
})
```

Response is a SPEC §7.9 envelope with each row carrying
`co_change_count` and `last_co_changed_at`.

### 3 — Format

Sort desc by co-change count:

```
Co-changes with <symbol> (<N>)

  <count>  <file:line>  <symbol_name>   last seen <relative time>
  ...
```

### 4 — Suggest follow-ups

- "High co-change with `<X>` and no call-graph edge usually means
  shared intent — run `/oxagen:explain` on both before editing."
- "Run `/oxagen:recent` to see the current change frontier."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Repo may be young (few commits) or ingest is partial |
| `ontology_get_node` null | Run `/oxagen:search` first |
