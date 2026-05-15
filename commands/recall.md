---
description: Recall procedures and episodes the memory module has stored that match an intent — primary M8 retrieval call.
argument-hint: "<intent> [--symbol <id>] [--outcome succeeded|failed|abandoned] [--limit N]"
allowed-tools: mcp__oxagen__memory_recall
---

# /oxagen:recall — retrieve memory by intent

Wraps `memory.recall` (M8, SPEC §6.4). Given a free-form intent
plus optional anchors, returns the most relevant procedures and
episodes the memory module has stored.

Use this at the start of a non-trivial task — past sessions may
have already worked the same problem, and recalling their
procedures is cheaper than rediscovering them.

## Steps

### 1 — Parse arguments

- `<intent>` (required) — natural-language description, ≤ 2048 chars.
- `--symbol <node-id>` — optional code.symbol UUID anchor; narrows
  recall to memories tied to that symbol.
- `--outcome` — filter to `succeeded`, `failed`, or `abandoned`.
  `failed` is the most useful for "how did the agent recover last
  time?"
- `--limit N` — max combined procedures + episodes (default 10,
  range 1–50).

### 2 — Call `memory.recall`

```
mcp__oxagen__memory_recall({
  intent: "<intent>",
  symbol_id: "<uuid-or-null>",
  outcome: "<filter-or-null>",
  limit: <n>
})
```

SPEC §7.9 envelope. Top-level fields:
- `procedures[]` — each carries `id`, `trigger_pattern`,
  `outcome_distribution`, `confidence`
- `episodes[]` — each carries `id`, `summary`, `outcome`,
  `error_signature`, `recorded_at`

### 3 — Format

```
Recall: "<intent>"

## Procedures (<n>)
  <confidence>  <trigger_pattern>
                outcomes: <succeeded>/<failed>/<abandoned>
  ...

## Episodes (<n>)
  <relative-time>  [<outcome>]  <summary>
                   error: <error_signature>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:procedure-for <trigger>` for a specific recipe."
- "Run `/oxagen:remember <fact>` to add a new episode."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Memory may be disabled — run `/oxagen:memory-enable` |
| `intent too long` | Trim under 2048 chars |
