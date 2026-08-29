---
trigger: always_on
description: This file provides context for AI coding agents (GitHub Copilot, Claude, Gemini, etc.) working in this repository.
---

# github-code-search — Agent instructions

This file provides context for AI coding agents (GitHub Copilot, Claude, Gemini, etc.) working in this repository.

## What this project does

`github-code-search` is an interactive CLI (powered by [Bun](https://bun.sh)) to search GitHub code across an organisation. It aggregates results per repository, displays a keyboard-driven TUI and lets the user select extracts before printing structured markdown or JSON output. A `query` subcommand and an `upgrade` subcommand are exposed via [Commander](https://github.com/tj/commander.js).

## Runtime & toolchain

| Tool           | Version                                                  |
| -------------- | -------------------------------------------------------- |
| **Bun**        | ≥ 1.4 (runtime, bundler, test runner, package manager)   |
| **TypeScript** | via Bun (no separate `tsc` invocation needed at runtime) |
| **oxlint**     | linter (`bun run lint`)                                  |
| **oxfmt**      | formatter (`bun run format`)                             |
| **knip**       | dead-code detector (`bun run knip`)                      |

There is **no Node.js / npm** involved. Always use `bun` commands.

## Bootstrap

```bash
bun install          # install dependencies (reads bunfig.toml + package.json)
```

`bunfig.toml` sets `smol = true` (lighter install). No additional setup step is needed.

## Build commands

```bash
bun run build.ts                            # compile a self-contained binary → dist/github-code-search
bun run build.ts --target=bun-darwin-arm64  # cross-compile (see CONTRIBUTING.md for all targets)
```

The build script (`build.ts`) injects the git commit SHA, target OS and architecture into the binary. The produced binary has no runtime dependency and can be distributed as a single file.

## Running tests

```bash
bun test            # run the whole TypeScript test suite
bun test --watch    # re-run on file changes (development)
bun run test:bats   # run the shell-integration tests for install.sh (requires bats-core)
```

All TypeScript tests use Bun's built-in test runner (`@jest/globals`-compatible API: `describe`, `it`, `expect`). No additional testing library is needed. The setup file is `src/test-setup.ts` (referenced in `bunfig.toml`).

Shell-integration tests use [bats-core](https://github.com/bats-core/bats-core) (`brew install bats-core` or install via your package manager). They cover `install_completions()` from `install.sh` and are located in `install.test.bats`.

## Linting & formatting

```bash
bun run lint          # oxlint — must pass before submitting
bun run format        # oxfmt write (auto-fix)
bun run format:check  # oxfmt check (CI check)
bun run knip          # detect unused exports / files
```

Always run `bun run lint` and `bun run format:check` before considering a change complete.

## Project layout

```
github-code-search.ts    # CLI entry point — Commander subcommands: query, upgrade, completions
build.ts                 # Build script (Bun.build)
bunfig.toml              # Bun configuration (smol install, test preload)
tsconfig.json            # TypeScript configuration
knip.json                # knip (dead-code) configuration

src/
  types.ts               # All shared TypeScript interfaces (TextMatchSegment,
                         #   TextMatch, CodeMatch, RepoGroup, Row, TeamSection,
                         #   OutputFormat, OutputType)
  api.ts                 # GitHub REST API client (search, team fetching)
  api-utils.ts           # Shared retry (fetchWithRetry) and pagination (paginatedFetch)
                         #   helpers used exclusively by api.ts — performs network I/O
  cache.ts               # Disk cache for the team list (getCacheDir, getCacheKey,
                         #   readCache, writeCache) — performs filesystem I/O
  aggregate.ts           # Result grouping & filtering (applyFiltersAndExclusions)
  completions.ts         # Pure shell-completion generators: generateCompletion(),
                         #   detectShell(), getCompletionFilePath() — no I/O
  group.ts               # groupByTeamPrefix, applyTeamPick, rebuildTeamSections,
                         #   flattenTeamSections — team-prefix grouping + pick logic
  regex.ts               # Pure query parser: isRegexQuery(), buildApiQuery()
                         #   Detects /pattern/ syntax, derives safe API term,
                         #   returns RegExp for local client-side filtering — no I/O
  render.ts              # Façade re-exporting sub-modules + top-level
                         #   renderGroups() / renderHelpOverlay()
  scroll-cooldown.ts     # Pure scroll-cooldown state machine (createScrollCooldownState,
                         #   isScrollCooldownActive, updateScrollCooldown) — debounces
                         #   clicks during trackpad momentum scrolling — no I/O
  tui.ts                 # Interactive keyboard- and mouse-driven UI (navigation, filter
                         #   mode, help overlay, selection, SGR mouse tracking)
  output.ts              # Text (markdown) and JSON output formatters
  upgrade.ts             # Auto-upgrade logic (fetch latest GitHub release, replace binary)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulll/github-code-search](https://github.com/fulll/github-code-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
