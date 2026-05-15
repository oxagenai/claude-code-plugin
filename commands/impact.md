---
description: Blast-radius report for a symbol — combines callers, dependencies, longest reach path, neighborhood, and active claims, ordered by PageRank.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_find_callers, mcp__oxagen__code_find_dependencies, mcp__oxagen__code_find_path, mcp__oxagen__code_get_neighborhood, mcp__oxagen__code_find_cycles, mcp__oxagen__code_find_dead_code, mcp__oxagen__ontology_pagerank, mcp__oxagen__ontology_get_node, mcp__oxagen__agent_list_active_claims, mcp__oxagen__agent_get_findings
---

# /oxagen:impact — blast-radius report

Answer "what does changing this symbol affect?" by combining the graph's
caller, dependency, path, and neighborhood views into one structured
report. Use it before any signature change, behaviour change, deletion, or
rename of a public symbol.

## Steps

### 1 — Resolve the symbol

The argument is either a qualified name (`org.module.Class.method`) or a
file:line locator (`packages/oxagen/oxagen/domains/billing/credits.py:142`).

Resolve to a graph node ID:

```
mcp__oxagen__ontology_get_node({
  identifier: "<argument>"  // server accepts either form
})
```

If the lookup returns no node, tell the user:

> No symbol matched `<argument>`. The graph may not have ingested the
> latest changes yet, or the symbol name doesn't appear in this workspace.
> Try `/oxagen:search "<symbol>"` to confirm the name.

If multiple nodes match (overloads, generics), list them and ask which one
to analyze. Don't pick blindly.

### 2 — Fan-out parallel queries

Once the symbol resolves to a `node_id`, fire these calls in parallel:

```
mcp__oxagen__code_find_callers({ node_id, depth: 1 })
mcp__oxagen__code_find_dependencies({ node_id, depth: 1 })
mcp__oxagen__code_get_neighborhood({ node_id, depth: 1 })
```

Then, if `code_find_callers` returned at least one caller, fire:

```
mcp__oxagen__ontology_pagerank({
  node_ids: [<symbol node_id>, <all caller node_ids>, <all dependency node_ids>]
})
```

Use the PageRank scores to order the report sections.

### 3 — Compute the longest reach path

Identify the highest-PageRank caller from step 2. Then call:

```
mcp__oxagen__code_find_path({
  from_node_id: "<highest-pagerank-caller-node-id>",
  to_node_id: "<symbol-node-id>",
  max_hops: 6
})
```

This shows the longest realistic chain reaching the symbol — useful for
understanding which entry points propagate changes.

### 4 — Format the report

```
Impact: <symbol qualified name>
Defined: <file:line>
Type: <node_type>
PageRank: <score>

## Direct callers (<N>, sorted by PageRank desc)

  <pr>  <file:line>  <caller_name>  (<caller_type>)
  ...

  +M more — see `<workspace_url>/explore/node/<id>` for full list.

## Direct dependencies (<N>)

  <file:line>  <dep_name>  (<dep_type>)
  ...

## Longest reach path (<hop_count> hops)

  (1) <entry_caller_name> — <file:line>
  (2) <intermediate> — <file:line>
  ...
  (n) <symbol_name> — <file:line>

## Same-module neighborhood (<N> siblings)

  <file:line>  <sibling_name>
  ...

## Recommendation

Editing `<symbol>` will require reviewing N direct callers and M
transitive callers. Run `/oxagen:search "tests for <symbol>"` to find
existing test coverage before changing behaviour.
```

Cap each section at the top 10 entries; show `+N more` for overflow.

### 5 — Check active claims and prior findings

Before declaring the report complete, check whether another agent is
actively working in this code:

```
mcp__oxagen__agent_list_active_claims({ node_id })
mcp__oxagen__agent_get_findings({ node_id, limit: 10 })
```

If `list_active_claims` returns a claim from a different session,
prepend an "Active claims" section to the report:

```
## Active claims

  <session_label>  claimed <file:line-range>  <relative time>
  ...

Another agent is in this code. Coordinate before editing — see the
`using-oxagen` skill's agent-coordination guidance.
```

If `get_findings` returns findings, append them under "Recent findings":

```
## Recent findings (most recent first)

  <session_label> <when>: <one-line summary>
  ...
```

Findings from prior sessions often save the next session a round of
investigation.

### 6 — Surface red flags

If any of the following are true, add a "Warnings" block above the
recommendation:

- Caller count > 20 — high blast radius; recommend a deprecation cycle
- Symbol is in a `code.find_cycles` cluster — circular dep; warn the user
- Symbol is in `code.find_dead_code` results — looks unused; confirm before deleting
- Any caller is a `Test` node and the symbol's behaviour is changing — flag tests that will need updating
- An active claim by a different session overlaps the symbol's source range — block-level warning, coordinate with the other session or wait for the claim to expire before running `/oxagen:claim`

## Failure modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| `ontology_get_node` returns null | Symbol not yet ingested or misspelled | Run `/oxagen:search` to find the right name |
| `find_callers` returns empty for a known-public symbol | Ingest didn't complete; or caller modules excluded | Re-run setup or check `app.oxagen.ai/workspaces/.../connections` |
| `find_path` returns no path | Highest-PageRank caller doesn't actually reach this symbol | Pick the second-highest and retry, or skip the path section |
| `ontology_pagerank` errors | PageRank job not yet computed for this workspace | Fall back to alphabetical ordering and surface a note |

## Performance notes

The four parallel calls plus the path call total ~5 round-trips. Server
caches PageRank and graph reads, so repeat calls on the same symbol
within a session return quickly. For symbols with thousands of callers,
the server pages — request `depth: 1` for the first pass and only deepen
on follow-up.

## When the user wants a different cut

- "Just callers" → run `/oxagen:impact` and read the Direct Callers section
- "Just dependencies" → same; read the Dependencies section
- "Across multiple symbols" → run `/oxagen:impact` per symbol; aggregate
  the unique caller list manually
- "Visualize" → ask the user; if yes, follow up with the appropriate
  `viz.*` tool (see the `using-oxagen` skill for guidance)
