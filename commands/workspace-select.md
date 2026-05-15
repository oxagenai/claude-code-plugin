---
description: Select which Oxagen workspace this agent session should use — for MCP-only installs where multiple workspaces are available.
argument-hint: "[workspace-id-or-name]"
allowed-tools: Bash, Write, mcp__oxagen__workspace_list
---

# /oxagen:workspace-select — switch active workspace

Run this to switch the workspace your Claude Code session is pointed at.
Most useful for MCP-only installs where you have multiple workspaces (e.g.
one per repo, one per team) and need to direct the agent to a specific one
without re-running the full setup flow.

## Steps

### 1 — Load current settings

```bash
test -f "$HOME/.claude/oxagen.local.json" && cat "$HOME/.claude/oxagen.local.json" || echo "NO_SETTINGS"
```

If `NO_SETTINGS`, tell the user to run `/oxagen:setup` first — the MCP
token is required before workspace selection works.

Note the current `workspace_id` so you can show "switching from X to Y".

### 2 — List available workspaces

```
mcp__oxagen__workspace_list({})
```

This returns the workspaces the bearer token has access to.

### 3a — Argument provided

If `$1` is provided, match it against the listed workspaces by:
1. Exact UUID match on `id`
2. Case-insensitive substring match on `name`

If no match: "No workspace matched `<arg>`. Available workspaces:" then
list them and exit.

If one match: proceed to step 4.

If multiple matches: show the matches and ask the user to re-run with the
full UUID.

### 3b — Interactive selection

If no argument, print the numbered list:

```
Available workspaces:

  1. frontend-monorepo    (id: 019dc8b3-...)   [current]
  2. data-pipeline        (id: 02a4f1c8-...)
  3. mobile-app           (id: 03b2e9d1-...)

Enter a number to switch:
```

Wait for user input. If they enter a number in range, proceed with that
workspace. If out of range: "Invalid selection."

### 4 — Update settings file

Read the current `~/.claude/oxagen.local.json`, update `workspace_id`,
and write it back:

```bash
SETTINGS=$(cat "$HOME/.claude/oxagen.local.json")
# update workspace_id in the JSON
NEW_SETTINGS=$(echo "$SETTINGS" | python3 -c "
import json, sys
d = json.load(sys.stdin)
d['workspace_id'] = '<selected_id>'
print(json.dumps(d, indent=2))
")
echo "$NEW_SETTINGS" > "$HOME/.claude/oxagen.local.json"
```

### 5 — Confirm to user

```
Switched workspace:
  From: <old_name> (<old_id>)
  To:   <new_name> (<new_id>)

All subsequent MCP calls (ontology.*, memory.*, code.*) will use
workspace '<new_name>'.

If memory is enabled on this workspace, run `/oxagen:memory-logs` to
see its recent agent activity.
```

## Notes

- This command only updates the local settings file. It does not affect
  other Claude Code sessions or other machines.
- The MCP server reads `workspace_id` from the bearer token on every call.
  If your token is scoped to a single workspace, switching here has no
  effect — the server will still enforce the token's workspace scope.
- For persistent workspace assignment across machines, update the install
  string via https://app.oxagen.ai/workspaces/<id>/settings/mcp
