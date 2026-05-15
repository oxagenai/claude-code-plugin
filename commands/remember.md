---
description: Persist a free-form fact as a memory episode — anchored to the active agent session and optionally a symbol.
argument-hint: "<fact> [--outcome succeeded|failed|abandoned] [--symbol <id>] [--error-signature <sig>]"
allowed-tools: mcp__oxagen__memory_remember
---

# /oxagen:remember — store a memory episode

Wraps `memory.remember` (M8, SPEC §6.4). Stores a free-form fact
as an Episode in the memory module so future `/oxagen:recall`
calls can find it.

Use this after a non-trivial decision — what you tried, what
worked, what didn't — so the next session doesn't redo the
investigation.

## Steps

### 1 — Parse arguments

- `<fact>` (required) — free-form text, 1–4096 chars.
- `--outcome` — `succeeded` (default), `failed`, or `abandoned`.
- `--symbol <uuid>` — optional `code.symbol` UUID the agent was
  working on; promoted to the SPEC §6.2 cluster anchor.
- `--error-signature <sig>` — required (alongside
  `--outcome failed`) to participate in failure clustering.

### 2 — Resolve the active session

Memory writes need an active `agent.session` UUID. The plugin's
session tooling already handles this; surface an error if no
session is bound.

### 3 — Call `memory.remember`

```
mcp__oxagen__memory_remember({
  fact: "<fact>",
  session_id: "<active-session-uuid>",
  kind: "episode",
  outcome: "<outcome>",
  triggering_symbol_id: "<uuid-or-null>",
  error_signature: "<sig-or-null>"
})
```

### 4 — Confirm

```
Remembered episode <episode_id>.
Outcome: <outcome>
Anchor: <symbol or "none">
```

### 5 — Suggest follow-ups

- "Run `/oxagen:recall <intent>` to confirm retrieval works."
- "Run `/oxagen:procedure-for <trigger>` if a recipe exists."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `no active session` | Bind one via the agent.start_session flow |
| `fact too long` | Trim under 4096 chars |
| Memory disabled | Run `/oxagen:memory-enable` |
