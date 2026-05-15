---
description: Show failed jobs and steps for a CI workflow run — with cold-log preview / pointer per failed step.
argument-hint: "<workflow-run-id> [--include-log-blobs]"
allowed-tools: mcp__oxagen__code_run_failures
---

# /oxagen:run-failures — failed jobs and steps for a run

Wraps `code.run_failures` (M4). For a given workflow-run UUID,
returns each failed job and its failed steps with the
SPEC §10 cold-log preview / pointer attached.

## Steps

### 1 — Parse arguments

- `<workflow-run-id>` (required) — UUID from
  `/oxagen:last-run` or the workspace ontology.
- `--include-log-blobs` — inline the cold log blobs for each failed
  step. Defaults off (the 200-char preview is usually enough).

### 2 — Call `code.run_failures`

```
mcp__oxagen__code_run_failures({
  workflow_run_id: "<uuid>",
  include_log_blobs: <bool>
})
```

SPEC §7.9 envelope. Rows are nested by job:
- `job_name`, `conclusion`, `started_at`, `completed_at`
- `failed_steps[]` — `name`, `number`, `conclusion`, `log_preview`,
  `log_pointer` (or `log_blob` if `include_log_blobs=true`)

### 3 — Format

```
Run failures: <run_id>

## <job_name>  (<conclusion>)
  Step <num>: <step_name>
    <log_preview>
    cold-log pointer: <log_pointer>
  ...
```

If `include_log_blobs=true`, render the full blob inline indented
under the step.

### 4 — Suggest follow-ups

- "Run `/oxagen:flaky-tests` to see if any of these are known
  flakes."
- "Run `/oxagen:pr <num>` for any linked PR."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `workflow_run_id not found` | Resolve via `/oxagen:last-run` first |
| All `log_preview` empty | CI didn't capture stdout — turn on log retention in the workflow |
