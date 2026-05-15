---
description: Fetch the source bytes for a symbol — defined-range plus configurable surrounding context.
argument-hint: "<file:line | qualified-name> [--context-lines N]"
allowed-tools: mcp__oxagen__code_read_symbol
---

# /oxagen:read — read symbol source

Wraps `code.read_symbol` (M9, SPEC §7.1). Returns the symbol's
summary, exact defined line range, and source bytes plus N lines
of surrounding context (PR #608, merged 2026-05-06, commit
`5dedb28b`).

## Steps

### 1 — Parse arguments

- `<argument>` — `<file>:<line>` or qualified name.
- `--context-lines N` — extra lines before / after the symbol.
  Default 5, range 0–200. Larger values pull more bytes; set to 0
  for an exact-range fetch.

### 2 — Call `code.read_symbol`

```
mcp__oxagen__code_read_symbol({
  name: "<argument>",
  context_lines: <n>
})
```

SPEC §7.9 envelope. Top-level fields:
- `qualified_name`, `kind`
- `defined_at` (`file`, `line_start`, `line_end`)
- `summary`
- `source` — file bytes for the requested range, including the
  `context_lines` padding. Available on servers running PR #608
  (merged 2026-05-06, commit `5dedb28b`); absent on older
  servers, in which case fall back to `/oxagen:describe` +
  `/oxagen:explain`.

### 3 — Format

```
<qualified_name>  (<kind>)
Defined: <file>:<line_start>-<line_end>  (+/- <context_lines>)

<summary>

```<lang from extension>
<source>
```
```

If `source` is absent, render `summary` + `defined_at` and tell
the user to upgrade their MCP server to read source bytes.

### 4 — Suggest follow-ups

- "Run `/oxagen:explain <symbol>` for behavioural breakdown."
- "Run `/oxagen:impact <symbol>` for blast radius."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `source` field missing | Server pre-dates PR #608 — fetch via `/oxagen:describe` and IDE read |
| `defined_at` missing | Symbol not ingested — run `/oxagen:find-symbol` |
| `context_lines` rejected | Server < the OXA-PENDING upgrade — drop the flag |
