---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development
```bash
pnpm start          # Run app in development mode (with DEBUG logging enabled)
pnpm run reset-accessibility      # Reset accessibility permissions for Prompt Line
```

- `pnpm run setup-codesign` creates a "Prompt Line" self-signed certificate in the login Keychain. Automatically run by `install-app`, so manual execution is not needed.
- `pnpm start` sets `LOG_LEVEL=debug` automatically. Packaged apps always use INFO level.
- Logs: `~/.prompt-line/app.log` (use `tail -f ~/.prompt-line/app.log` for real-time monitoring)

### Testing
```bash
pnpm test                    # Run all tests
pnpm run test:watch         # Run tests in watch mode
pnpm run test:coverage      # Generate coverage report
pnpm run test:unit          # Run unit tests only
pnpm run test:integration   # Run integration tests only
pnpm run test:mutation      # Run mutation tests with Stryker
pnpm test tests/unit/utils.test.js              # Specific test file
pnpm test -- --testNamePattern="formatTimeAgo"  # Pattern matching
```

### Build & Distribution
```bash
pnpm run build      # Build the application (creates app + DMG for current architecture)
pnpm run install-app # Build and install directly to /Applications (skip DMG, for development)
pnpm run compile    # Full build: TypeScript + Renderer + Native Tools
pnpm run lint       # Check code style
pnpm run lint:fix   # Auto-fix code style issues
pnpm run typecheck  # Run TypeScript type checking
pnpm run pre-push   # Run all pre-push checks (lint + typecheck + test)
pnpm run clean      # Removes build artifacts (DMG, zip files)
pnpm run clean:cache     # Clears build caches
pnpm run clean:full      # Full cleanup (artifacts + caches + dist)
pnpm run generate:settings-example  # Regenerate settings.example.yaml
pnpm run migrate-settings           # Backup existing settings and replace with fresh defaults
pnpm run plugin:install <source>    # Install plugins from local path or GitHub repo
```

`pnpm run compile` performs: tsc → Vite renderer build → native tools (`cd native && make install`) → copy to dist.

### Code Signing
- `scripts/afterSign.js` auto-detects "Prompt Line" certificate in Keychain; falls back to ad-hoc signing if not found
- Override with `CODE_SIGN_IDENTITY` env var (e.g., `CODE_SIGN_IDENTITY=- pnpm run build` for ad-hoc)
- Verify signature: `codesign -d --requirements -` should show `certificate leaf = H"..."` (not `cdhash`)

### Git Hooks
- **Pre-commit**: ESLint --fix on staged .js/.ts files + TypeScript type checking
- **Pre-push**: typecheck + full test suite
- Setup: `pnpm install` (husky auto-configured via "prepare" script)

### Commit Message Guidelines
Follow Angular Commit Message Conventions: `<type>(<scope>): <subject>`

Types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`

```
feat(history): add search functionality to paste history
fix(window): resolve positioning issue on multi-monitor setups
```

### Pull Request Guidelines
- **Target Branch**: Create PRs against `develop` if it exists, otherwise against `main`
- **Language**: Write all PR titles and descriptions in English
- **Merge Strategy**: **Squash and merge** for feature PRs into `develop`. Use **regular merge commit** (no squash) when merging `develop` into `main`.

### Release Process
Uses [Release Please](https://github.com/googleapis/release-please) for automated releases. Config: `release-please-config.json`, manifest: `.release-please-manifest.json`, workflow: `.github/workflows/release-please.yml`.

Pushes to `main` with conventional commits automatically trigger a Release Please PR with version bump and CHANGELOG updates. Merging that PR creates a GitHub Release.

## Architecture Overview

### Electron Process Architecture
```
User Input → Renderer → IPC Event → IPCHandlers (coordinator) → Specialized Handler → Manager → Data/System
                ↑                                                                            ↓
                └─────────────────────────── IPC Response ───────────────────────────────────┘
```

- **Main Process** (`src/main.ts`): Application lifecycle, window management, system interactions
- **Renderer Process** (`src/renderer/`): UI and user interactions with 13+ specialized managers. See `src/renderer/CLAUDE.md`
- **Preload Script** (`src/preload/preload.ts`): Secure context bridge with whitelisted IPC channels
- **IPC Handlers** (`src/handlers/`): 9 specialized files, 52 IPC channels. See `src/handlers/CLAUDE.md`
- **Managers** (`src/managers/`): 16 specialized managers + window sub-module. See `src/managers/CLAUDE.md`
- **Config** (`src/config/`): Centralized settings with `default-settings.ts` as Single Source of Truth. See `src/config/CLAUDE.md`
- **Utils** (`src/utils/`): Shared utilities, native tools, file/symbol search. See `src/utils/CLAUDE.md`
- **Native Tools** (`native/`): 4 compiled Swift tools for macOS integration. See `native/CLAUDE.md`
- **Shared Types** (`src/types/`): TypeScript definitions shared across processes
- **Shared Libraries** (`src/lib/`): Custom search, template resolution, scoring utilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nkmr-jp/prompt-line](https://github.com/nkmr-jp/prompt-line) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
