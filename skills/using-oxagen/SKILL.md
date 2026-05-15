---
name: using-oxagen
description: Always-on routing policy for the Oxagen plugin. Use whenever the user asks a substantive question about the workspace, requests non-trivial changes to artifacts the workspace tracks (code, docs, runbooks, contracts, tickets, incidents), wants memory recall, or coordinates with other agents on the same workspace. Pairs with oxagen-tool-reference (full catalog) and oxagen-dogfooding (write-back).
---

# Using Oxagen — when to reach for the graph and memory

The workspace this session is acting on is already a typed, queryable
knowledge graph with durable agent memory layered on top. The graph knows
what calls what, what depends on what, who owns what, what's dead, what
cycles, what changed when, and which past sessions failed where. Use it.
Default to graph + memory queries before reasoning from name matches, file
contents, or your own prior context — the graph is deterministic, you are
not.

## The strong rules

### Rule 1 — Recall memory at the start of any non-trivial task

Before acting on a substantive request, call `memory.recall` (or
`memory.context_inject` on the target node IDs) with the user's intent. If
a procedure or episode matches, prefer it over rediscovery. After
meaningful work, write back via `memory.remember` and
`agent.record_finding` so the next session inherits it.

This applies to code, but also to: incident triage, customer-impact
analysis, runbook execution, security review, doc edits, research synthesis
— anything the workspace ontology tracks.

### Rule 2 — Graph first before any non-trivial edit

Before any of these on existing artifacts, call the relevant Oxagen tools
— no exceptions:

- Code: changing a function or method signature; changing observable
  behaviour of a public symbol; deleting a function, class, or method;
  renaming an exported symbol; moving a symbol between modules → call
  `code.find_callers` and `code.find_dependencies` (or one-shot
  `ontology.symbol_context`).
- Docs / runbooks: editing a procedure other artifacts reference → call
  `code.find_pattern` or `ontology.traverse` to find back-references.
- Incidents / tickets: linking, escalating, resolving → call
  `incident.link` and `code.find_issues` to surface adjacent context.
- Security alerts: triaging or suppressing → call `security.alert_context`
  to surface affected nodes before action.

Skipping is acceptable only for: comment / docstring / whitespace-only
changes; renames `grep` confirms are file-local and unexported; edits inside
a brand-new artifact no other artifact references.

Surfacing upstream impact *is* the value of this plugin. Don't optimize for
tokens on the wrong side of the ledger — shipping a change that breaks ten
upstream callers, escalates the wrong incident, or contradicts a stored
runbook is the expensive outcome.

## Activation signals — fire on any of these

- **Edit intent on an entity the workspace tracks** — refactors, signature
  changes, behaviour changes, deletions, renames of exported symbols, moves
  between modules, doc rewrites, runbook updates, ticket transitions,
  contract amendments.
- **Question forms** — "what calls this?", "what does this depend on?",
  "what's the impact of changing this?", "if I delete/rename X, what
  breaks?", "who owns this?", "what's the latest on incident N?", "what
  customers are affected by alert M?", "what does our runbook say about
  Q?", "has this come up before?".
- **Graph terminology** — "callers", "callsites", "dependencies",
  "dependents", "blast radius", "impact", "reach", "cycles", "dead code",
  "the graph", "the ontology", "the workspace graph", "the knowledge
  graph".
- **Memory terminology** — "remember", "recall", "last time", "we've seen
  this before", "our procedure for", "is there a runbook", "did we already
  fix this".
- **Multi-agent coordination** — "while I'm doing X another session is
  doing Y", "lock these lines", "what claims are active?", "what did the
  other agent find?". Use the `agent.*` surface (claims, findings,
  heartbeat) so concurrent sessions don't collide.

## Decision flow per task type

| Task | Call sequence |
|------|---|
| Edit function `foo` | `memory.recall("editing foo")` → `code.find_callers(foo)` → `code.find_dependencies(foo)` → if signature changes, surface callers in the response *before* the user approves → after editing, list callers that need follow-up + `memory.remember` the outcome |
| "Why is this slow?" | `ontology.pagerank` over `node_type=Function` → `code.find_path` from entry to bottleneck → `code.get_neighborhood` for context → `memory.recall("slow path")` for prior diagnoses |
| "Find duplicate code or content" | `dedupe.find_similar(<snippet>)` first; recommend reuse over reimplementation |
| Audit dead code | `code.find_dead_code` → cross-check `EXPORTS` edges → only recommend removing nodes that are dead AND unexported |
| Add a new helper / runbook / doc | `dedupe.find_similar(<intended behaviour>)` first; if a match exists, extend not duplicate |
| "Find <thing> in the workspace" | `ontology.ask("<NL query>")` first; fall back to `code.find_pattern` only if confidence is low |
| "What changed recently?" | `code.recent_changes` → `code.pr_history` on the affected nodes → `memory.search` for related captured actions |
| Edit in a workspace with parallel agents | `agent.list_active_claims` → `agent.get_findings` on target nodes → `agent.claim_work` the range → edit → `agent.record_finding` → `agent.release_claim` |
| Triage a production incident | `incident.open` → `code.find_callers` on the suspected symbol → `code.find_path` from entrypoint → `memory.recall("incident in <area>")` for prior root causes → on resolve `incident.resolve` + `memory.remember` |
| Review a security alert | `security.alert_context(<id>)` → `code.find_callers` on affected nodes → `memory.recall("alert <type>")` for prior triage outcomes |
| Customer-impact question | `ontology.ask("which customers depend on <feature>")` → `code.find_callers` on the symbol → cross-reference with the workspace's customer ontology |
| Onboarding question / "how do we…" | `memory.procedure_for(<trigger>)` → `code.find_issues(<topic>)` for prior discussions → `code.who_knows_about(<area>)` to surface the human expert |

For the full tool catalog (history, tests, CI, security, ontology
typed-traversal, and the agent coordination surface), invoke the
`oxagen:oxagen-tool-reference` skill. For write-back behaviour after
finishing meaningful work — `memory.remember`, `agent.record_finding` —
invoke the `oxagen:dogfooding` skill.

## When the graph contradicts grep / your prior context

The graph is the source of truth for entity resolution and edges. If
`grep` finds a string match the graph doesn't, the match is likely a
comment, a string literal, or a dead reference — trust the graph. If the
graph is missing a recently-added entity, the ingest hasn't run yet —
surface that to the user rather than silently falling back to other
sources. If your in-context memory contradicts a graph result from this
session, the graph wins.

## Writes require explicit confirmation

`ontology.create_node`, `ontology.create_edge`, and any future
`ontology.update_*` / `ontology.delete_*` are user-confirm tools. The MCP
server enforces the confirm protocol; never auto-confirm. Surface the
confirm prompt verbatim to the user.

## Output expectations

- Always cite `file:line` (or canonical entity reference) for nodes. The
  graph stores it; use it.
- Sort callers and dependencies by PageRank when the result set > 5.
- For long lists, show top-10 + a count of remaining.
- For paths, number the hops: `(1) entry → (2) middleware → (3) target`.
- Quote actual node names from the graph, not paraphrases.
- Cite memory hits explicitly: "Memory recall (procedure `id`, confidence
  0.82): …" — the user should see when a past session is being relied on.
