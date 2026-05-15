---
description: List tests with high pass / fail churn — flake-rate ranking sourced from the workspace ontology.
argument-hint: "[--repo owner/name] [--days N]"
allowed-tools: mcp__oxagen__code_flaky_tests
---

# /oxagen:flaky-tests — flake-rate ranking

Wraps `code.flaky_tests` (M5). Returns tests that flip between
pass and fail in CI without a code change, ranked by flake rate.

## Steps

### 1 — Parse arguments

- `--repo owner/name` — required for multi-repo workspaces.
- `--days N` — rolling window (default 14).

### 2 — Call `code.flaky_tests`

```
mcp__oxagen__code_flaky_tests({
  repo_full_name: "<owner/name>",
  window_days: <n>,
  limit: 25
})
```

SPEC §7.9 envelope. Rows carry `test_qualified_name`, `file:line`,
`flake_rate`, `runs_in_window`, `failures_in_window`,
`last_failed_at`, `last_passed_at`.

### 3 — Format

Sort desc by `flake_rate`:

```
Flaky tests (last <N> days)

  <flake_pct>%  <file:line>  <test_qualified_name>
                <failures>/<runs> failed
                last failed <rel> · last passed <rel>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:tests-for <symbol>` to see all tests on the
  symbol the flake covers."
- "Run `/oxagen:pr-history <test_qualified_name>` to find the PR
  that introduced the flake."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Workspace may not have enough CI history yet |
