---
description: Show full issue context — title, body, comments, linked PRs, and affected graph nodes — from the workspace ontology.
argument-hint: "<issue-key | issue-number> [--repo owner/name]"
allowed-tools: mcp__oxagen__code_issue_context
---

# /oxagen:issue — issue context from the ontology

Wraps `code.issue_context` (M9, SPEC §7.6). One call returns the
issue's title, body, comments, labels, linked PRs, and the graph
nodes the issue mentions.

## Steps

### 1 — Parse arguments

- `<issue-key>` — Linear issue key (e.g. `OXA-620`) or
  GitHub issue number.
- `--repo owner/name` (optional) — disambiguate when the workspace
  has multiple repos.

### 2 — Call `code.issue_context`

```
mcp__oxagen__code_issue_context({
  issue_key: "<key-or-number>",
  repo_full_name: "<owner/name>"   // omit for single-repo workspace
})
```

SPEC §7.9 envelope. Top-level fields:
- `title`, `body`, `state`, `created_at`, `closed_at`
- `labels`, `assignees`, `author`
- `comments` (author, body, created_at)
- `linked_prs` (number, title, merged_at)
- `affected_nodes` (graph node IDs the issue references)

### 3 — Format

```
Issue <key>: <title>
State: <state>  ·  Created: <relative-time>  ·  Author: <login>
Labels: <l1>, <l2>

<body trimmed to 800 chars>

## Comments (<n>)
  <relative-time>  <author>: <body trimmed to 200 chars>
  ...

## Linked PRs
  PR #<num>: <title>  (<merged or open>)

## Affected graph nodes (<n>)
  <file:line>  <symbol>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:pr <num>` for any linked PR."
- "Run `/oxagen:impact <symbol>` on a high-priority affected node."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `issue not found` | Verify the key / number; pass `--repo` if needed |
| Empty `affected_nodes` | Issue body had no resolvable symbol references |
