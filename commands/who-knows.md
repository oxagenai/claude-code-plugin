---
description: Identify the people most likely to know about a symbol — git history + review activity scored across the workspace ontology.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_who_knows_about, mcp__oxagen__ontology_get_node
---

# /oxagen:who-knows — find the experts on a symbol

Wraps `code.who_knows_about` (M9, SPEC §7.3). Returns the people
ranked by authorship + review activity on the symbol — useful for
"who do I tag on this PR?" or "who can sanity-check this refactor?"

## Steps

### 1 — Resolve the symbol

Argument is `<file>:<line>` or a qualified name.

### 2 — Call `code.who_knows_about`

```
mcp__oxagen__code_who_knows_about({
  name: "<symbol>",
  limit: 10
})
```

SPEC §7.9 envelope. Each row carries `commits_authored`,
`reviews_given`, `last_touched_at`, and a composite `score`.

### 3 — Format

```
Experts on <symbol> (<N>)

  <score>  <login>  — <commits> commits, <reviews> reviews,
                     last touched <relative time>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:expert <login>` to see what else this person owns."
- "Run `/oxagen:pr-history <symbol>` to see recent PRs touching it."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | GitHub connection may not be ingested yet — check `app.oxagen.ai/workspaces/.../connections` |
| `ontology_get_node` null | Pass `<file>:<line>` instead |
