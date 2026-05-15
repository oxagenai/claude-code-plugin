# Oxagen — Claude Code plugin

> Ontology layer for AI agents. Typed, queryable knowledge graph exposed
> to Claude Code as MCP-native tools so every answer, edit, triage, or
> recall is grounded in deterministic graph context instead of guessing.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![Plugin: Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-d97706)](https://docs.claude.com/en/docs/claude-code/plugins)
[![Docs](https://img.shields.io/badge/docs-docs.oxagen.ai-blue)](https://docs.oxagen.ai/plugins/claude-code-plugin)

- **Docs:** <https://docs.oxagen.ai/plugins/claude-code-plugin>
- **App:** <https://app.oxagen.ai>
- **Issues:** <https://github.com/oxagenai/claude-code-plugin/issues>
- **Marketplace listing:** `oxagenai/claude-code-plugin`

---

## TL;DR

Install once. Get:

- A typed, deterministic **knowledge graph** of your workspace (code,
  history, tests, CI, security alerts, incidents, ownership, runbooks,
  customers — whatever the workspace ontology tracks).
- **Always-on memory** — every meaningful action is captured, recurring
  patterns are detected, past failures are injected as warnings before
  similar work, and procedures get promoted to reusable recipes.
- **50+ slash commands** under `/oxagen:*` covering discovery, traversal,
  history, tests, CI, security, memory, incidents, and multi-agent
  coordination.
- A **SessionStart hook** that pins three guarantees on every turn:
  *graph first*, *memory always recalled*, *coordination respected*.
  Persists across `/clear` and `/compact`.

---

## Install

### From the Claude Code plugin marketplace (recommended)

```bash
# Add the Oxagen marketplace
/plugin marketplace add oxagenai/claude-code-plugin

# Install the plugin
/plugin install oxagen@oxagen
```

That's it — no manual symlinks, no `~/.claude/plugins` editing. The
manifest, MCP server config, hooks, commands, and skills all come from the
marketplace tree.

### Local development install

If you've cloned this repo and want to dogfood your changes:

```bash
ln -s "$(pwd)" ~/.claude/plugins/oxagen
```

Then restart Claude Code.

### Wire your workspace

```
/oxagen:setup
```

The command opens `https://app.oxagen.ai/setup/claude-code` in your
browser, walks you through workspace + GitHub repo selection, hands back a
one-line install string, writes `~/.claude/oxagen.local.json`, polls until
the first ingest is ready, and verifies the MCP tool list is live.

Add to your shell profile so the MCP token survives restarts:

```bash
export OXAGEN_MCP_TOKEN="$(jq -r .token ~/.claude/oxagen.local.json)"
```

Verify with:

```
/mcp
```

You should see `oxagen` listed with its full tool set.

---

## What the plugin guarantees

When the Oxagen plugin is enabled, **on every session** the SessionStart
hook injects an `<EXTREMELY_IMPORTANT>` preamble that tells the agent:

1. **Graph first, guess second.** Before any non-trivial action — code
   edit, research synthesis, incident triage, ops change, customer-impact
   analysis, policy lookup — query the Oxagen graph (`ontology.*`,
   `code.*`, `incident.*`, `security.*`).
2. **Memory always recalled.** At the start of any task, call
   `memory.recall` (or `memory.context_inject`) on the intent. If a
   procedure or episode matches, prefer it over rediscovery. After
   meaningful work, persist what you learned.
3. **Coordinate.** Use `agent.list_active_claims` and `agent.claim_work`
   when other sessions may be touching the same scope.

The full routing policy lives in the `oxagen:using-oxagen` skill, which
is loaded into the system prompt by the hook on `startup`, `clear`, and
`compact`. The complete tool catalog and write-back conventions live in
the `oxagen:oxagen-tool-reference` and `oxagen:dogfooding` skills, which
are loaded on demand.

---

## Real-world use cases — beyond agentic coding

The plugin's value is the typed graph + memory layer. Coding is the most
obvious surface. It is not the only one. Anything Oxagen ingests becomes
queryable and groundable.

### Engineering

- **Refactor with confidence.** `/oxagen:impact <symbol>` returns
  callers, dependents, PageRank, and reach paths *before* you ship a
  signature change. Stops the "I didn't realize ten things called this"
  class of regression.
- **Onboarding.** `/oxagen:repo-overview`, `/oxagen:module-tree`,
  `/oxagen:who-knows <symbol>`, `/oxagen:expert <login>` — the new hire
  gets a cartography of the system and its owners in five minutes
  instead of three weeks.
- **Dead-code audit.** `/oxagen:ask "list nodes with zero callers and no
  EXPORTS edge"` returns provably dead symbols. Safe to delete.
- **Architecture review.** `/oxagen:ask "show cycles in the call graph
  scoped to packages/oxagen"` surfaces SCCs you can't see from grep.

### Incident response & SRE

- **Blast-radius from a stack trace.**
  `/oxagen:find-symbol <function>` → `/oxagen:impact <symbol>` →
  `/oxagen:pr-history <symbol>` reconstructs what changed, who touched
  it, and what depends on it — minutes after the page fires.
- **Runbook recall.** `/oxagen:procedure-for "redis OOM during ingest"`
  pulls back the procedure stored from the last incident; no wiki
  hunting at 3am.
- **Persist the post-mortem.** `/oxagen:incident resolve` plus
  `/oxagen:remember "OOM root-caused to chunk size 16k; mitigated by
  bounded queue at packages/oxagen/oxagen/ingest/queue.py:142"` ensures
  the next agent who sees a similar trace gets a context injection.

### Security & compliance

- **Triage with reach.** `/oxagen:security-alerts --severity=high`
  returns open Dependabot / CodeQL / secret-scan alerts joined to the
  call graph. `/oxagen:alert <id>` shows affected nodes + recommended
  fix + advisory links — no GitHub tab-switching.
- **Evidence chains.** Map controls → artifacts → owners in a
  workspace-scoped ontology; ask the agent to gather evidence by
  traversing typed edges instead of grepping.
- **Supply-chain audit.** `/oxagen:ask "dependencies introduced in the
  last 30 days with no PR review"` — typed query, deterministic answer.

### Customer success & support

- **Customer-impact analysis.** Workspaces with a customer ontology
  layer can ask "which customers depend on the feature gated by
  `payments.charge`" — typed traversal returns the list with confidence,
  not a guess.
- **Escalation graphs.** `/oxagen:expert <area>` resolves to the human
  who most recently authored or reviewed code in that area; pair with
  `memory.procedure_for` to surface the runbook the last on-call used.
- **Ticket → code link.** `/oxagen:issue <key>` joins issue body,
  comments, linked PRs, and affected nodes — the agent pulls the same
  context a senior engineer would assemble manually.

### Research, R&D, internal knowledge

- **Concept graphs.** Ingest a corpus (papers, docs, internal wikis) into
  a workspace; the agent answers "how does concept A relate to concept
  B" via `ontology.traverse` instead of vector-search guessing.
- **Long-running research memory.** `memory.remember` + `memory.recall`
  give a research agent durable, anchored episodes — failed approaches
  show up as warnings before the agent retries them.
- **Cross-document synthesis.** `/oxagen:find-pattern` does hybrid
  semantic + grep search across the typed ontology, scoped to a
  workspace.

### Multi-agent fleets

- **Claim before edit.** `/oxagen:claim packages/oxagen/.../service.py:50-120`
  registers the lock; other Oxagen-connected sessions see it and back
  off. Auto-releases on session end.
- **Share findings.** `agent.record_finding` persists a typed observation
  ("this caller breaks if signature changes") on the affected node;
  `agent.get_findings` retrieves them at the start of the next task.
- **Orchestrate.** `/oxagen:go --p0 "investigate auth regression"`
  dispatches the Oxagen agent swarm against the workspace; each agent
  inherits the same graph + memory + claim coordination.

---

## Slash commands

All commands use the `/oxagen:` prefix. Run any command with no arguments
to get interactive prompts.

### Setup and Q&A

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:setup` | `[install-string]` | End-to-end onboarding: workspace → GitHub → token → ingest |
| `/oxagen:ask` | `<question>` | Direct Q&A against the ontology — prose answer backed by Cypher |
| `/oxagen:search` | `<query>` | NL→Cypher graph search with traversal fallback, returns file:line hits |

### Discovery and structure

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:repo-overview` | — | Workspace summary — language mix, top modules, entry points, node counts |
| `/oxagen:module-tree` | `[path]` | Directory tree with per-subtree symbol counts |
| `/oxagen:find-symbol` | `<name>` | Resolve a name to one or more ontology nodes |
| `/oxagen:describe` | `<symbol>` | Structural metadata: kind, signature, decorators, docstring, location |
| `/oxagen:read` | `<symbol>` | Source bytes + summary + line range with configurable surrounding context |
| `/oxagen:find-pattern` | `<pattern>` | Hybrid semantic + grep search across the ontology |
| `/oxagen:explain` | `<symbol>` | Behavioural description: inputs, outputs, calls, reads, writes, raises |

### Traversal

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:impact` | `<symbol>` | Blast-radius report — callers, deps, PageRank, reach path, active claims |
| `/oxagen:callers` | `<symbol>` | Direct callers (alias `code.callers_of`) |
| `/oxagen:callees` | `<symbol>` | Outgoing CALLS edges from the symbol |
| `/oxagen:refs` | `<symbol>` | All references — calls, imports, type uses |
| `/oxagen:path` | `<from> <to>` | Directed path between two symbols (alias `code.dependency_path`) |
| `/oxagen:cochange` | `<symbol>` | Symbols that historically change together with the target |

### History and ownership

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:who-knows` | `<symbol>` | People ranked by authorship + review activity on a symbol |
| `/oxagen:expert` | `<login>` | What a person owns — inverse of `/oxagen:who-knows` |
| `/oxagen:recent` | `[--days] [--path]` | Recent commits across the workspace |
| `/oxagen:blame` | `<file>` | Enriched blame with PR, issue, and reviewer joins |
| `/oxagen:pr` | `<num>` | Full PR context — files, commits, reviewers, linked issues, affected nodes |
| `/oxagen:pr-history` | `<symbol>` | PRs that have touched a symbol, newest first |
| `/oxagen:issue` | `<key>` | Issue context — title, body, comments, linked PRs, affected nodes |
| `/oxagen:find-issues` | `<query>` | Hybrid full-text search across the issue corpus |

### Tests and CI

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:tests-for` | `<symbol>` | Tests covering a symbol — direct + transitive |
| `/oxagen:coverage` | `<file>` | Per-line coverage from the latest CI run |
| `/oxagen:failing-tests` | `[--branch]` | Tests currently failing on the branch |
| `/oxagen:flaky-tests` | `[--days]` | Flake-rate ranking over a rolling window |
| `/oxagen:last-run` | `[workflow]` | Most-recent CI workflow run for a workflow / branch / PR |
| `/oxagen:run-failures` | `<run-id>` | Failed jobs and steps for a workflow run with cold-log preview |

### Security

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:security-alerts` | `[--severity]` | Open Dependabot, CodeQL, and secret-scan alerts |
| `/oxagen:alert` | `<id>` | Full alert context — affected nodes, recommendation, advisory links |

### Memory

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:recall` | `<intent>` | Retrieve procedures + episodes matching an intent |
| `/oxagen:remember` | `<fact>` | Persist a free-form fact as a memory episode |
| `/oxagen:procedure-for` | `<trigger>` | Look up a stored procedure by exact trigger pattern |
| `/oxagen:forget` | `<memory-id>` | Soft-delete an episode or procedure |
| `/oxagen:memory-enable` | — | Enable durable agent memory (automatic capture + pattern injection) |
| `/oxagen:memory-metric` | `[name] [condition]` | Define a custom self-improvement eval metric |
| `/oxagen:memory-logs` | `[limit]` | View recent agent actions captured by memory |

### Coordination and ops

| Command | Argument | What it does |
|---------|----------|---|
| `/oxagen:claim` | `<file:line \| symbol>` | Lock a range so parallel agents back off; auto-releases on session end |
| `/oxagen:incident` | `<open\|resolve\|link>` | Track production incidents against graph nodes |
| `/oxagen:workspace-select` | `[id-or-name]` | Switch active workspace for this session |
| `/oxagen:go` | `[--p0\|--issue OXA-N]` | Dispatch the Oxagen agent swarm |
| `/oxagen:watch` | `[--daemons]` | Start the Oxagen monitor supervisor |

> Full per-command reference, including arguments and failure modes, lives
> in [`commands/`](./commands) — each file is the runbook the agent
> follows when the slash command fires.

---

## MCP tool surface

All tools follow `mcp__oxagen__<group>_<action>` and return the canonical
SPEC §7.9 envelope: `results`, `evidence`, `tokens_used_estimate`,
`counterfactual_estimate_tokens`, `counterfactual_method`, `cursor`,
`tenant_scoped_at`. The agent parses one shape regardless of which tool
emitted it.

| Group | Surface |
|---|---|
| `ontology.*` | `ask`, `get_node`, `list_nodes`, `create_node`, `list_edges`, `create_edge`, `list_types`, `pagerank`, `communities`, `traverse`, `explain_function`, `impact_of`, `symbol_context`, `refresh_repo`, merge-queue tools |
| `code.*` (discovery) | `repo_overview`, `module_tree`, `find_symbol`, `describe_symbol`, `read_symbol`, `find_pattern` |
| `code.*` (traversal) | `find_callers`, `callees_of`, `find_dependencies`, `find_path`, `get_neighborhood`, `references_to`, `co_changes_with`, `find_cycles`, `find_dead_code`, `stats` |
| `code.*` (history) | `recent_changes`, `blame_enriched`, `pr_context`, `pr_history`, `who_knows_about`, `expertise`, `issue_context`, `find_issues`, `discussion_context` |
| `code.*` (tests + CI) | `tests_for`, `coverage_for`, `failing_tests`, `flaky_tests`, `last_run`, `run_failures` |
| `code.*` (deps) | `dependencies`, `dependency_graph` |
| `agent.*` | `start_session`, `heartbeat`, `end_session`, `claim_work`, `release_claim`, `record_finding`, `get_findings`, `list_active_claims` |
| `memory.*` | `recall`, `remember`, `procedure_for`, `forget`, `search`, `context`, `annotate`, `metric_*`, `benchmark_*`, `context_inject` |
| `security.*` | `open_alerts`, `alert_context` |
| `workspace.*` | `list`, `bind`, `enable_memory`, `disable_memory` |
| `incident.*` | `open`, `resolve`, `link` |

Per-tool descriptions live in
[the docs site](https://docs.oxagen.ai/plugins/claude-code-plugin#mcp-tool-reference)
and in the `oxagen:oxagen-tool-reference` skill ([`skills/oxagen-tool-reference/SKILL.md`](./skills/oxagen-tool-reference/SKILL.md)).

---

## How it's wired

```
oxagenai/claude-code-plugin/        # this repo
├── .claude-plugin/
│   ├── marketplace.json            # marketplace listing
│   └── plugin.json                 # plugin manifest (name, version, author, license)
├── .mcp.json                       # MCP server config — points at mcp.oxagen.ai
├── commands/                       # slash-command runbooks
├── hooks/
│   ├── hooks.json                  # SessionStart hook registration
│   ├── session-start               # injects the <EXTREMELY_IMPORTANT> preamble
│   └── run-hook.cmd                # cross-platform polyglot wrapper
├── skills/
│   ├── using-oxagen/SKILL.md       # always-on routing policy
│   ├── oxagen-tool-reference/SKILL.md  # full tool catalog (load on demand)
│   └── dogfooding/SKILL.md         # write-back conventions
├── README.md                       # this file
├── CHANGELOG.md
├── CONTRIBUTING.md
└── LICENSE                         # MIT
```

### Environment

| Variable | Required | Purpose |
|----------|----------|---------|
| `OXAGEN_MCP_TOKEN` | Yes | Bearer token for `mcp.oxagen.ai`. `/oxagen:setup` writes it to `~/.claude/oxagen.local.json`; export it in your shell so `.mcp.json` substitution picks it up. |

### Settings file

```json
{
  "workspace_id": "ws_…",
  "connection_id": "conn_…",
  "token": "oxa_live_…",
  "minted_at": 1714000000
}
```

`~/.claude/oxagen.local.json` is gitignored (`*.local.json`) and chmod'd
to `600` by `/oxagen:setup`.

---

## Troubleshooting

**No tools listed after setup.** Check that `OXAGEN_MCP_TOKEN` is exported
in the shell that launched Claude Code. `echo $OXAGEN_MCP_TOKEN` to
verify, then restart Claude Code from that shell and run `/mcp`.

**`memory.*` tools not returning results.** Memory tools are always
registered; capture only activates after `/oxagen:memory-enable`.

**401 on every call.** Token expired or revoked. Generate a new token at
**Settings → MCP → Tokens** in the app, then re-run `/oxagen:setup`.

**Ingest stuck / graph empty.** Check **Settings → Connections** in the
app. Reconnect GitHub if errored, then trigger a manual re-ingest. Or ask
the agent to call `ontology.refresh_repo`.

**Stale claim blocking another session.** `/oxagen:ask "list active
claims"` to find it, then `/oxagen:claim <file:line> --force` to override
(records an audit event).

**Low confidence from `/oxagen:ask` or `/oxagen:search`.** The graph may
be sparse; trigger a re-ingest via **Settings → Connections → Re-ingest**
and retry once status is `ready`.

---

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). PRs welcome — especially new
slash commands, real-world use-case examples, and adapters for adjacent
agent platforms.

This repo is the canonical, ejectable home of the Oxagen Claude Code
plugin. The platform monorepo at
<https://github.com/oxagenai/oxagen-platform> mirrors this directory under
`plugins/oxagen-claude-code/` for internal dogfooding; ejection happens
via `scripts/eject-claude-code-plugin.sh`.

---

## License

[MIT](./LICENSE) · © 2026 Oxagen, Inc.
