---
description: List currently failing tests across the workspace — joined to commit, PR, and symbol nodes for quick triage.
argument-hint: "[--repo owner/name] [--branch <name>]"
allowed-tools: mcp__oxagen__code_failing_tests
---

# /oxagen:failing-tests — currently failing tests

Wraps `code.failing_tests` (M5). Returns the tests that failed in
the most-recent CI runs the workspace has ingested, with each
failure linked back to commit, PR, and symbol nodes.

## Steps

### 1 — Parse arguments

- `--repo owner/name` — optional, required when workspace has
  multiple repos.
- `--branch <name>` — defaults to the repo's default branch.

### 2 — Call `code.failing_tests`

```
mcp__oxagen__code_failing_tests({
  repo_full_name: "<owner/name>",   // omit for single-repo workspace
  branch: "<branch>",
  limit: 50
})
```

SPEC §7.9 envelope. Each row carries `test_qualified_name`,
`file:line`, `failure_message`, `failed_in_run_id`, `commit_sha`,
`first_failed_at`, `consecutive_failures`.

### 3 — Format

```
Failing tests on <branch> (<N>)

  <file:line>  <test_qualified_name>
              consecutive failures: <n>  since <relative-time>
              run: <run_id>  sha: <sha[:7]>
              <failure_message trimmed to 200 chars>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:run-failures <run_id>` for the full failure block."
- "Run `/oxagen:flaky-tests` if a test keeps flipping state."
- "Run `/oxagen:tests-for <symbol>` to see all tests on a symbol."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | All tests passing, or CI runs not ingested yet |
| `multi-repo` error | Pass `--repo` |
