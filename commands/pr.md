---
description: Show the full PR context — title, body, commits, reviewers, files, and linked issues — joined to the workspace ontology.
argument-hint: "<pr-number> [--repo owner/name]"
allowed-tools: mcp__oxagen__code_pr_context
---

# /oxagen:pr — PR context from the ontology

Wraps `code.pr_context` (M3). One call returns the PR's title,
body, commits, reviewers, files touched, issue links, and the
graph nodes affected — without cloning the PR or hitting the
GitHub API yourself.

## Steps

### 1 — Parse arguments

- `<pr-number>` (required) — integer PR number
- `--repo owner/name` (optional) — required when the workspace has
  more than one ingested repo; omit for single-repo workspaces.

### 2 — Call `code.pr_context`

```
mcp__oxagen__code_pr_context({
  pr_number: <n>,
  repo_full_name: "<owner/name>"   // omit if workspace has one repo
})
```

SPEC §7.9 envelope. Top-level `results` shape:
- `title`, `body`, `state`, `merged_at`, `author`
- `files` (path, additions, deletions, status)
- `commits` (sha, subject, author, committed_at)
- `reviewers` (login, state, submitted_at)
- `linked_issues` (key, title)
- `affected_nodes` (graph node IDs touched by this PR)

### 3 — Format

```
PR #<n> — <title>
State: <state> · Author: <author> · Merged: <relative-time>

<body trimmed to 600 chars>

## Files (<n>)
  +<adds> -<dels>  <path>
  ...

## Commits (<n>)
  <sha[:7]>  <subject>  (<author>)

## Reviewers
  <login>: <state>  <relative-time>

## Linked issues
  <key>: <title>

## Affected graph nodes (<n>)
  <file:line>  <symbol>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:impact <symbol>` on any high-PageRank affected node."
- "Run `/oxagen:issue <key>` to see the issue context."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `multiple repos in workspace` error | Pass `--repo owner/name` |
| Empty `affected_nodes` | PR landed before the ontology ingested this repo |
