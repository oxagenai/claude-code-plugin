---
description: Dispatch the Oxagen agent swarm against Linear issues — sandboxed Claude Code sessions that open PRs and write back to the knowledge graph.
argument-hint: "[--p0,p1,hotfix | --issue OXA-123]"
allowed-tools: Bash
---

# /oxagen:go — dispatch the agent swarm

In-session counterpart to the `oxagen go` CLI. Use when you want to fan
out work to sandboxes from inside an existing Claude Code session.

## Steps

1. Confirm `.oxagen.toml` exists at the repo root. If not, instruct the
   user to run `oxagen init --workspace-id <uuid>`.
2. Translate the arguments into the corresponding `oxagen go` flags:
   - `--p0` / `--p1` / `--hotfix` map directly
   - `--issue OXA-123` becomes `--issue OXA-123`
3. Run:

```bash
oxagen go $ARGS --auto-merge
```

4. Report the resulting outcome table back to the user. Surface any
   `blockedReasons` from the no-slop quality gate verbatim — never
   summarise them away.
