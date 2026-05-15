---
description: List open security alerts for the workspace — Dependabot, CodeQL, and secret-scan findings with severity and affected nodes.
argument-hint: "[--severity critical|high|medium|low] [--repo owner/name]"
allowed-tools: mcp__oxagen__security_open_alerts
---

# /oxagen:security-alerts — open security alerts

Wraps `security.open_alerts`. Returns the open Dependabot,
CodeQL, and secret-scan alerts the workspace has ingested, joined
to the affected graph nodes.

## Steps

### 1 — Parse arguments

- `--severity` — filter to `critical`, `high`, `medium`, or `low`.
  Omit for all severities.
- `--repo owner/name` — required for multi-repo workspaces.

### 2 — Call `security.open_alerts`

```
mcp__oxagen__security_open_alerts({
  repo_full_name: "<owner/name>",
  severity: "<severity-or-null>",
  limit: 50
})
```

SPEC §7.9 envelope. Rows carry `alert_id`, `kind` (`dependabot` /
`codeql` / `secret_scan`), `severity`, `summary`, `created_at`,
`affected_path`, `affected_node_id`.

### 3 — Format

Sort by severity desc:

```
Open security alerts (<N>)

  [<severity>]  <kind>  <alert_id>
                <summary>
                <affected_path>
                opened <relative-time>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:alert <id>` for the full alert context."
- "Run `/oxagen:impact <symbol>` if the alert names a function."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | All clear, or security ingest not connected — check `app.oxagen.ai` |
