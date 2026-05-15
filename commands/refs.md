---
description: Find every reference to a symbol — call sites, imports, type uses, and string mentions tied to the workspace ontology.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_references_to, mcp__oxagen__ontology_get_node
---

# /oxagen:refs — all references to a symbol

Wraps `code.references_to` (M9, SPEC §7.2). Broader than
`/oxagen:callers`: returns CALL edges plus IMPORT, USES_TYPE, and
documentation references.

## Steps

### 1 — Resolve the symbol

Argument is `<file>:<line>` or a qualified name.

### 2 — Call `code.references_to`

```
mcp__oxagen__code_references_to({
  name: "<symbol>",
  limit: 100
})
```

Returns the SPEC §7.9 envelope. Each `results` row has the source
file:line and the edge kind.

### 3 — Format

Group by edge kind so the output is scannable:

```
References to <symbol> (<N>)

## Calls (<n>)
  <file:line>  <caller_name>
  ...

## Imports (<n>)
  <file:line>  <importer_name>
  ...

## Type uses (<n>)
  <file:line>  <user_name>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:impact <symbol>` for the full blast-radius report."
- "Run `/oxagen:callers <symbol>` to filter to CALL edges only."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Symbol may be unused or not yet ingested |
| `ontology_get_node` null | Run `/oxagen:search` to confirm the name |
