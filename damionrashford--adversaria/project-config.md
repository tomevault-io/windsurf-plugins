---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Adversaria is a **Claude Code plugin**, not an application. There is no build step, no bundler, and no dependency manifest. Every file is one of three things: a JSON manifest, a Markdown instruction file that Claude reads at runtime, or a TypeScript file run directly by Bun.

The consequence: **the Markdown *is* the product.** Editing `agents/*.md` or `skills/*/SKILL.md` changes behavior as directly as editing code. Treat prose changes in those files with the same care as a code change, and re-read the frontmatter contract below before touching it.

The plugin is distributed through a public marketplace and runs on other people's machines, inside their projects. Nothing in this repo may hard-code an absolute path, assume a particular operating system or directory layout, or carry information specific to one contributor. Paths resolve through `${CLAUDE_PLUGIN_ROOT}` and `${CLAUDE_SKILL_DIR}` (see *Path contract*).

## Development tooling

[.claude/settings.json](.claude/settings.json) enables `plugin-dev`, Anthropic's official plugin-authoring toolkit, for anyone working in this repo. It is the authority on plugin structure — prefer it over recalling conventions from memory, since the plugin spec changes.

It loads automatically once the official marketplace is known locally. If Claude Code reports the marketplace is missing:

```
/plugin marketplace add anthropics/claude-plugins-official
```

| Component | Use it when |
| :--- | :--- |
| `plugin-dev:plugin-validator` (agent) | Before any commit that touches `.claude-plugin/`, `hooks/hooks.json`, or agent/skill frontmatter. Validates manifests and file layout. |
| `plugin-dev:skill-reviewer` (agent) | After editing any `SKILL.md`, especially the `description` — that field is what makes the skill auto-trigger. |
| `plugin-dev:agent-creator` (agent) | Adding a fourth agent, or reworking an existing agent's frontmatter. |
| `/plugin-dev:create-plugin` (command) | Scaffolding only. This plugin already exists; rarely needed here. |
| `plugin-dev:plugin-structure` (skill) | Questions about directory layout and manifest fields. |
| `plugin-dev:skill-development` (skill) | Writing or restructuring a skill, including `references/` and progressive disclosure. |
| `plugin-dev:agent-development` (skill) | Agent frontmatter fields: `model`, `tools`, `skills`, `memory`, `effort`, `maxTurns`. |
| `plugin-dev:hook-development` (skill) | Anything in `hooks/` — event names, matchers, exit-code contract, `${CLAUDE_PLUGIN_ROOT}`. |
| `plugin-dev:command-development` (skill) | Slash-command surface and `argument-hint`. |
| `plugin-dev:mcp-integration` (skill) | Only if an MCP server is ever added. There is none today. |
| `plugin-dev:plugin-settings` (skill) | `settings.json` keys and plugin enablement scopes. |

`plugin-dev` ships **no tools and no MCP server** — it contributes instructions only, so it cannot break this plugin at runtime.

For Claude Code's own behavior (hook events, settings keys, plugin loading, agent frontmatter), check the official docs at https://code.claude.com/docs rather than asserting harness behavior from memory. These surfaces change between releases, and a stale assumption here ships to every installed user.

## Commands

Run the reasoning engines directly to test them, exactly as the skills do:

```bash
bun scripts/argument-graph.ts add "cost-reduction" claim --obs "cuts spend 40%" --agent advocate
bun scripts/argument-graph.ts add "vendor-lockin" weakness --obs "single-provider dependency" --agent adversary
bun scripts/argument-graph.ts relate "vendor-lockin" undermines "cost-reduction" --agent adversary
bun scripts/argument-graph.ts lock      # seal the record (appeal); reset + edits to prior testimony refuse
bun scripts/argument-graph.ts unlock
bun scripts/argument-graph.ts analyze
bun scripts/argument-graph.ts verdict
bun scripts/argument-graph.ts export
bun scripts/argument-graph.ts transcript
bun scripts/argument-graph.ts show
bun scripts/argument-graph.ts reset
```

```bash
bun scripts/thought-chain.ts init "the subject" --total 7
bun scripts/thought-chain.ts think 1 "<content>"
bun scripts/thought-chain.ts revise 1 "<content>" --reason "<why>"
bun scripts/thought-chain.ts branch "<id>" 1 2 "<content>"
bun scripts/thought-chain.ts status
bun scripts/thought-chain.ts render
bun scripts/thought-chain.ts reset
```

```bash
bun scripts/transcript.ts serve --port 7391 --no-open
```

Run the tests — this is the one place a change can be verified automatically:

```bash
bun test
```

**No dependency manifest, ever.** Bun runs `.ts` directly and `bun test` works with no `package.json`, no `node_modules`, and no lockfile. Adding any of them breaks skill loading. There are no third-party imports anywhere in this repo, and there must not be.

Load the working tree as a plugin to exercise agents, skills, and hooks end to end:

```bash
claude --plugin-dir /path/to/this/repo
```

Verify what actually loaded, and inspect enablement scopes:

```bash
claude plugin list
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damionrashford/Adversaria](https://github.com/damionrashford/Adversaria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
