---
description: Soft-delete a memory episode or procedure. Use when a stored memory is wrong or obsolete.
argument-hint: "<memory-id> [--require-grant]"
allowed-tools: mcp__oxagen__memory_forget
---

# /oxagen:forget — delete a memory entry

Wraps `memory.forget` (M8, SPEC §6.4). Soft-deletes an Episode or
Procedure UUID from the memory module so future `/oxagen:recall`
calls don't return it.

## Steps

### 1 — Parse arguments

- `<memory-id>` (required) — UUID of the Episode or Procedure.
- `--require-grant` — set when the target is pinned or covered by
  a tenant-policy lock. Without the flag, the call raises a
  permission error on protected rows.

### 2 — Confirm with the user

Before calling, surface the target:

> About to forget memory `<id>`.
> This is a soft-delete; the row is hidden from future recalls
> but kept for audit. Continue? (y/n)

Only proceed on explicit confirmation.

### 3 — Call `memory.forget`

```
mcp__oxagen__memory_forget({
  memory_id: "<uuid>",
  require_grant: <bool>
})
```

### 4 — Confirm

```
Forgot memory <id>.
```

### 5 — Suggest follow-ups

- "Run `/oxagen:recall <intent>` to confirm the entry is gone."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `permission denied — pinned` | Pass `--require-grant` if you genuinely need to delete |
| `not found` | Memory id may already be soft-deleted |
