---
description: View recent agent actions written to memory — see what your agent has been doing and how each action resolved.
argument-hint: "[limit]"
allowed-tools: Bash, mcp__oxagen__memory_search
---

# /oxagen:memory-logs — view recent agent memory actions

Shows the most recent agent actions captured by the memory layer. Useful
for understanding what your agent did in a past session, auditing failures,
or confirming memory capture is working.

## Steps

### 1 — Load settings

```bash
test -f "$HOME/.claude/oxagen.local.json" && cat "$HOME/.claude/oxagen.local.json" || echo "NO_SETTINGS"
```

If no `workspace_id`, tell the user to run `/oxagen:setup` first.

### 2 — Parse limit argument

Default limit: 20. If `$1` is a positive integer, use it (cap at 200).

### 3 — Fetch recent actions

```
mcp__oxagen__memory_search({
  limit: <limit>
})
```

### 4 — Format output

If results are returned, render as a table:

```
AGENT MEMORY — last <N> actions

timestamp            | agent_id         | action_type          | status   | duration_ms
---------------------|------------------|----------------------|----------|------------
2026-04-29 03:12:44  | claude-code       | ontology_mutation    | success  | 142
2026-04-29 03:12:41  | claude-code       | graph_query          | success  | 38
2026-04-29 03:11:55  | claude-code       | tool_call            | failure  | 2041
...
```

Then print a summary line:

```
<N> actions shown (workspace <workspace_id>).

Sequences: actions are grouped into sequences every 5 minutes.
Patterns:  recurring failures are promoted to patterns every 15 minutes.

View full timeline: https://app.oxagen.ai/workspaces/<workspace_id>/memory/
```

### 5 — Empty results

If no actions are returned:

> No memory actions found for this workspace.
>
> Possible reasons:
> 1. Memory is not enabled — run `/oxagen:memory-enable` to activate it
> 2. No MCP tool calls have been made yet in this workspace
> 3. The workspace_id in settings doesn't match an active workspace
>
> Check: https://app.oxagen.ai/workspaces/<workspace_id>/memory/

### 6 — Filter by outcome (optional)

If the user adds `--failures`, pass `outcome: "failure"` to `memory.search`
and show only failed actions with their `error_code`.

## Combining with other commands

- `/oxagen:memory-logs 50 --failures` — last 50 failures only
- After reviewing failures, run `/oxagen:memory-metric` to define a metric
  tracking your specific failure pattern
- Run `/oxagen:impact` to see which nodes the failing actions touched
