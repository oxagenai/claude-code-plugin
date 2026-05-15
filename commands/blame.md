---
description: Enriched blame for a file — line-level authorship plus the PRs and issues that touched each block.
argument-hint: "<file> [--from-line N] [--to-line N]"
allowed-tools: mcp__oxagen__code_blame_enriched
---

# /oxagen:blame — enriched blame for a file

Wraps `code.blame_enriched` (M3). Goes beyond `git blame` by
joining each line back to the PR that landed it, the issue that
prompted it, and the reviewer who signed off.

## Steps

### 1 — Parse arguments

- `<file>` (required) — repo-relative path
- `--from-line N` / `--to-line N` (optional) — line range

### 2 — Call `code.blame_enriched`

```
mcp__oxagen__code_blame_enriched({
  path: "<file>",
  from_line: <n>,   // omit for whole file
  to_line:   <n>
})
```

SPEC §7.9 envelope. Rows carry `line_start`, `line_end`, `author`,
`committed_at`, `commit_sha`, `pr_number`, `issue_keys`,
`reviewer_logins`.

### 3 — Format

Group by contiguous block:

```
Blame: <file>  lines <from>-<to>

  L<a>-<b>  <author>  <relative-time>  <sha[:7]>
            PR #<num>: <pr-title>
            Reviewers: <login>, <login>
            Issues: <key>, <key>

  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:pr <num>` to see the merged PR view."
- "Run `/oxagen:who-knows <symbol>` to see expertise for a specific
  function in this file."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | File may not be tracked in git, or ingest is incomplete |
| PR / issue columns blank | GitHub connection not ingested yet — check `app.oxagen.ai` |
