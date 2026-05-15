---
description: Show the directory tree under a path with per-node symbol counts pulled from the workspace ontology.
argument-hint: "[path]"
allowed-tools: mcp__oxagen__code_module_tree
---

# /oxagen:module-tree — annotated directory tree

Wraps `code.module_tree` (M9, SPEC §7.1). Returns the directory
tree under a path, annotated with the count of functions, classes,
and tests in each subtree. Cheaper than walking the file system —
the ontology already has the structure indexed.

## Steps

### 1 — Parse argument

`<path>` (optional) — repo-relative path. Omit for the repo root.

### 2 — Call `code.module_tree`

```
mcp__oxagen__code_module_tree({
  path: "<path-or-empty>",
  max_depth: 3
})
```

SPEC §7.9 envelope. Each row carries `path`, `kind` (`dir` or
`file`), `function_count`, `class_count`, `test_count`,
`line_count`.

### 3 — Format

```
Module tree: <path or "/">

  <path>/                  <fns>f / <classes>c / <tests>t
    <subpath>/             <fns>f / <classes>c / <tests>t
      <file>               <fns>f / <classes>c / <tests>t
  ...
```

Indent by depth. Cap output at ~50 rows; if more, show top-level
only and tell the user to pass a deeper path.

### 4 — Suggest follow-ups

- "Run `/oxagen:module-tree <path>` to drill into any subdir."
- "Run `/oxagen:find-symbol <name>` to find a specific function."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `path not found` | Path doesn't exist in the ingested tree — check `/oxagen:repo-overview` |
| Counts all zero | Ingest may be incomplete |
