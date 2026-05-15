---
description: End-to-end onboarding — opens the Oxagen setup wizard, parses the install string, writes settings, polls ingest, and verifies the MCP connection.
argument-hint: "[paste-install-string]"
allowed-tools: Bash, Read, Write
---

# /oxagen:setup — end-to-end onboarding

Run this when wiring a new workspace into Claude Code or re-onboarding after
a token rotation. The flow is browser-driven for steps that need real UI
(login, GitHub OAuth, repo pick) and chat-driven for the install + verify
half.

## Steps

### 1 — Detect existing wiring

Run:

```bash
test -f "$HOME/.claude/oxagen.local.json" && cat "$HOME/.claude/oxagen.local.json" || echo "NO_SETTINGS"
```

If the file exists and contains a token, list MCP tools to confirm it still
works (step 6). If it lists tools without errors, tell the user "already
wired — workspace `<id>`, ingest status `<state>`" and exit. Otherwise
proceed to step 2.

### 2 — Open the browser wizard

Print this message to the user verbatim:

> Opening the Oxagen setup wizard at https://app.oxagen.ai/setup/claude-code.
> Sign in, pick or create a workspace, connect a GitHub repo, and copy the
> install string from the final screen. Paste it back into this chat when
> ready.

Run:

```bash
case "$(uname -s)" in
  Darwin) open "https://app.oxagen.ai/setup/claude-code" ;;
  Linux)  xdg-open "https://app.oxagen.ai/setup/claude-code" ;;
  *)      echo "Open https://app.oxagen.ai/setup/claude-code in your browser" ;;
esac
```

Then wait for the user to paste an install string starting with
`oxa_install_`.

### 3 — Parse the install string

Install strings look like `oxa_install_<base64-json>`. Validate and decode:

```bash
INSTALL_STRING="$1"  # provided by user
if [[ "$INSTALL_STRING" != oxa_install_* ]]; then
  echo "ERROR: Install string must start with 'oxa_install_'" >&2
  exit 1
fi

PAYLOAD_B64="${INSTALL_STRING#oxa_install_}"
if [[ "$(uname -s)" == "Darwin" ]]; then
  PAYLOAD_JSON="$(printf '%s' "$PAYLOAD_B64" | base64 -D 2>/dev/null)" || {
    echo "ERROR: Install string is not valid base64" >&2
    exit 1
  }
else
  PAYLOAD_JSON="$(printf '%s' "$PAYLOAD_B64" | base64 -d 2>/dev/null)" || {
    echo "ERROR: Install string is not valid base64" >&2
    exit 1
  }
fi

VERSION="$(printf '%s' "$PAYLOAD_JSON" | jq -r '.v // 0')"
TOKEN="$(printf '%s' "$PAYLOAD_JSON" | jq -r '.token // empty')"
WORKSPACE_ID="$(printf '%s' "$PAYLOAD_JSON" | jq -r '.workspace_id // empty')"
CONNECTION_ID="$(printf '%s' "$PAYLOAD_JSON" | jq -r '.connection_id // empty')"

if [[ "$VERSION" != "1" ]]; then
  echo "ERROR: Unsupported install-string version: $VERSION" >&2
  exit 1
fi

for var in TOKEN WORKSPACE_ID CONNECTION_ID; do
  if [[ -z "${!var}" ]]; then
    echo "ERROR: Install string missing field: $var" >&2
    exit 1
  fi
done
```

The token has the prefix `oxa_live_`. Do not log the full token — log only
the first 8 chars + `…` for any diagnostics.

### 4 — Write settings file

```bash
mkdir -p "$HOME/.claude"
jq -n \
  --arg workspace_id "$WORKSPACE_ID" \
  --arg connection_id "$CONNECTION_ID" \
  --arg token "$TOKEN" \
  --argjson minted_at "$(date +%s)" \
  '{workspace_id: $workspace_id, connection_id: $connection_id, token: $token, minted_at: $minted_at}' \
  > "$HOME/.claude/oxagen.local.json"
chmod 600 "$HOME/.claude/oxagen.local.json"
```

The `chmod 600` matters — a workspace-scoped MCP token grants read/write
access to the user's ontology.

### 5 — Persist token to shell environment

The MCP server reads `OXAGEN_MCP_TOKEN` from the environment. Tell the user
to add this line to their shell rc (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export OXAGEN_MCP_TOKEN="$(jq -r .token "$HOME/.claude/oxagen.local.json")"
```

Then have them restart Claude Code, or export it in the current session if
Claude Code reads env at MCP-connect time.

### 6 — Poll ingest status

The repo's first ingest may still be running. Poll until ready or fail:

```bash
TOKEN="$(jq -r .token "$HOME/.claude/oxagen.local.json")"
CONNECTION_ID="$(jq -r .connection_id "$HOME/.claude/oxagen.local.json")"
DEADLINE=$(( $(date +%s) + 600 ))  # 10 min cap

while [[ $(date +%s) -lt $DEADLINE ]]; do
  STATUS="$(curl -fsS \
    -H "Authorization: Bearer $TOKEN" \
    "https://api.oxagen.ai/v1/connections/github/$CONNECTION_ID/status" \
    | jq -r .status)"
  case "$STATUS" in
    ready)     echo "Ingest ready."; break ;;
    failed)    echo "Ingest failed. Visit app.oxagen.ai/workspaces/.../connections to retry." >&2; exit 1 ;;
    *)         echo "Ingest $STATUS — waiting…"; sleep 10 ;;
  esac
done
```

If the loop times out, tell the user the ingest is still running and they
can re-run `/oxagen:setup` later to verify, or check
`app.oxagen.ai/workspaces/<ws>/connections`.

### 7 — Verify MCP wiring

After ingest is ready, ask the user to restart Claude Code (or run
`/mcp` if available in their version) so the MCP client picks up the new
token. Then call any read-only tool — e.g. ask the connected MCP server
to `code.stats` — and surface the response. If tools are missing, suspect
the env var didn't propagate; instruct the user to verify
`echo $OXAGEN_MCP_TOKEN` is set in the shell that launched Claude Code.

### 8 — Confirm to user

Print a summary:

> Oxagen wired up.
>
> - Workspace: `<workspace_id>`
> - Connection: `<connection_id>` (status: ready)
> - Token: `oxa_live_<first8>…` stored at `~/.claude/oxagen.local.json`
> - Tools live: `<count>` (verified via `code.stats`)
>
> Try `/oxagen:search "<question>"` or `/oxagen:impact <symbol>` to exercise
> the graph. The `using-oxagen` skill is now active for any code-edit task.

## Failure modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| Install string parse error | Whitespace / partial copy | Ask user to copy the full string; some terminals wrap |
| Status loop times out | Large repo, slow ingest | Tell user it's normal for repos >100k LOC; come back in 30 min |
| MCP tools list empty | `OXAGEN_MCP_TOKEN` not in Claude Code's env | Restart Claude Code from a shell that has the export |
| 401 from status endpoint | Token revoked or workspace deleted | Re-run setup to mint a fresh token |

## Settings file format (reference)

```json
{
  "workspace_id": "ws_...",
  "connection_id": "conn_...",
  "token": "oxa_live_...",
  "minted_at": 1714000000
}
```

`~/.claude/oxagen.local.json` is gitignored by the plugin's `.gitignore`
and should never be committed. The `chmod 600` is enforced on every write.
