---
description: Line-level coverage for a file — covered, uncovered, and partial lines from the latest CI run in the workspace ontology.
argument-hint: "<file>"
allowed-tools: mcp__oxagen__code_coverage_for
---

# /oxagen:coverage — line coverage for a file

Wraps `code.coverage_for` (M5). Returns per-line coverage status
for the file from the most-recent CI run that produced coverage
artefacts.

## Steps

### 1 — Take the file path

Argument is the repo-relative path, e.g.
`packages/oxagen/oxagen/domains/billing/service.py`.

### 2 — Call `code.coverage_for`

```
mcp__oxagen__code_coverage_for({
  path: "<file>"
})
```

SPEC §7.9 envelope. Top-level fields:
- `path`, `total_lines`, `covered_lines`, `partial_lines`,
  `uncovered_lines`
- `pct_covered`
- `uncovered_blocks` — contiguous ranges of uncovered lines
- `source_run_id` — the workflow run id that generated the data
- `recorded_at`

### 3 — Format

```
Coverage: <file>
Source run: <run_id>  (<relative-time>)

Total: <covered>/<total> lines covered  (<pct>%)
  covered:    <n>
  partial:    <n>
  uncovered:  <n>

Uncovered blocks
  L<a>-<b>  (<n> lines)
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:tests-for <symbol>` for any uncovered function."
- "Run `/oxagen:failing-tests` if the run is red."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `no coverage data for path` | CI run didn't emit lcov / cobertura for this file — check workflow config |
| `recorded_at` very old | Coverage hasn't been refreshed — kick a CI run on default |
