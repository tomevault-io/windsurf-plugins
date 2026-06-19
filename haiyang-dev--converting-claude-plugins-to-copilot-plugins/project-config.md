---
trigger: always_on
description: Use when: converting a Claude Code plugin into a GitHub Copilot CLI plugin, especially when plugin manifests, skills directories, marketplace files, and path-sensitive references must be translated safely.
---


# Converting Claude Plugins To Copilot Plugins

## Overview

Convert a Claude Code plugin directory into a GitHub Copilot CLI plugin directory.

The output is a sibling directory named by appending `-copilot` to the source directory name.

**Primary goal:** preserve the plugin structure while translating Claude-specific file locations into Copilot CLI plugin locations safely.

Use the standard Copilot plugin layout with `plugin.json` at the target root. Copilot can also discover manifests in `.github/plugin/plugin.json` or `.claude-plugin/plugin.json`, but this skill emits the root manifest because that is the documented default plugin structure.

## When to Use

Use this skill when:

- A user gives you a Claude Code plugin folder and wants a GitHub Copilot CLI plugin as output.
- The source plugin uses Claude plugin conventions such as an optional `.claude-plugin/plugin.json`, an optional `.claude-plugin/marketplace.json`, and root-level component directories such as `skills/`, `agents/`, or `hooks/`.
- Path handling is the risky part: manifest locations, marketplace files, mixed separators, relative references, or Windows path normalization.
- The user wants the final output in a sibling directory named `<source>-copilot`.

Do not use this skill when:

- The user only wants a project-level Copilot skill under `.github/skills/`.
- The source directory does not look like a Claude plugin.
- The task is unrelated to plugin conversion.

## Procedure

1. Ask the user for the Claude Code plugin root folder.
2. Normalize the source path and derive the sibling `-copilot` output path.
3. Validate the Claude plugin structure before writing anything.
4. Create a Copilot plugin root with a root-level `plugin.json` manifest, translating source metadata when present and deriving only the minimum required metadata when it is not.
5. Copy and translate plugin components using the official Copilot plugin file locations.
6. Rewrite path-sensitive references conservatively.
7. Generate a conversion report that lists copied, translated, skipped, and ambiguous items.

## Claude Source Plugin Structure

### Claude Input Structure

Per the Claude plugin documentation, expect a source tree like this:

```text
my-plugin/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── agents/
├── commands/
├── hooks/
│   └── hooks.json
├── skills/
│   └── <skill>/
│       └── SKILL.md
├── .mcp.json
├── .lsp.json
└── settings.json
```

The `.claude-plugin/` directory contains metadata such as an optional `plugin.json` and, optionally, `marketplace.json`. Component directories such as `skills/`, `agents/`, `commands/`, and `hooks/` still belong at the plugin root.

Do not require the Claude manifest as the sole indicator that the source is a Claude plugin. A source tree can still be a valid Claude-style plugin package if it lacks `.claude-plugin/plugin.json` but contains recognizable plugin components in the documented root locations.

## GitHub Copilot Plugin Structure

### Copilot Output Structure

Per the GitHub Copilot CLI plugin docs, produce a plugin tree like this:

```text
my-plugin-copilot/
├── plugin.json
├── agents/
├── commands/
├── hooks/
│   └── hooks.json
├── skills/
│   └── <skill>/
│       └── SKILL.md
├── .mcp.json
├── lsp.json
├── README.md            # optional copied docs
├── LICENSE              # optional copied docs
└── .github/
    └── plugin/
        └── marketplace.json   # optional
```

The required Copilot plugin manifest is `plugin.json` at the plugin root.

`marketplace.json` is not required for a valid Copilot plugin. It is only needed when the user wants marketplace distribution metadata or when the source Claude plugin already includes marketplace metadata that should be translated.

Use `hooks/hooks.json` as the default file-based hooks location. Only emit a root `hooks.json` if the source already uses a root hooks file or the user explicitly asks for the flat layout.

## Shared Path Rules

### Normalize Input First

When the user gives you a source path:

1. Remove wrapping single or double quotes.
2. Expand `.` and `..`.
3. Convert the path to an absolute path.
4. Normalize separators for internal computation.
5. On Windows, compare normalized paths case-insensitively.

When writing rewritten documentation paths back out, prefer `/` separators for consistency.

Keep two explicit variables in your reasoning:

- `SOURCE_ROOT`: normalized Claude plugin root
- `TARGET_ROOT`: `dirname(SOURCE_ROOT) / (basename(SOURCE_ROOT) + "-copilot")`

For every file you copy, derive the target path from that file's normalized path relative to `SOURCE_ROOT`. Do not build target paths from ad hoc string concatenation or partially normalized fragments.

### Script, Command, And Documentation Environment Variable Rules

Handle script paths, command strings, and copied documentation separately from generic prose rewrites.

Claude hooks explicitly document path-sensitive environment variables such as:

- `$CLAUDE_PROJECT_DIR` for project-root-relative scripts
- `${CLAUDE_PLUGIN_ROOT}` for plugin-root-relative scripts bundled with a plugin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haiyang-dev/converting-claude-plugins-to-copilot-plugins](https://github.com/haiyang-dev/converting-claude-plugins-to-copilot-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
