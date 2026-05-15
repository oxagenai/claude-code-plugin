---
name: oxagen-dogfooding
description: Save findings to the Oxagen knowledge graph after every meaningful step. Use whenever you make a non-trivial decision, learn something new about the codebase, or finish a task — so the next agent (or the human) can find it via memory_search.
when_to_load:
  - on file edits in dispatched sandbox sessions
  - on `oxagen go` runs
---

# Dogfooding skill — write to the KG as you work

This skill makes Oxagen-dispatched sessions act like senior engineers
who take notes. It's also the marketing demo: every successful run
proves the product can be the long-term memory layer for an agent
fleet.

## When to call which tool

| Situation | Tool |
| --- | --- |
| You finished the task | `memory.remember` (M8 §6.4 — fact + outcome=succeeded + linked symbol) |
| You made a non-obvious decision (chose A over B) | `agent.record_finding` |
| You discovered something the next agent needs (gotcha, hidden coupling) | `memory.remember` with `outcome=succeeded` and a clear summary, plus an `agent.record_finding` on the symbol |
| You hit a failure and recovered | `memory.remember` with `outcome=failed` + `error_signature` so it can participate in M8 promotion |
| You changed a public API or contract | `agent.record_finding` (typed) plus a `memory.remember` summary |

## Anti-slop rules (enforced by the no-slop quality gate)

The dispatched session **MUST NOT**:

- Introduce speculative abstractions or feature flags
- Add comments that narrate the change ("# added for OXA-123")
- Add try/except that swallows errors silently
- Touch unrelated files (stay in the issue scope)
- Skip tests for new public functions or endpoints
- Exceed 500 LOC net of generated files

If any of these is unavoidable, document why in the PR body and use
`--allow-large-pr` (set by the orchestrator).

## Heartbeat

Long-running sessions should call `agent.heartbeat` every ~5 minutes so
the supervisor knows the work is still in flight and won't reissue the
claim to another agent.
