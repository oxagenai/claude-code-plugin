---
description: Resolve a name to one or more symbols in the workspace ontology — fuzzy / structural lookup with file:line anchors.
argument-hint: "<name>"
allowed-tools: mcp__oxagen__code_find_symbol
---

# /oxagen:find-symbol — locate a symbol by name

Wraps `code.find_symbol` (M9, SPEC §7.1). Resolves a partial or
ambiguous name into the actual ontology nodes (functions, classes,
methods) that match. Use this before any other `/oxagen:*` command
that needs a precise symbol.

## Steps

### 1 — Take the name verbatim

The argument is the partial or qualified symbol name. Don't
rephrase.

### 2 — Call `code.find_symbol`

```
mcp__oxagen__code_find_symbol({
  name: "<argument>",
  limit: 25
})
```

SPEC §7.9 envelope. Rows carry `name`, `qualified_name`, `kind`
(`code.function`, `code.class`, etc.), and `file:line`.

### 3 — Format

```
Symbols matching "<name>" (<N>)

  <kind>  <qualified_name>  — <file:line>
  ...
```

If only one match: surface it and suggest a follow-up command. If
multiple: list and ask which one to act on.

### 4 — Suggest follow-ups

- "Run `/oxagen:explain <qualified_name>` for behaviour."
- "Run `/oxagen:impact <qualified_name>` for blast radius."
- "Run `/oxagen:read <qualified_name>` to fetch source."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Refine the query, or trigger an ingest refresh via `/oxagen:setup` |
| Wrong kind matched | Pass a more qualified name (e.g. `module.Class.method`) |
