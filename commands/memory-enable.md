---
description: Enable agent memory on the connected workspace — seeds 3 built-in metrics and activates automatic action capture, pattern detection, and nightly benchmarks.
argument-hint: ""
allowed-tools: Bash, mcp__oxagen__workspace_enable_memory
---

# /oxagen:memory-enable — enable agent memory

Run this once per workspace to turn on durable self-improving agent memory.
After enabling, every MCP tool call is automatically captured as a
`_mem:action` node. The evaluator groups actions into sequences, promotes
recurring patterns, and writes nightly benchmarks you can track over time.

## Steps

### 1 — Check current wiring

```bash
test -f "$HOME/.claude/oxagen.local.json" && cat "$HOME/.claude/oxagen.local.json" || echo "NO_SETTINGS"
```

If `NO_SETTINGS` or no `workspace_id`, tell the user:

> Not connected. Run `/oxagen:setup` first to wire this workspace.

Exit.

### 2 — Enable memory via MCP

```
mcp__oxagen__workspace_enable_memory({})
```

The server seeds the 3 built-in metrics (task success rate, efficiency
delta, pattern utilization) and activates action capture for the workspace.

### 3 — Confirm to user

Print verbatim (substituting the actual workspace_id from settings):

> Agent memory enabled for workspace `<workspace_id>`.
>
> What happens next:
> - Every MCP tool call is now captured as a memory action
> - The evaluator groups actions into sequences every 5 minutes
> - Patterns are detected every 15 minutes once enough data accumulates
> - Nightly benchmarks track your agent's improvement over time
>
> 3 built-in metrics are seeded:
>   1. Task success rate — % of sequences with outcome = success (7-day window)
>   2. Efficiency delta — average actions per sequence vs. your baseline
>   3. Pattern utilization — % of sequences where a warning was avoided
>
> View your memory timeline and configure custom metrics:
> https://app.oxagen.ai/workspaces/<workspace_id>/memory/
>
> To define your own success metric: `/oxagen:memory-metric`
> To view recent agent actions: `/oxagen:memory-logs`

## Already enabled

If the server returns a 409 or "already enabled" response, tell the user:

> Agent memory is already enabled for this workspace. Run
> `/oxagen:memory-logs` to see recent actions, or visit
> https://app.oxagen.ai/workspaces/<workspace_id>/memory/ to review
> patterns and benchmarks.
