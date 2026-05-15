---
description: List the most recent code changes in the workspace ontology — commits, files touched, and authors over a configurable window.
argument-hint: "[--days N] [--path <path>]"
allowed-tools: mcp__oxagen__code_recent_changes
---

# /oxagen:recent — recent changes in the workspace

Wraps `code.recent_changes` (M3). Shows the latest commits the
ontology has ingested, optionally filtered by repo path.

## Steps

### 1 — Parse arguments

- `--days N` — window size (default 7, max 90)
- `--path <path>` — repo-relative path filter (optional)

### 2 — Call `code.recent_changes`

```
mcp__oxagen__code_recent_changes({
  days: <n>,
  path: "<path-or-null>",
  limit: 50
})
```

SPEC §7.9 envelope. Each row carries `commit_sha`, `author`,
`committed_at`, `files_touched`, and a one-line subject.

### 3 — Format

```
Recent changes (last <N> days, <path or "all paths">)

  <relative-time>  <author>  <sha[:7]>  <subject>
                              files: <count>
  ...
```

Cap at 25 rows; show `+M more` for overflow.

### 4 — Suggest follow-ups

- "Run `/oxagen:cochange <symbol>` if you see two unrelated files
  keep changing together."
- "Run `/oxagen:blame <file>` on a specific file from the list."
- "Run `/oxagen:pr <number>` for the merged-PR view."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Path filter may not match any ingested commits — drop the filter |
| Window too narrow | Bump `--days` (max 90) |
