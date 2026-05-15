---
description: Define a custom eval metric for your workspace — specify what "better" means for your agents and track improvement over time.
argument-hint: "[name] [condition]"
allowed-tools: Bash, mcp__oxagen__memory_metric_create, mcp__oxagen__memory_metric_list
---

# /oxagen:memory-metric — define a custom eval metric

Run this to create a workspace-scoped metric that the nightly evaluator
computes against your agent's action history. Metrics answer the question
"are my agents getting better at this specific thing?"

## Examples

```
/oxagen:memory-metric "Schema merge success rate" "outcome == 'success'"
/oxagen:memory-metric "Fast mutations" "duration_ms < 3000"
/oxagen:memory-metric
```

Omit arguments for interactive mode.

## Steps

### 1 — Load settings

```bash
test -f "$HOME/.claude/oxagen.local.json" && cat "$HOME/.claude/oxagen.local.json" || echo "NO_SETTINGS"
```

If no `workspace_id`, tell the user to run `/oxagen:setup` first.

### 2 — Parse arguments

If both `$1` (name) and `$2` (condition) are provided, skip to step 4.

If only `$1` is provided, use it as the name and ask for the condition
interactively (step 3).

If no arguments, run interactive mode (step 3).

### 3 — Interactive collection

Ask each question in sequence (do not ask all at once):

**Name:** "What should this metric be called? (e.g. 'Schema merge success rate')"

**Action filter:** "Which action types should this metric cover?
  Options: ontology_mutation, graph_query, tool_call, connection_ingest, config_change, dead_end_search
  Press Enter to include all action types."

**Success condition:** "What counts as a success? Enter an expression:
  Examples:
    outcome == 'success'          → sequence completed without failure
    action_count < 5              → agent solved it in under 5 steps
    duration_ms < 3000            → completed in under 3 seconds
  Expression:"

**Aggregation:** "How should this be aggregated?
  1. rate   — fraction of sequences that meet the condition (default)
  2. average — mean value across sequences
  3. p95    — 95th percentile
  Enter 1, 2, or 3 (default: 1):"

**Window:** "How many days should the rolling window cover? (default: 7)"

### 4 — Create the metric

```
mcp__oxagen__memory_metric_create({
  name: "<name>",
  event_filter: { "action_type": ["<types>"] },   // omit for all types
  success_condition: "<condition>",
  aggregation: "<rate|average|p95>",
  window_days: <n>
})
```

### 5 — Confirm to user

```
Metric '<name>' created (id: <id>).

It will be evaluated in tonight's nightly pass (04:00 UTC). After
the first evaluation you can track your trend at:
https://app.oxagen.ai/workspaces/<workspace_id>/memory/metrics/

To list all metrics: /oxagen:memory-metric --list
To see benchmark history: use memory.benchmark_get via the MCP server
```

### 6 — List mode

If the user runs `/oxagen:memory-metric --list`:

```
mcp__oxagen__memory_metric_list({ include_inactive: false })
```

Format results as a table:

| Name | Aggregation | Window | Builtin |
|------|-------------|--------|---------|
| ...  | ...         | ...    | ...     |

## Failure modes

| Symptom | Fix |
|---------|-----|
| "memory not enabled" error | Run `/oxagen:memory-enable` first |
| Condition syntax unclear | Use exact strings from the examples above |
| Metric not appearing in benchmarks | Wait for tonight's nightly pass at 04:00 UTC |
