---
description: Natural-language search of the workspace ontology — runs NL→Cypher via `ontology.ask`, falls back to graph traversal, returns top hits with file:line.
argument-hint: "<natural-language-query>"
allowed-tools: mcp__oxagen__ontology_ask, mcp__oxagen__ontology_traverse
---

# /oxagen:search — natural-language graph search

Translate a user query into a Cypher query against the workspace ontology
and return the top hits. Use this for any "find X" question that the typed
`code.*` and `ontology.list_*` tools don't cover directly.

## Steps

### 1 — Take the query verbatim

The slash-command argument is the natural-language query. Pass it through
without rewriting unless it's empty.

If the argument is empty, ask: "What do you want to search for? Examples:
'all functions that touch the billing module', 'classes implementing
PaymentProvider', 'tests for the auth flow'."

### 2 — Call `ontology.ask`

```
mcp__oxagen__ontology_ask({
  question: "<user's natural-language query>",
  limit: 25
})
```

The MCP server handles NL→Cypher translation. The response includes:
- `cypher` — the generated Cypher query (echo back to the user for transparency)
- `rows` — result rows
- `confidence` — float 0–1; values < 0.5 indicate the LLM was uncertain
- `executed_at` — server timestamp

### 3 — Inspect confidence and row count

Trigger the graph traversal fallback if either:
- `confidence < 0.5`
- `rows.length === 0`

Otherwise format and return the results from step 2.

### 4 — Graph traversal fallback (when needed)

```
mcp__oxagen__ontology_traverse({
  query: "<user's query, stripped of stopwords>",
  limit: 25
})
```

Combine results: `ontology.ask` rows first (if any), traversal hits below.
Tag each result with its source so the user knows where it came from.

### 5 — Format output

For each result row, render:

```
<file>:<line>  <symbol>  (<node_type>)
  <one-line context — the surrounding line or docstring snippet>
```

Show top 10 hits. If more exist, append:

> +N more results. Refine your query or run `/oxagen:search` with a more
> specific phrase.

Order:
- If `cypher.run` returned rows with a `score` or `pagerank` property, sort
  desc by that
- Otherwise preserve the server's row order

### 6 — Show the generated Cypher

Append a collapsed-by-default block with the Cypher the server generated:

```
Generated Cypher (confidence: <conf>):

  <cypher query as returned by the server>
```

This is for transparency — Cypher-fluent users will refine the query and
re-run.

### 7 — Suggest follow-ups

After the results, suggest one of:

- For a single high-relevance hit: "Run `/oxagen:impact <symbol>` to see who calls it."
- For a list of hits across multiple modules: "Run `/oxagen:impact <module>` to see cross-module dependencies."
- For empty results: "Try broader terms or remove module-specific qualifiers."

## Query patterns that work well

| User asks | Server's Cypher tends to produce |
|-----------|---|
| "all functions that touch billing" | `MATCH (f:Function)-[:CALLS*1..2]->(:Function {module:'billing'}) RETURN f` |
| "classes implementing PaymentProvider" | `MATCH (c:Class)-[:IMPLEMENTS]->(:Interface {name:'PaymentProvider'}) RETURN c` |
| "tests for auth flow" | `MATCH (t:Test)-[:TESTS]->(:Function)-[:IN_MODULE]->(:Module {name:'auth'}) RETURN t` |
| "dead code in connectors" | `MATCH (f:Function)-[:IN_MODULE]->(:Module {name:'connectors'}) WHERE NOT (:Function)-[:CALLS]->(f) RETURN f` |

If the user's phrasing is ambiguous (e.g. "show me payments"), surface the
generated Cypher and ask whether to refine.

## Output discipline

- Always cite `file:line`. The graph stores it.
- Quote actual symbol names from the graph, not paraphrases.
- Don't invent properties the server didn't return.
- If `cypher.run` errors, surface the error message — do not silently
  fall back. Most errors mean the schema doesn't match the query
  ("unknown label 'Foo'") and the user needs to adjust.

## When to escalate to `/oxagen:impact`

Search answers "where is X?". Impact answers "what depends on X?". If the
user's intent is the latter, recommend they run `/oxagen:impact <symbol>`
on the specific symbol they care about.
