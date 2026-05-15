---
description: Summarize the workspace's repos — language mix, top packages, file count, and entry points pulled from the ontology.
argument-hint: ""
allowed-tools: mcp__oxagen__code_repo_overview
---

# /oxagen:repo-overview — high-level repo summary

Wraps `code.repo_overview` (M9, SPEC §7.1). Returns a structural
overview of every repo connected to the workspace — language
breakdown, top-level modules, file counts, and detected entry
points (CLIs, server bindings, lambdas).

Useful as the first call in a brand-new workspace, or when the
user asks "what's in this codebase?"

## Steps

### 1 — Call the tool

```
mcp__oxagen__code_repo_overview({})
```

SPEC §7.9 envelope. Top-level fields per repo:
- `repo_full_name`, `default_branch`, `last_ingested_at`
- `language_mix` ([{name, lines, pct}, ...])
- `top_modules` ([{path, file_count, line_count}, ...])
- `entry_points` ([{kind, path, name}, ...])
- `node_counts` ({class, function, ...})

### 2 — Format

```
Workspace overview

## <repo_full_name>  (default: <branch>)
Last ingested: <relative-time>

Language mix
  <lang>  <lines>  <pct>%
  ...

Top modules (by file count)
  <path>  <files> files / <lines> lines
  ...

Entry points (<n>)
  <kind>: <path>  <name>
  ...

Graph nodes
  function: <n>, class: <n>, ...
```

If multiple repos: render one block per repo.

### 3 — Suggest follow-ups

- "Run `/oxagen:module-tree <path>` for the directory tree under
  any top module."
- "Run `/oxagen:find-symbol <name>` to drill into a specific
  symbol."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty `language_mix` / `entry_points` | Re-ingest in `app.oxagen.ai/workspaces/.../connections` |
