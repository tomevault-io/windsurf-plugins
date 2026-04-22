---
trigger: always_on
description: ├── cli/                    # CLI entrypoint and commands
---

# Repository Guidelines

## Project Structure

```
src/
├── cli/                    # CLI entrypoint and commands
│   ├── index.ts           # Main CLI entry
│   ├── commands/          # create, update, remove, doctor, etc.
│   └── args.ts            # Argument parsing
├── tui/                    # Ink-based TUI wizard
│   ├── app.tsx            # Main TUI application
│   ├── screens/           # Individual screen components
│   ├── components/        # Reusable UI components
│   ├── hooks/             # React hooks for business logic
│   ├── state/             # State types and management
│   └── router/            # Screen navigation
├── core/                   # Core variant management
│   ├── index.ts           # Public API (createVariant, updateVariant, etc.)
│   ├── variant-builder/   # Step-based variant creation
│   │   ├── VariantBuilder.ts
│   │   ├── VariantUpdater.ts
│   │   ├── steps/         # Build steps (PrepareDirectories, InstallNative, etc.)
│   │   └── update-steps/  # Update steps
│   ├── prompt-pack/       # System prompt overlays
│   │   ├── providers/     # Per-provider overlays (zai.ts, minimax.ts)
│   │   ├── overlays.ts    # Overlay resolution
│   │   └── targets.ts     # Target file mapping
│   └── *.ts               # Utils (paths, fs, tweakcc, skills, etc.)
├── providers/              # Provider templates
│   └── index.ts           # Provider definitions and defaults
├── brands/                 # TweakCC brand presets
│   ├── index.ts           # Brand resolution
│   ├── types.ts           # TweakCC config types
│   ├── zai.ts             # Z.ai theme + blocked tools
│   ├── minimax.ts         # MiniMax theme + blocked tools
│   └── *.ts               # Other brand configs

test/                       # Node test runner tests
├── e2e/                   # End-to-end tests
├── tui/                   # TUI component tests
├── unit/                  # Unit tests
└── helpers/               # Test utilities

repos/                      # Upstream reference copies (vendor data, history)
├── anthropic-claude-code-*/       # Claude Code versions for comparison/reference
├── claude-code-system-prompts/    # System prompt changelog and sources
└── tweakcc/                       # TweakCC repo (prompt patching tool)

notes/                      # Research notes and deep dive documentation
├── CLI-VERSIONS.md               # Version comparison notes
├── *-DEEP-DIVE.md                # Feature research and analysis
└── RECONSTRUCTION-LEDGER.md      # Project state and decisions

docs/                       # User documentation
dist/                       # Build output (generated)
```

## Build, Test, and Development Commands

```bash
npm install          # Install dependencies
npm run dev          # Run CLI from TypeScript sources
npm run tui          # Launch TUI wizard
npm test             # Run all tests
npm run typecheck    # TypeScript check without emit
npm run bundle       # Build dist/cc-mirror.mjs
npm run render:tui-svg  # Regenerate docs/cc-mirror-tree.svg
```

## Coding Conventions

- **TypeScript + ESM**: Use `import`/`export`, avoid CommonJS
- **Formatting**: 2-space indent, single quotes, semicolons
- **Tests**: Name as `*.test.ts`, place in `test/` mirroring `src/` structure
- **New files**: Place in relevant `src/<area>/` folder

## Runtime Layout & Config Flow

### Variant Directory Structure

```
~/.cc-mirror/<variant>/
├── config/
│   ├── settings.json       # Env overrides (API keys, base URLs, model defaults)
│   ├── .claude.json        # API-key approvals + onboarding/theme + MCP servers
├── tweakcc/
│   ├── config.json         # Brand preset + theme list + toolsets
│   └── system-prompts/     # Prompt-pack overlays (after tweakcc apply)
├── native/
│   └── claude              # Native Claude Code binary (or claude.exe on Windows)
└── variant.json            # Metadata
```

### Wrapper Script

Location: `<bin-dir>/<variant>` (macOS/Linux) or `<bin-dir>/<variant>.cmd` (Windows)

Default `<bin-dir>` is `~/.local/bin` on macOS/Linux and `~/.cc-mirror/bin` on Windows.

- Sets `CLAUDE_CONFIG_DIR` to variant config
- Loads `settings.json` into env at runtime
- Shows provider splash ASCII art when TTY and `CC_MIRROR_SPLASH != 0`
- Auto-update disable: `DISABLE_AUTOUPDATER=1` in settings.json env

### Provider Auth Modes

| Provider                              | Auth Mode  | Key Variable                                 |
| ------------------------------------- | ---------- | -------------------------------------------- |
| zai, minimax, kimi, custom            | API Key    | `ANTHROPIC_API_KEY`                          |
| openrouter, vercel, nanogpt, gatewayz | Auth Token | `ANTHROPIC_AUTH_TOKEN`                       |
| ollama                                | Auth Token | `ANTHROPIC_AUTH_TOKEN` + `ANTHROPIC_API_KEY` |
| ccrouter                              | Optional   | placeholder token                            |
| mirror                                | None       | user authenticates normally                  |

### Model Mapping (env vars)

- `ANTHROPIC_DEFAULT_SONNET_MODEL`
- `ANTHROPIC_DEFAULT_OPUS_MODEL`
- `ANTHROPIC_DEFAULT_HAIKU_MODEL`
- Optional: `ANTHROPIC_SMALL_FAST_MODEL`, `ANTHROPIC_MODEL`, `CLAUDE_CODE_SUBAGENT_MODEL`

## Provider Blocked Tools


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numman-ali/cc-mirror](https://github.com/numman-ali/cc-mirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
