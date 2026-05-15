---
description: Show full context for a security alert — description, affected nodes, recommended fix, and links to upstream advisories.
argument-hint: "<alert-id> [--repo owner/name]"
allowed-tools: mcp__oxagen__security_alert_context
---

# /oxagen:alert — full security-alert context

Wraps `security.alert_context`. Given an alert id (Dependabot,
CodeQL, or secret-scan), returns the full alert body, affected
graph nodes, recommended fix, and advisory links.

## Steps

### 1 — Parse arguments

- `<alert-id>` (required) — id from `/oxagen:security-alerts`.
- `--repo owner/name` — required for multi-repo workspaces.

### 2 — Call `security.alert_context`

```
mcp__oxagen__security_alert_context({
  alert_id: "<id>",
  repo_full_name: "<owner/name>"
})
```

SPEC §7.9 envelope. Top-level fields:
- `kind`, `severity`, `state`, `created_at`, `dismissed_at`
- `summary`, `description`, `recommendation`
- `cve_id`, `ghsa_id`, `cwe_ids`, `references[]`
- `affected_nodes[]` — graph nodes the alert touches
- `fixed_versions[]` — for Dependabot

### 3 — Format

```
Alert <id>  [<severity>] <kind>
State: <state>  ·  Opened: <relative-time>

<summary>

## Description
<description>

## Recommendation
<recommendation>

## Affected
  <node>  <file:line>
  ...

## References
  <url>
  ...

## Identifiers
  CVE: <cve_id>  ·  GHSA: <ghsa_id>  ·  CWE: <cwe_ids>
```

### 4 — Suggest follow-ups

- "Run `/oxagen:impact <symbol>` on any affected node before
  patching."
- "Run `/oxagen:find-pattern <package>` to locate every usage of
  the affected dependency."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `alert not found` | Confirm the id via `/oxagen:security-alerts` |
| Empty `affected_nodes` | Alert maps to a manifest line only — no symbol resolution |
