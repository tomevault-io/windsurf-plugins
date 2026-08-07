---
trigger: always_on
description: Grimoire is a private, pre-release Obsidian plugin that embeds agentic CLI assistants in a vault-native workspace. It is not a standalone CLI. The plugin shell must stay provider-neutral while provider adapters wrap external tools such as Claude Code, Codex, OpenCode, Qwen Code, and Antigravity CLI.
---

# Agent Instructions

Grimoire is a private, pre-release Obsidian plugin that embeds agentic CLI assistants in a vault-native workspace. It is not a standalone CLI. The plugin shell must stay provider-neutral while provider adapters wrap external tools such as Claude Code, Codex, OpenCode, Qwen Code, and Antigravity CLI.

Repository documentation and user-facing product copy should be in English unless the task explicitly targets localized UI text.

## Instruction Layout

- `AGENTS.md` is the canonical shared instruction file for coding agents.
- `CLAUDE.md` files exist so Claude Code can load the same instructions. They should import the nearest `AGENTS.md` and contain only Claude-specific additions.
- Keep root instructions durable. Put path-specific details in nested `AGENTS.md` files next to the code they govern.
- If a design handoff directory is named by the user, treat it as the source of truth for that task. Keep temporary handoff/debug artifacts untracked unless the user explicitly asks to commit them.

## Provider Directories

- `src/providers/claude/` - Claude Code SDK adapter and Claude-compatible vault files.
- `src/providers/codex/` - Codex app-server adapter and Codex-owned workspace services.
- `src/providers/antigravity/` - Antigravity CLI print-mode adapter and Google's official Gemini CLI replacement.
- `src/providers/gemini/` - Legacy Gemini CLI ACP adapter and Google-owned runtime, history, settings, and UI behavior.
- `src/providers/grok/` - Grok Build ACP adapter and xAI-owned runtime, history, settings, and UI behavior.
- `src/providers/opencode/` - OpenCode ACP adapter and launch/workspace artifacts.
- `src/providers/mimocode/` - MiMoCode ACP adapter and launch/workspace artifacts.
- `src/providers/kimicode/` - Kimi Code ACP adapter and launch/workspace artifacts.
- `src/providers/qwen/` - Qwen Code ACP adapter and Qwen-owned runtime, history, settings, and UI behavior.
- `src/providers/acp/` - Shared ACP transport and normalization helpers.

Read the nested `AGENTS.md` in a provider directory before changing provider-specific runtime, storage, history, settings, or UI behavior.
OpenCode and MiMoCode intentionally mirror each other closely; when changing launch, ACP runtime, workspace, storage, history, settings, or UI behavior in one provider, check and usually apply the same change to the other provider unless the CLIs intentionally differ.

## Architecture Rules

- Keep `src/core/` provider-neutral. Shared chat/runtime/settings contracts belong there only when at least two providers need the behavior.
- Keep provider-specific protocol, storage, CLI resolution, history parsing, model discovery, settings UI, and launch artifacts inside `src/providers/<provider>/`.
- Register provider runtime and auxiliary services through `ProviderRegistry`.
- Register provider workspace services through `ProviderWorkspaceRegistry`.
- Feature code must consume provider-neutral contracts. Do not read provider-specific `Conversation.providerState` fields directly from `src/features/`.
- Preserve provider-native behavior first. Prefer adapting official CLI/runtime semantics over reimplementing provider features inside Grimoire.
- Use `.grimoire/` for Grimoire-owned vault data. Do not add legacy storage migration behavior unless a migration milestone explicitly asks for it.

## Key Paths

| Path | Purpose |
|------|---------|
| `src/main.ts` | Obsidian plugin entry point, view registration, commands, lifecycle |
| `src/app/` | Settings defaults and plugin-level storage helpers |
| `src/core/` | Provider-neutral runtime, providers, MCP, security, storage, tools, shared types |
| `src/providers/` | Provider adapters and provider-owned services |
| `src/features/chat/` | Main sidebar chat interface and tab lifecycle |
| `src/features/inline-edit/` | Inline edit modal and provider-backed edit services |
| `src/features/settings/` | Shared settings shell plus provider-owned settings tabs |
| `src/shared/` | Reusable UI components, modals, mention UI, icons |
| `src/style/` | Modular CSS, built into root `styles.css` |
| `tests/` | Unit and integration tests mirroring `src/` |

## Commands

```bash
npm ci
npm run dev
npm run build
npm run build:release
npm run typecheck
npm run lint
npm run lint:fix
npm run test
npm run test -- --selectProjects unit
npm run test -- --selectProjects integration
```

Use this full local gate before publishing or pushing meaningful UI/provider changes:

```bash
npm run test -- --selectProjects unit
npm run typecheck
npm run lint
npm run build:release
```

`npm run build:release` refreshes generated `main.js`, root `styles.css`, and `dist/grimoire`. Generated release artifacts must match source output after the build.

When bumping the plugin version, update `package.json`, `package-lock.json`, `manifest.json`, and `versions.json` together. `versions.json` maps each released plugin version to the minimum supported Obsidian app version and must include the new release before tagging or publishing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandsaber/Grimoire](https://github.com/sandsaber/Grimoire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
