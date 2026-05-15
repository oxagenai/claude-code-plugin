---
description: Structural metadata for a symbol — kind, signature, decorators, docstring, and location — without the full behavioural breakdown that `/oxagen:explain` produces.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_describe_symbol
---

# /oxagen:describe — structural metadata for a symbol

Wraps `code.describe_symbol` (M9, SPEC §7.1). Returns kind
(function / class / method), full qualified name, signature,
decorators, return type, docstring, and source range — without
running the heavier `ontology.explain_function` traversal.

Use this when you just need to confirm a signature or read a
docstring; reach for `/oxagen:explain` for the full call / read /
write breakdown.

## Steps

### 1 — Call `code.describe_symbol`

```
mcp__oxagen__code_describe_symbol({
  name: "<argument>"
})
```

SPEC §7.9 envelope. Top-level fields:
- `kind`, `qualified_name`, `signature`, `decorators`
- `return_type`, `docstring`
- `defined_at` (`file`, `line_start`, `line_end`)

### 2 — Format

```
<qualified_name>  (<kind>)
Defined: <file>:<line_start>-<line_end>

Signature: <signature>
Decorators: <d1>, <d2>
Returns: <return_type>

<docstring>
```

### 3 — Suggest follow-ups

- "Run `/oxagen:explain <symbol>` for behavioural breakdown."
- "Run `/oxagen:read <symbol>` to fetch source."
- "Run `/oxagen:impact <symbol>` for blast radius."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty / null docstring | Source had no docstring — fall back to `/oxagen:read` |
| Symbol not found | Run `/oxagen:find-symbol` first |
