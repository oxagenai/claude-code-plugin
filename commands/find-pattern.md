---
description: Search the workspace ontology for a code pattern — hybrid semantic + grep, returns file:line hits ranked by relevance.
argument-hint: "<pattern-or-phrase>"
allowed-tools: mcp__oxagen__code_find_pattern
---

# /oxagen:find-pattern — pattern search across the workspace

Wraps `code.find_pattern` (M9, SPEC §7.1). Hybrid search that
combines semantic similarity with structural grep against the
ontology. Returns ranked hits with file:line anchors.

Use this for:
- "Find every place we catch and re-raise"
- "Where do we set retry timeouts?"
- "Show me all usages of this idiom"

## Steps

### 1 — Take the pattern verbatim

Don't rephrase. The argument is what gets sent.

### 2 — Call `code.find_pattern`

```
mcp__oxagen__code_find_pattern({
  query: "<pattern>",
  limit: 25
})
```

SPEC §7.9 envelope. Rows carry `file:line`, `snippet` (one-line
context), and `score`.

### 3 — Format

```
Pattern hits for "<pattern>" (<N>)

  <score>  <file:line>  <snippet>
  ...
```

Top 15; surface `+M more` for overflow. Show
`counterfactual_method` ("`hybrid`" expected) so the agent records
the cost-saving claim.

### 4 — Suggest follow-ups

- "Run `/oxagen:explain <symbol>` on any hit you want to dive into."
- "Run `/oxagen:read <symbol>` to fetch the surrounding source."

## Failure modes

| Symptom | Fix |
|---------|-----|
| No hits | Pattern too narrow — try broader phrasing |
| Low scores | Pattern semantically diffuse — pass an exact substring instead |
