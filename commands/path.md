---
description: Find the shortest call-graph path between two symbols. Wraps `code.find_path` (alias `code.dependency_path`).
argument-hint: "<from-symbol> <to-symbol> [--max-hops N]"
allowed-tools: mcp__oxagen__code_find_path, mcp__oxagen__ontology_get_node
---

# /oxagen:path — directed path between two symbols

Answers "how does A reach B?" by walking CALLS / IMPORTS edges in
the workspace ontology. Useful for tracing how an entry point
reaches a specific target, or for confirming a refactor preserves a
required reachability.

## Steps

### 1 — Parse arguments

Two positional arguments, each a `<file:line>` or qualified name.
Optional `--max-hops N` (default 6, range 1–10).

### 2 — Resolve both symbols (if name-based)

```
mcp__oxagen__ontology_get_node({ identifier: "<from>" })
mcp__oxagen__ontology_get_node({ identifier: "<to>"   })
```

### 3 — Call `code.find_path`

```
mcp__oxagen__code_find_path({
  from_node_id: "<from-id>",
  to_node_id:   "<to-id>",
  max_hops: <n>
})
```

### 4 — Format

```
Path: <from> → <to>  (<hop_count> hops)

  (1) <name>  — <file:line>
  (2) <name>  — <file:line>
  ...
  (n) <name>  — <file:line>
```

If the response includes alternate paths, show the shortest first
and mention the count of additional paths.

If no path is found:

> No call-graph path from `<from>` to `<to>` within `<max-hops>`
> hops. The two symbols may live in disjoint subgraphs, or the
> ingest may be incomplete.

## Failure modes

| Symptom | Fix |
|---------|-----|
| No path found | Increase `--max-hops`, or check for missing ingest |
| `ontology_get_node` null | Run `/oxagen:search` to find the right name |
