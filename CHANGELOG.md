# Changelog

All notable changes to the Oxagen Claude Code plugin are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versioning follows [SemVer](https://semver.org/spec/v2.0.0.html) and tracks
the parent platform release.

## [Unreleased]

### Added
- Repo extracted into a standalone, ejectable tree at
  `oxagenai/claude-code-plugin`. The platform monorepo at
  `oxagenai/oxagen-platform` continues to mirror it under
  `plugins/oxagen-claude-code/` for internal dogfooding.
- `.claude-plugin/marketplace.json` so the repo can be added directly via
  `/plugin marketplace add oxagenai/claude-code-plugin`.
- SessionStart hook now injects an `<EXTREMELY_IMPORTANT>` preamble that
  pins three guarantees on every turn — *graph first*, *memory always
  recalled*, *coordination respected* — covering non-coding surfaces
  (incidents, security, customer impact, research) in addition to code.
- `using-oxagen` routing skill rewritten to fire on memory/ops/incident
  triggers in addition to code-edit intent.
- `CONTRIBUTING.md` with eject + sync instructions.
- `scripts/eject-claude-code-plugin.sh` (in the platform repo) that
  packages this directory for push to `oxagenai/claude-code-plugin`.

### Changed
- `plugin.json` and `marketplace.json` updated with full marketplace
  metadata (homepage, repository, documentation, issues, license,
  category, keywords) for Claude Code plugin marketplace submission.

## [0.21.6] — 2026-05-12

### Added
- `ontology.symbol_context` MCP registration; `/oxagen:symbol-context`
  command will route here in a follow-up.
- `code.read_symbol` `context_lines` parameter (default 5, range 0–200)
  via PR #608.

## [0.21.0] — 2026-05-01

- Initial public release of the Claude Code plugin from the platform
  monorepo. 40+ slash commands across discovery, traversal, history,
  tests, CI, security, memory, and coordination surfaces.
