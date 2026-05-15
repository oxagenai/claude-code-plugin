---
description: Open, resolve, or link an incident on the workspace ontology — tracks production issues against the graph nodes they affect.
argument-hint: "<open|resolve|link> [--title \"...\"] [--node <id>] [--incident <id>]"
allowed-tools: mcp__oxagen__incident_open, mcp__oxagen__incident_resolve, mcp__oxagen__incident_link, mcp__oxagen__ontology_get_node
---

# /oxagen:incident — incident tracking on the ontology

Links production incidents to the workspace knowledge graph so you can
query "which nodes were affected by incident X?" and "has this node been
involved in past incidents?" Run this when a production issue is detected
or resolved, or to associate an existing incident with a specific symbol.

## Subcommands

### `open` — create a new incident

```
/oxagen:incident open --title "Billing double-charge" [--node <file:line|name>]
```

### `resolve` — mark an incident resolved

```
/oxagen:incident resolve --incident <incident-id> [--note "root cause: ..."]
```

### `link` — associate a node with an open incident

```
/oxagen:incident link --incident <incident-id> --node <file:line|name>
```

## Steps

### 1 — Parse the subcommand

Valid values: `open`, `resolve`, `link`. If missing or unrecognized, print
the usage above and exit.

### 2a — Open an incident

Collect `--title` from arguments (required) and `--node` (optional).

If `--node` is provided, resolve it first:

```
mcp__oxagen__ontology_get_node({ identifier: "<node>" })
```

Then create the incident:

```
mcp__oxagen__incident_open({
  title: "<title>",
  node_id: "<resolved-id>"   // omit if no node provided
})
```

Confirm:

```
Incident opened: <incident_id>
Title: <title>
Linked node: <qualified-name> at <file:line>   // if provided

To link additional nodes:
  /oxagen:incident link --incident <id> --node <symbol>

To resolve:
  /oxagen:incident resolve --incident <id> --note "<root cause>"
```

### 2b — Resolve an incident

Require `--incident <id>`. `--note` is optional.

```
mcp__oxagen__incident_resolve({
  incident_id: "<id>",
  note: "<note>"    // omit if not provided
})
```

Confirm:

```
Incident <id> resolved.
Note recorded: <note>   // if provided

The incident and its linked nodes are preserved in the ontology for
post-mortem queries. Run `/oxagen:ask "which nodes were affected by
incident <id>?"` to review.
```

### 2c — Link a node to an incident

Require both `--incident <id>` and `--node <symbol>`.

Resolve the node:

```
mcp__oxagen__ontology_get_node({ identifier: "<node>" })
```

Then link:

```
mcp__oxagen__incident_link({
  incident_id: "<id>",
  node_id: "<resolved-id>"
})
```

Confirm:

```
Linked <qualified-name> (<file:line>) to incident <id>.
```

## Failure modes

| Symptom | Fix |
|---------|-----|
| `incident_open` errors with "node not found" | Pass the node via `--node` only after confirming with `ontology.get_node` |
| `incident_resolve` "incident not found" | Double-check the incident ID; list open incidents via `/oxagen:ask "list open incidents"` |
| Node resolves to null | Symbol not ingested; pass an explicit `<file>:<line>` |

## Post-mortem queries

After resolving, use `/oxagen:ask` to query incident history:

- "Which nodes were affected by the most incidents this month?"
- "What functions were linked to incident INC-042?"
- "Show incidents involving the billing module."
