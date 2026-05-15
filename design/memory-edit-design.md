# /oxagen:memory-edit — Design Notes

**Status:** Not yet implemented. All backend MCP tools exist. UX decisions
needed before building the command.

## What it should do

Allow an operator to view, annotate, suppress, or soft-delete agent memory
entries (patterns and sequences) directly from Claude Code without opening
the app UI.

## Proposed UX

Four sub-commands:

1. `/oxagen:memory-edit patterns` — paginated list of patterns, pick one
   to annotate or suppress
2. `/oxagen:memory-edit sequences [--session <id>]` — show a sequence
   timeline for the current or a named session
3. `/oxagen:memory-edit suppress <pattern-id>` — suppress a pattern
   immediately (skips interactive flow)
4. `/oxagen:memory-edit annotate <pattern-id> "<text>"` — add operator
   annotation without suppressing

## MCP tools needed (all exist today)

| Action | Tool |
|--------|------|
| List patterns | `memory.search` |
| Suppress / annotate | `memory.annotate` |
| View sequences | `memory.search` with outcome filter |
| View context for a node | `memory.context_inject` |

No new backend work required.

## Open questions before implementation

1. **Pattern list UX:** should `/memory-edit patterns` show a paginated
   table (numbered rows, user enters a number) or a search prompt ("type
   a pattern_key prefix to filter")? The search approach scales better
   once pattern counts grow.

2. **Bulk suppress:** should the command support suppressing all patterns
   matching a key prefix? e.g. `suppress dead_end_search:*`. Useful for
   clearing a class of false positives quickly. Backend already supports
   individual suppression; bulk would need a loop or a new API.

3. **Recommendation override:** the `memory.annotate` tool sets
   `operator_annotation`. Should `/memory-edit` also allow overriding the
   structured `recommendation` field? Or should recommendation always come
   from the evaluator and only annotation be operator-editable?

4. **Undo / undelete:** patterns can be suppressed but not easily
   unsuppressed via CLI today. The `memory.annotate` tool accepts
   `suppressed: false` to re-enable — the command should expose this.
   What should the UX be? A confirmation prompt? An explicit
   `/oxagen:memory-edit unsuppress <id>`?

5. **Sequence editing:** sequences are evaluator-written audit records.
   Should operators be able to edit `intent_summary` to correct a
   mis-labelled sequence? Or is that field evaluator-only and operators
   annotate at the pattern level?

## Verdict

Implement once questions 1 and 4 are resolved — those two gate the basic
interactive flow. Questions 2, 3, and 5 can ship as v2 once the core
command is in place. The backend is ready; this is purely a UX design
decision.

## Implementation sketch (once questions resolved)

```markdown
---
description: View and edit agent memory entries — annotate or suppress patterns.
argument-hint: "patterns | sequences | suppress <id> | annotate <id> \"<text>\""
allowed-tools: mcp__oxagen__memory_search, mcp__oxagen__memory_annotate
---
```

Dispatch on `$1`:
- `patterns` → fetch + list → pick → annotate/suppress
- `sequences` → fetch + display timeline
- `suppress <id>` → call `memory.annotate({ pattern_id, suppressed: true })`
- `annotate <id> "<text>"` → call `memory.annotate({ pattern_id, annotation })`
