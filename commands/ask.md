---
description: Ask a natural-language question about the workspace ontology and get a direct answer backed by Cypher — faster than `/oxagen:search` for factual lookups.
argument-hint: "<question>"
allowed-tools: mcp__oxagen__ontology_ask
---

# /oxagen:ask — direct ontology Q&A

Answers a free-form question about the workspace knowledge graph using
`ontology.ask`. Unlike `/oxagen:search` (which returns a list of matching
nodes), this command returns a prose answer grounded in Cypher results.
Use it for factual questions: "How many nodes?", "What calls X?", "Which
modules are most connected?"

## Steps

### 1 — Take the question verbatim

The slash-command argument is the question. Pass it through without
paraphrasing.

If empty, ask: "What would you like to know about this workspace? Examples:
'How many function nodes are there?', 'What is the most-called module?',
'Which files have no test coverage in the graph?'"

### 2 — Call `ontology.ask`

```
mcp__oxagen__ontology_ask({
  question: "<user's question>",
  limit: 50
})
```

Response fields:
- `answer` — prose answer synthesized from Cypher results
- `cypher` — the generated Cypher query
- `rows` — raw result rows (for power users)
- `confidence` — float 0–1

### 3 — Surface the answer

Print `answer` directly. Then append a collapsed block:

```
Generated Cypher (confidence: <conf>):

  <cypher>
```

If `confidence < 0.4`, add a caveat:

> Low confidence — the generated Cypher may not match your question
> exactly. Try rephrasing or use `/oxagen:search` to browse nodes
> directly.

### 4 — Offer follow-ups

- If the answer mentions a specific symbol: "Run `/oxagen:impact <symbol>`
  to see its blast radius."
- If the answer is a count: "Run `/oxagen:search` with a filter to list
  the individual nodes."
- If the answer involves a module: "Run `/oxagen:explain <function>` to
  get a full behavioural description of a specific function in that module."

## Good question patterns

| Question | What the server does |
|----------|---------------------|
| "How many nodes are in this workspace?" | `MATCH (n) RETURN count(n)` |
| "What functions call the billing module?" | Traverses CALLS edges to billing nodes |
| "Which modules have no tests?" | Anti-join test coverage edges |
| "What are the top 5 highest-PageRank nodes?" | Reads precomputed PageRank scores |

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty `answer` with 0 rows | Rephrase; ontology may not have those nodes yet |
| `confidence` always < 0.3 | The schema may be sparse; trigger an ingest refresh via `/oxagen:setup` |
| Tool errors | Check MCP connection with `/mcp`; verify `OXAGEN_MCP_TOKEN` is set |
