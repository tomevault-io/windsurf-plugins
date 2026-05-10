---
trigger: always_on
description: Interactive CLI for managing model assignments in `oh-my-opencode.json`.
---

# oh-my-opencode-config

Interactive CLI for managing model assignments in `oh-my-opencode.json`.

**Runtime**: Bun only. No npm/yarn/pnpm.

```bash
bun install        # deps
bun run build      # tsc → dist/
bun run dev        # hot reload (--conditions=source)
bun run test       # bun:test (--conditions=source)
bun run check      # biome lint+format
bun run typecheck  # tsc --noEmit
```

## Task Completion Requirements

- Both `bun run check:fix` and `bun run typecheck` must pass before considering tasks completed.
- Use `bun run test` for task completion rather than invoking `bun test` directly.

## Structure

```
src/
├── cli/
│   ├── index.ts       # Commander setup, all command registration
│   ├── types.ts       # BaseCommandOptions (imported by ALL commands)
│   └── commands/      # Command implementations [see: cli/commands/AGENTS.md]
├── config/            # Config handling [see: config/AGENTS.md]
├── types/             # Zod schemas (Config, Model, Requirements, Colors)
├── models/            # Models cache parser + opencode.json custom model loader
├── validation/        # Model capability validation (type guards, not assertions)
├── prompts/           # @clack/prompts TUI (agents, models, variants, provider, searchable select)
├── profile/           # Named config profiles via symlinks (save/load/switch/delete/rename)
├── diff/              # Config diff generation + chalk formatting
├── backup/            # Backup rotation (keep 10) + restore
├── errors/            # Error classes (7) + centralized handler
├── update/            # CLI update notifier (npm registry check, 24h cache)
├── utils/             # Shared utilities (fs, output, npm, merge, execa, bun-file)
├── scripts/           # sync-agent-defaults (fetch upstream model-requirements.ts)
├── test-utils/        # Shared test helpers (asMock, clack mocks)
└── index.ts           # Entry (#!/usr/bin/env bun)
```

## Where to Look

| Task | Location |
|------|----------|
| Add CLI command | `src/cli/commands/` + register in `src/cli/index.ts` |
| New config field | `src/types/config.ts` (schema) + `src/config/defaults.ts` |
| Agent requirements | `src/types/requirements.ts` (`AGENT_REQUIREMENTS`) |
| Model validation | `src/validation/capabilities.ts` |
| Error handling | `src/errors/handlers.ts` (handler) + `src/errors/types.ts` (classes) |
| File paths / constants | `src/config/paths.ts` |
| Default models | `src/config/defaults.ts` (synced from oh-my-opencode repo) |
| Upstream sync | `src/config/upstream-agent-sync.ts` (parser) + `src/scripts/sync-agent-defaults.ts` |
| Profile management | `src/profile/manager.ts` (CRUD) + `src/profile/constants.ts` (validation) |
| TUI prompts | `src/prompts/` (agents, models, variants, provider, search) |
| Searchable select | `src/prompts/search.ts` (generic filterable list component) |
| Config path resolution | `src/config/resolve.ts` (shared by all commands) |
| File utilities | `src/utils/fs.ts` (atomicWrite, isErrnoException, getFileMtime) |
| Terminal output | `src/utils/output.ts` (printLine, printTable, printBlank) |
| Deep merge | `src/utils/merge.ts` (prototype-pollution-safe) |
| Update notifier | `src/update/notifier.ts` (background npm check, 24h cache) |
| Agent brand colors | `src/types/colors.ts` (hex map + colorizeAgent) |

## Key Files

- **Models cache**: `~/.cache/opencode/models.json` (read-only, from `opencode models`)
- **Available models cache**: `~/.cache/oh-my-opencode-config/available-models.json` (1h TTL)
- **User config**: `~/.config/opencode/oh-my-opencode.json`
- **Project config**: `.opencode/oh-my-opencode.json` (precedence)
- **OpenCode config**: `~/.config/opencode/opencode.json` (custom models from plugins)
- **Profiles**: sibling to config (e.g. `oh-my-opencode-{name}.json`, symlinked)
- **Profile template**: `oh-my-opencode.template.json` (merged into profiles on save)

## Most-Imported Modules (Centrality)

| Module | Importers | Role |
|--------|-----------|------|
| `utils/output` | 16 | ALL terminal output |
| `cli/types` | 14 | BaseCommandOptions for all commands |
| `config/resolve` | 13 | Config path resolution |
| `config/loader` | 10 | Load + validate config |
| `types/config` | 10 | Core Config type |
| `utils/fs` | 8 | File operations (atomicWrite, isErrnoException) |
| `errors/types` | 7 | Custom error classes |
| `types/requirements` | 7 | AGENT_REQUIREMENTS, Capability enum |

## Conventions

### Type Safety (STRICT)

- **No `any`** — enforced by biome + tsconfig
- **No `!`** (non-null assertion) — enforced
- **No `as Type`** — use type guards (`isAgentName`, `isCapability`, `hasProperty`, `isErrnoException`)
- **Zod schemas** for all external data (never raw `JSON.parse`)
- `noUncheckedIndexedAccess: true`
- `as const satisfies` for readonly constant records

### Shared Utilities (USE THESE, don't inline)

| Utility | Location | Replaces |
|---------|----------|----------|
| `resolveConfigPath(options.config)` | `config/resolve.ts` | `options.config \|\| discoverConfigPath() \|\| USER_CONFIG_FULL_PATH` |
| `isErrnoException(error)` | `utils/fs.ts` | `error instanceof Error && "code" in error` |
| `getFileMtime(path)` | `utils/fs.ts` | `fs.stat().catch() + stats?.mtime.getTime()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srod/oh-my-opencode-config](https://github.com/srod/oh-my-opencode-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
