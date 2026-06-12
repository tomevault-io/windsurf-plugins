---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

GSmart is a CLI tool that generates smart commit messages using AI. It analyzes git changes and creates conventional commit messages using various AI providers (OpenAI, Anthropic, Google, Mistral, Fireworks AI, PlataformIA).

## Project Structure

```
src/
├── index.ts          # CLI bootstrap with Commander.js and signal handling
├── gsmart.ts         # Command registration and wiring
├── definitions.ts    # Shared types and interfaces
├── build-info.ts     # Generated build metadata (name, version, description)
├── types/
│   └── conf.d.ts     # Module type declaration for conf
├── commands/         # CLI command implementations (one file per command)
│   ├── main.ts       # Default command for commit message generation
│   ├── login.ts      # API key configuration
│   ├── reset.ts      # Configuration reset
│   ├── config.ts     # Custom prompt configuration (set, get, clear)
│   ├── completions.ts # Shell completion script generator (bash, zsh, fish)
│   └── index.ts      # Command barrel export
└── utils/            # Reusable helpers (side-effect free)
    ├── ai.ts         # AI provider abstraction, prompt building, retry, and timeout handling
    ├── config.ts     # Persistent API key storage and validation using conf package
    ├── constants.ts  # Shared constants (DEFAULT_PROVIDER, DEFAULT_TIMEOUT_MS, DEFAULT_MAX_RETRIES, INITIAL_RETRY_DELAY_MS)
    ├── debug.ts      # Debug logging controlled by --debug flag
    ├── git.ts        # Git command wrappers for status, diff, commits, staging
    ├── holiday.ts    # Seasonal greeting messages for CLI output
    ├── index.ts      # Shared helpers: file staging, clipboard, and retrieval logic
    ├── prompt-config.ts # Custom prompt persistence
    ├── providers.ts  # AI provider definitions and active-provider filter
    ├── version-check.ts # Update notification via update-notifier
    └── welcome.ts    # First-run welcome message with shell completion instructions

test/                 # Tests mirror source with .test.ts suffix
dist/                 # Compiled output (read-only, gitignored)
```

## Development Commands

Use **pnpm** for all workflows. Requires Node.js >=20 with ESM support.

| Command                  | Description                         |
| ------------------------ | ----------------------------------- |
| `pnpm install`           | Install dependencies                |
| `pnpm run dev`           | Watch mode for development          |
| `pnpm run build`         | Production bundle to `dist/`        |
| `pnpm run start`         | Run compiled CLI from dist/index.js |
| `pnpm test`              | Run test suite                      |
| `pnpm run test:coverage` | Run tests with coverage report      |
| `pnpm test -- --watch`   | Run tests in watch mode             |
| `pnpm run lint`          | ESLint checks                       |
| `pnpm run lint:fix`      | ESLint with auto-fix                |
| `pnpm run typecheck`     | TypeScript type checking            |
| `pnpm run format`        | Format with Prettier                |
| `pnpm run format:check`  | Check formatting without writing    |
| `pnpm run check`         | Run lint, typecheck, and test       |
| `pnpm run clean`         | Remove dist directory               |

## Architecture

### Command Architecture

The CLI follows a command pattern:

- Each command exports an `ICommand` object with name, description, options, arguments, and action
- Commands are registered in `src/gsmart.ts` and loaded by the main program
- The default command is "generate" which analyzes staged changes and creates commit messages

### AI Provider System

- Multiple providers supported through unified `AIBuilder` class
- Supported providers: OpenAI, Anthropic, Google, Mistral, Fireworks AI, PlataformIA
- Models: gpt-5-codex, claude-3-5-haiku-latest, gemini-2.0-flash, mistral-large-latest, firefunction-v1, radiance
- Fireworks and PlataformIA use OpenAI-compatible endpoints with custom base URLs
- API keys stored securely using the conf package with format validation per provider
- Requests have a 30s default timeout (configurable via `GSMART_TIMEOUT` env var)
- Automatic retries with exponential backoff for transient errors (default 3 retries)
- Error classification: network, auth, rate-limit, timeout, and generic errors

### Git Integration

- Uses `spawnSync` for git commands (status, diff, commit, staging)
- Parses `git status -z` (null-separated) for file change detection
- Supports renames, copies, and deletions with original path tracking
- Requires staged changes to generate commit messages

## Key Design Patterns

### Type Safety

- Strong TypeScript typing with interfaces for commands (options and arguments), providers, and git status
- Provider types constrained to specific string literals
- `GitStatus.status` is a string matching `git status --porcelain` codes (e.g., "M", "??", "D")

### Error Handling

- AI generation returns union types `string | {error: string}` for graceful error handling
- Detailed error classification for provider errors (network, auth, rate-limit, timeout)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ragnarok22/gsmart](https://github.com/ragnarok22/gsmart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
