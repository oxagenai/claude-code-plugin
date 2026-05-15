# Contributing

Thanks for considering a contribution. The Oxagen Claude Code plugin is
the user-facing surface of the Oxagen ontology platform inside Claude
Code; changes here ship to every developer who installs the plugin from
the marketplace.

## Where the canonical source lives

The plugin is developed inside the platform monorepo at
[`oxagenai/oxagen-platform`](https://github.com/oxagenai/oxagen-platform)
under `plugins/oxagen-claude-code/`. The standalone repo at
[`oxagenai/claude-code-plugin`](https://github.com/oxagenai/claude-code-plugin)
is a mirror, refreshed on every release via
`scripts/eject-claude-code-plugin.sh` in the monorepo.

**Make changes in the monorepo PR, not in the mirror.** PRs opened
against the mirror are accepted but will be replayed into the monorepo
before being merged.

## Local development

1. Clone the platform monorepo.
2. Symlink the plugin tree into Claude Code's plugin path:
   ```bash
   ln -s "$(pwd)/plugins/oxagen-claude-code" ~/.claude/plugins/oxagen
   ```
3. Restart Claude Code. The SessionStart hook fires; `/mcp` should list
   `oxagen` once `OXAGEN_MCP_TOKEN` is exported.
4. Edit, reload Claude Code, repeat.

## Adding a slash command

1. Create `commands/<name>.md` with frontmatter:
   ```yaml
   ---
   description: Short, single-line summary the agent surfaces in `/help`.
   argument-hint: "<arg>"
   allowed-tools: mcp__oxagen__<tool>, Bash, Read, Write
   ---
   ```
2. Body is the runbook the agent follows when the command fires.
   Numbered steps, explicit tool calls, expected response formats,
   failure-mode table at the end. Use existing commands like
   [`commands/setup.md`](./commands/setup.md) and
   [`commands/ask.md`](./commands/ask.md) as templates.
3. Add the command to the appropriate table in `README.md` and in the
   docs site (`services/docs/content/docs/plugins/claude-code-plugin.mdx`
   in the platform monorepo).
4. Add a Linear issue under the Plugins project, link it in the PR.

## Editing the routing skill

`skills/using-oxagen/SKILL.md` is loaded into the system prompt on every
session start. Changes affect every user instantly. Be conservative:
prefer additive activation triggers over removing existing ones.

## Bumping the version

Versions are coordinated with the platform monorepo. The
`pnpm release:*` script in the monorepo bumps every package; the plugin's
`plugin.json` is included. Don't bump the version manually.

## Eject + push to the standalone repo

From the monorepo root, after a release merges to `main`:

```bash
scripts/eject-claude-code-plugin.sh /tmp/claude-code-plugin-eject
cd /tmp/claude-code-plugin-eject
git remote add origin git@github.com:oxagenai/claude-code-plugin.git
git push -u origin main
```

The script copies a clean tree, initializes git, commits with a
release-stamped message, and prints the next-step `git remote add` /
`git push` commands.

## Reporting issues

- Bugs / feature requests: <https://github.com/oxagenai/claude-code-plugin/issues>
- Security issues: <security@oxagen.ai> (do not file public issues for
  CVEs or token leaks).

## Code of conduct

Be excellent to each other. Brand voice and tone for any public-facing
copy follow `services/internal/content/docs/reference/brand-voice.mdx` in
the platform monorepo: technically precise, declarative, numerically
honest, infrastructure-calm, developer-respectful.
