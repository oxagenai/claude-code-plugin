---
description: Search issues across the workspace by free-text — returns matching Linear / GitHub issues with state, labels, and linked PRs.
argument-hint: "<query>"
allowed-tools: mcp__oxagen__code_find_issues
---

# /oxagen:find-issues — search issues in the ontology

Wraps `code.find_issues` (M9, SPEC §7.6). Hybrid full-text search
across the issue corpus the workspace has ingested.

## Steps

### 1 — Take the query verbatim

Pass it through. Empty argument → ask the user for a query.

### 2 — Call `code.find_issues`

```
mcp__oxagen__code_find_issues({
  query: "<query>",
  limit: 25
})
```

SPEC §7.9 envelope. Each row has `key`, `title`, `state`, `labels`,
`updated_at`, and a `score`.

### 3 — Format

```
Issues matching "<query>" (<N>)

  <state>  <key>  <title>   <relative-time>
           labels: <l1>, <l2>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:issue <key>` for full context on a specific issue."
- "Run `/oxagen:pr <num>` for a linked PR."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Try a broader query; check that the issue source is connected |
