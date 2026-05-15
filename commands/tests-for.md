---
description: List the tests that exercise a symbol — derived from the workspace ontology's TESTS edges.
argument-hint: "<file:line | qualified-name>"
allowed-tools: mcp__oxagen__code_tests_for, mcp__oxagen__ontology_get_node
---

# /oxagen:tests-for — tests covering a symbol

Wraps `code.tests_for` (M5). Returns the test functions that
directly or transitively exercise the target symbol.

## Steps

### 1 — Resolve the symbol

Argument is `<file>:<line>` or qualified name.

### 2 — Call `code.tests_for`

```
mcp__oxagen__code_tests_for({
  name: "<symbol>",
  limit: 50
})
```

SPEC §7.9 envelope. Rows carry `test_qualified_name`, `file:line`,
`test_kind` (`unit` / `integration` / `e2e`), and
`coverage_kind` (`direct` / `transitive`).

### 3 — Format

Group by `coverage_kind`:

```
Tests for <symbol> (<N>)

## Direct (<n>)
  <file:line>  <test_qualified_name>  (<test_kind>)
  ...

## Transitive (<n>)
  <file:line>  <test_qualified_name>  (<test_kind>)
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:coverage <file>` for line-level coverage."
- "Run `/oxagen:failing-tests` to see what's currently red."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | No tests cover this symbol or test ingest is incomplete |
| `ontology_get_node` null | Run `/oxagen:find-symbol` |
