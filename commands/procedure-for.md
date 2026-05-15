---
description: Look up a memory procedure by exact trigger pattern — returns the recipe a past session promoted for this scenario.
argument-hint: "<trigger-pattern>"
allowed-tools: mcp__oxagen__memory_procedure_for
---

# /oxagen:procedure-for — fetch a stored procedure

Wraps `memory.procedure_for` (M8, SPEC §6.4). Given the exact
canonical trigger pattern a procedure was promoted under, returns
the procedure body, success / failure distribution, and the
episodes that fed into it.

## Steps

### 1 — Parse argument

`<trigger-pattern>` (required) — exact string the writer stamped
at promotion time. Length 1–512.

### 2 — Call `memory.procedure_for`

```
mcp__oxagen__memory_procedure_for({
  trigger: "<pattern>"
})
```

SPEC §7.9 envelope. Top-level fields:
- `procedure_id`, `trigger_pattern`, `body`
- `outcome_distribution` — succeeded / failed / abandoned counts
- `confidence`
- `source_episodes[]` — episode UUIDs the promotion was based on

### 3 — Format

```
Procedure for "<trigger>"
Confidence: <conf>  ·  Distribution: <s>/<f>/<a>

## Body
<body>

## Source episodes (<n>)
  <episode_id>  <relative-time>
  ...
```

### 4 — Suggest follow-ups

- "Run `/oxagen:recall` for fuzzy intent matching."
- "Run `/oxagen:forget <id>` if the procedure is stale."

## Failure modes

| Symptom | Fix |
|---------|-----|
| `no procedure` | Trigger pattern doesn't match — try `/oxagen:recall` for fuzzy lookup |
| Empty `source_episodes` | Procedure was seeded manually; no source chain to walk |
