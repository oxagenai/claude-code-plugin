---
description: Show the most-recent CI workflow run for a workflow / branch / PR — pulled from the workspace ontology.
argument-hint: "[workflow] [--branch <name>] [--pr <num>] [--repo owner/name]"
allowed-tools: mcp__oxagen__code_last_run
---

# /oxagen:last-run — most-recent CI run

Wraps `code.last_run` (M4). Returns the latest GitHub Actions run
matching a workflow / branch / PR filter, without hitting the
GitHub API yourself.

At least one of `<workflow>`, `--branch`, `--pr` must be supplied.

## Steps

### 1 — Parse arguments

- `<workflow>` (positional, optional) — substring match on the
  workflow name (e.g. `pnpm-gate`, `Deploy`).
- `--branch <name>` — exact match on `head_branch`.
- `--pr <num>` — match runs that exercised this PR.
- `--repo owner/name` — required for multi-repo workspaces.

### 2 — Call `code.last_run`

```
mcp__oxagen__code_last_run({
  repo_full_name: "<owner/name>",
  workflow_name: "<substring>",
  branch: "<branch>",
  pr_number: <num>
})
```

SPEC §7.9 envelope. Top-level fields:
- `workflow_run_id`, `display_name`, `status`, `conclusion`
- `head_branch`, `head_sha`, `linked_pr_numbers`
- `started_at`, `completed_at`, `duration_seconds`

### 3 — Format

```
Last run: <display_name>
Status: <status> / <conclusion>
Branch: <head_branch>  ·  SHA: <head_sha[:7]>
PRs: <pr_numbers>

Started:   <relative-time>
Completed: <relative-time>  (<duration>)

Run ID: <workflow_run_id>
```

### 4 — Suggest follow-ups

- "Run `/oxagen:run-failures <run_id>` if the run was red."
- "Run `/oxagen:failing-tests --branch <branch>` to see the
  test-level breakdown."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `at least one of workflow/branch/pr_number is required` | Pass any of those filters |
| `no run found` | Filter too narrow, or workflow has no runs in the ingest |
