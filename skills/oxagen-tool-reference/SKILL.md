---
name: oxagen-tool-reference
description: Full catalog of Oxagen MCP tools — code-graph (discovery, traversal, history, tests, CI), ontology typed-traversal, security, agent coordination, memory, and general-purpose. Load on demand from the using-oxagen gate skill when a specific tool surface is needed.
---

# Oxagen tool reference

The using-oxagen gate skill describes *when* to reach for the graph. This
skill is the *catalog* — load it when you need to pick the exact tool for
a given task. Keep this open in context only as long as you're routing;
it's reference material, not policy.

## Code-graph tools — discovery and structure

| Tool | When to call |
|------|---|
| `code.repo_overview` | First call in a new workspace — language mix, top modules, entry points. |
| `code.module_tree` | Drill into a directory — annotated subtree with symbol counts. |
| `code.find_symbol` | Resolve a partial / qualified name to ontology nodes before any traversal. |
| `code.describe_symbol` | Structural metadata — kind, signature, decorators, docstring. |
| `code.read_symbol` | Source bytes + summary + line range with configurable surrounding context (`context_lines`, default 5, range 0–200). |
| `code.find_pattern` | Hybrid semantic + grep search across the ontology. |

## Code-graph tools — traversal

| Tool | When to call |
|------|---|
| `code.find_callers` (alias `code.callers_of`) | Anyone calling this symbol? Before any non-trivial edit. |
| `code.callees_of` | What does this symbol call? |
| `code.find_dependencies` | What does this symbol need? Before deletion or refactor. |
| `code.find_path` (alias `code.dependency_path`) | "How does A reach B?" — directed path in the call graph. |
| `code.get_neighborhood` | Unfamiliar code — one-hop callers + callees + sibling symbols. |
| `code.references_to` | Broader than callers — calls + imports + type uses. |
| `code.co_changes_with` | Hidden coupling — symbols that historically change together. |
| `code.stats` | Repo-level questions: file count, lines, language mix, top packages. |
| `code.find_dead_code` | Audit / cleanup tasks. Symbols with zero callers. |
| `code.find_cycles` | Architecture review. Strongly connected components in the call graph. |
| `ontology.impact_of` (alias `code.affected_by`) | Aggregate caller + dependent view — what's affected? |
| `ontology.symbol_context` | One-call laser bundle — class, siblings, tests, callers, callees, returns, recent commits. Replaces 5+ traversal calls. |

## Code-graph tools — history, ownership, issues

| Tool | When to call |
|------|---|
| `code.recent_changes` | What changed across the workspace recently? |
| `code.blame_enriched` | Line-level blame with PR / issue / reviewer joins. |
| `code.pr_context` | Full PR details without leaving the editor. |
| `code.pr_history` | PRs that touched a specific symbol. |
| `code.who_knows_about` | Who's an expert on this symbol? |
| `code.expertise` | What does this person own? |
| `code.issue_context` | Pull issue title, body, comments, linked PRs, affected nodes. |
| `code.find_issues` | Hybrid full-text issue search. |
| `code.discussion_context` | Pull GitHub Discussion context. |

## Code-graph tools — tests and CI

| Tool | When to call |
|------|---|
| `code.tests_for` | Which tests exercise this symbol? |
| `code.coverage_for` | Per-line coverage for a file. |
| `code.failing_tests` | What's currently red on the branch? |
| `code.flaky_tests` | Flake-rate ranking over a window. |
| `code.last_run` | Most-recent CI run for a workflow / branch / PR. |
| `code.run_failures` | Failed jobs and steps for a run with cold-log preview. |

## Security tools

| Tool | When to call |
|------|---|
| `security.open_alerts` | Triage open Dependabot, CodeQL, and secret-scan alerts. |
| `security.alert_context` | Full alert body — affected nodes, recommendation, advisories. |

## Ontology tools — typed traversal

| Tool | When to call |
|------|---|
| `ontology.list_nodes` | Filter by `node_type` for typed slices (e.g. all `Function` nodes in a module). |
| `ontology.list_edges` | Typed edges (`CALLS`, `IMPORTS`, `EXPORTS`, `DEFINES`). |
| `ontology.get_node` | Inspect a single node and its properties. |
| `ontology.list_types` | Discover what node/edge types exist in this workspace. |
| `ontology.pagerank` | Score nodes by structural importance — sort callers/deps by PageRank. |
| `ontology.communities` | Discover module clusters. |
| `ontology.create_node`, `ontology.create_edge` | Write tools — require explicit user confirmation. Never auto-call. |

## Agent coordination tools — multi-agent work

Multiple Oxagen-connected agents may operate against the same workspace.
The `agent.*` surface keeps them from stepping on each other and lets
them share findings.

| Tool | When to call |
|------|---|
| `agent.start_session` | First call after `/oxagen:setup` confirms wiring. Registers this Claude Code instance for the workspace. |
| `agent.heartbeat` | Liveness ping — the server reaps stale sessions. Call at the start of each new user turn during a long session. The setup wizard / shell hook owns timer-driven pings; the model only fires this when it's mid-turn anyway. |
| `agent.end_session` | When the user closes Claude Code or finishes the task. Releases all claims held by this session. |
| `agent.claim_work` | Before editing a file or function, claim a line range (or the whole symbol). Other sessions see the claim and back off. |
| `agent.release_claim` | After committing or abandoning the change. |
| `agent.list_active_claims` | Before claiming, check what's already locked. Before editing, check whether someone else is in this code. |
| `agent.record_finding` | Persist a typed finding ("this caller breaks if signature changes", "this test asserts the old behaviour") for other sessions to read. |
| `agent.get_findings` | At task start, read findings on the symbols you're about to touch — pick up where another session left off. |

## General-purpose tools

| Tool | When to call |
|------|---|
| `cypher.run` | Ad-hoc graph queries the typed tools don't cover. NL→Cypher. |
| `reads.search` | Keyword fallback when NL→Cypher confidence is low. |
| `dedupe.find_similar` | Before adding a new helper — confirm an existing one doesn't exist. |
| `viz.*` | Render a subgraph for the user (only when they ask to see it). |
| `web.fetch` | External docs — not workspace knowledge. |

## Agent memory — durable self-improvement

When agent memory is enabled on a workspace, every MCP tool call is
automatically captured as a `_mem:action` node. The evaluator groups
actions into sequences, detects recurring failure patterns, and writes
nightly benchmarks that quantify whether the agent is improving over
time.

### Memory MCP tools

Once memory is enabled these tools are always available:

| Tool | When to use |
|------|---|
| `memory.recall` | Primary M8 retrieval — procedures + episodes by intent, with optional symbol / outcome anchors. |
| `memory.remember` | Persist an episode anchored to the active session (and optionally a symbol / error signature). |
| `memory.procedure_for` | Look up a procedure by exact trigger pattern. |
| `memory.forget` | Soft-delete an episode or procedure. |
| `memory.context_inject` | Before acting on a set of node IDs — injects relevant past-failure patterns into context. |
| `memory.search` | Search recent patterns/sequences by action_type, outcome, or agent_id. |
| `memory.annotate` | Suppress a false-positive pattern or add an operator note. |
| `memory.metric_create` | Create a custom metric (prefer `/oxagen:memory-metric` for interactive flow). |
| `memory.benchmark_get` | Retrieve benchmark trend for a metric — "is the agent improving?". |

### When to use memory tools

- **Before acting on a high-risk node** — call `memory.context_inject`
  with the target node IDs. If past sessions failed here, the server
  injects a structured warning into context automatically (confidence
  ≥ 0.6 threshold).
- **After an unexpected failure** — call `memory.search` to see if this
  is a recurring pattern. If yes, `memory.annotate` it with a note for
  future sessions.
- **At session start in a new workspace** — `memory.context_inject` on
  the files / nodes you're about to work in; lets you start from where
  past agents left off.

### Memory slash commands (for the user, not the agent)

| Command | What it does |
|---------|---|
| `/oxagen:recall` | Retrieve procedures + episodes by intent. |
| `/oxagen:remember` | Persist a fact as a memory episode. |
| `/oxagen:procedure-for` | Look up a stored procedure by exact trigger pattern. |
| `/oxagen:forget` | Soft-delete an episode or procedure. |
| `/oxagen:memory-enable` | Enable memory on the connected workspace — seeds 3 built-in metrics. |
| `/oxagen:memory-metric` | Define a custom eval metric. |
| `/oxagen:memory-logs` | View recent agent actions captured by the memory layer. |
| `/oxagen:workspace-select` | Switch which workspace the agent session uses (MCP-only installs). |

### Automatic capture — no explicit calls required

Every `code.*`, `ontology.*`, and `agent.*` tool call is automatically
captured by the MCP server's action capture hook when memory is enabled.
You do not need to call any `memory.*` write tools yourself — the
evaluator handles grouping, pattern detection, and benchmarks
asynchronously.

## Cost discipline

Each `code.*` call is a single round-trip. The plugin is engineered to
make these cheap. The cost of *not* calling them — shipping a signature
change that breaks ten upstream callers — is much higher.
