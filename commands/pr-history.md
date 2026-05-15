---
description: List the PRs that have touched a symbol — chronological history from the workspace ontology.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_pr_history, mcp__oxagen__ontology_get_node
---

# /oxagen:pr-history — PRs that touched a symbol

Wraps `code.pr_history` (M9, SPEC §7.3). Returns the merged PRs
that modified the symbol, newest first.

## Steps

### 1 — Resolve the symbol

Argument is `<file>:<line>` or qualified name.

### 2 — Call `code.pr_history`

```
mcp__oxagen__code_pr_history({
  name: "<symbol>",
  limit: 20
})
```

SPEC §7.9 envelope. Rows carry `pr_number`, `title`, `merged_at`,
`author`, and `additions` / `deletions` against the symbol's lines.

### 3 — Format

```
PR history for <symbol> (<N>)

  <relative-time>  PR #<num>  +<a>/-<d>  <author>  <title>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:pr <num>` for the full PR context."
- "Run `/oxagen:who-knows <symbol>` to see who reviewed those PRs."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | GitHub connection not ingested or symbol is brand new |
| `ontology_get_node` null | Pass `<file>:<line>` instead |
