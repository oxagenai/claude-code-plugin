---
description: Claim a line range or symbol so other Oxagen-connected agents back off while you edit. Releases on completion or session end.
argument-hint: "<file:line-range | qualified-name> [--reason \"<why>\"] [--force]"
allowed-tools: mcp__oxagen__agent_claim_work, mcp__oxagen__agent_release_claim, mcp__oxagen__agent_list_active_claims, mcp__oxagen__ontology_get_node
---

# /oxagen:claim — coordinate with other agents

Run this before editing code in a workspace where multiple agents may be
active. The MCP server records the claim on the workspace's typed graph;
other Oxagen-connected sessions running `/oxagen:impact` or
`agent.list_active_claims` see your claim and back off.

## Steps

### 1 — Parse the argument

The argument is one of:

- `<file>:<start>-<end>` — explicit line range, e.g. `src/auth/login.ts:42-87`
- `<file>:<line>` — single-line claim, treated as `line-line`
- `<qualified-name>` — symbol claim; resolves via `ontology.get_node` to
  the symbol's source range

Optional flags:

- `--reason "<why>"` — appends a human-readable note to the claim so
  other sessions see context.
- `--force` — bypasses the conflict check in step 3 and forwards
  `force: true` to `agent_claim_work` in step 4. Only set this when
  the user passed the flag explicitly; never auto-enable.

### 2 — Resolve the symbol (if name-based)

```
mcp__oxagen__ontology_get_node({ identifier: "<argument>" })
```

If the lookup returns no node, surface to the user:

> No symbol matched `<argument>`. Pass an explicit `<file>:<line-range>`
> instead, or run `/oxagen:search` to find the right name.

### 3 — Check existing claims

```
mcp__oxagen__agent_list_active_claims({
  file: "<file>",
  // server filters by overlapping range
})
```

If any claim from a different session overlaps the requested range:

> Active claim from session `<label>` covers `<file:overlap-range>`
> (claimed `<relative time>` ago, reason: `<reason>`).
>
> Options:
>   1. Wait — try `/oxagen:claim` again in a few minutes
>   2. Coordinate — message the other session's owner
>   3. Override — pass `--force` (rare, surfaces an audit event)
>
> Aborting unless you re-run with `--force`.

Do not auto-override.

### 4 — Acquire the claim

If the user passed `--force`, include `force: true` in the call so the
server bypasses its conflict check and writes an audit event. Omit the
field otherwise.

```
mcp__oxagen__agent_claim_work({
  file: "<file>",
  start_line: <n>,
  end_line: <n>,
  reason: "<optional reason>",
  force: true   // only when the user passed --force
})
```

Server returns:

```
{
  "claim_id": "claim_...",
  "expires_at": "<iso8601>",
  "session_id": "<this-session>"
}
```

Default expiry is 30 minutes; the session's `agent.heartbeat` extends
it. If the session ends without `release_claim`, the server reaps the
claim at expiry.

### 5 — Confirm to user

```
Claimed <file:line-range> for ~30 minutes.
Reason: <reason or "none">
Claim ID: <claim_id>

While this claim is active:
  - Other Oxagen-connected agents see this range as locked
  - Heartbeats from this session extend the claim
  - To free it before expiry, ask Claude: "release claim <claim_id>"
    (or pass `--release <claim_id>` when that flag ships)
```

## Releasing

Two paths:

1. **Explicit:** the user runs `/oxagen:release <claim_id>` (TODO — not in
   v0.1) or this command supports `--release <claim_id>` to release
   immediately.
2. **Automatic:** session ends (Claude Code closes), heartbeat stops,
   server reaps after expiry.

For v0.1, recommend the user run `agent.release_claim` directly via the
chat ("release claim claim_xxx") if they want to free it before the
session ends.

## Failure modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| `ClaimOwnershipError` from server | Range overlaps an active claim by another session | Re-run after the other session releases; or use `--force` (records audit event) |
| `ontology.get_node` returns null on a name argument | Symbol not ingested or misspelled | Pass explicit `<file>:<line-range>` instead |
| Claim disappears mid-edit | This session lost heartbeat (sleep, network) | Re-claim; the server treats heartbeat-stale sessions as failed |

## When to use this

- Multiple agents working in the same repo (parallel automated review,
  swarm refactors, multi-agent feature work)
- Long-running edits where another session might wander in
- Refactors that touch many files — claim the file set up front, edit
  in any order, release at the end

## When NOT to use this

- Solo work in a single Claude Code session — the claim adds no value
- Read-only tasks (`/oxagen:search`, `/oxagen:impact` for analysis only)
- Edits to brand-new files no other session has indexed yet
