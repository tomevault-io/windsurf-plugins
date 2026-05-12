---
trigger: always_on
description: DeepL CLI is a command-line interface for the DeepL API that integrates translation, writing enhancement (DeepL Write API), and developer workflow automation.
---

# Claude Code Project Configuration

## Project Overview

DeepL CLI is a command-line interface for the DeepL API that integrates translation, writing enhancement (DeepL Write API), and developer workflow automation.

### Current Status

- **Version**: see `VERSION` file / `package.json`
- **Tests**: see `npm test` output (target: all green; coverage thresholds enforced by jest config)
- **Test mix**: ~70-75% unit, ~25-30% integration/e2e

### Architecture

```
CLI Commands (translate, write, voice, sync, watch, glossary, tm, …)
           ↓
Service Layer (Translation, Write, Voice, Batch, Watch, Glossary,
               TranslationMemory, StyleRules, Admin, Document,
               GitHooks, Usage, Detect, Languages)
           ↓                         ↓
Sync Engine (src/sync)        Format Parsers (src/formats — 11 i18n formats)
           ↓                         ↓
API Client (Translate, Write, Glossary, Document, Voice,
            StyleRules, Admin, TMS)
           ↓
Storage (SQLite Cache, Config) + Static Data (src/data — language registry)
```

### Configuration

- **Config**: XDG default `~/.config/deepl-cli/config.json`, legacy `~/.deepl-cli/config.json`
- **Cache**: XDG default `~/.cache/deepl-cli/cache.db`, legacy `~/.deepl-cli/cache.db`
- **Path priority**: `DEEPL_CONFIG_DIR` > legacy `~/.deepl-cli/` > XDG env vars > XDG defaults
- **Environment Variables**:
  - `DEEPL_API_KEY` - API authentication
  - `DEEPL_CONFIG_DIR` - Override config and cache directory (used for test isolation)
  - `XDG_CONFIG_HOME` - Override XDG config base (default: `~/.config`)
  - `XDG_CACHE_HOME` - Override XDG cache base (default: `~/.cache`)
  - `NO_COLOR` - Disable colored output

### Key Project Files

- **CHANGELOG.md** - Release history and version notes
- **docs/API.md** - Complete CLI command reference

## Development Philosophy

### Package Management

- Always use the latest versions of packages
- Never downgrade packages to avoid ESM/CommonJS issues
- If a package is ESM-only, refactor code to use dynamic imports or ESM

### Test-Driven Development

This project follows strict TDD:

1. **Red** - Write failing tests that define expected behavior
2. **Green** - Write minimal code to make tests pass
3. **Refactor** - Improve code quality while keeping tests green
4. **Commit** - Save progress after each cycle

Always write tests before implementation code. Test behavior, not implementation. Keep tests isolated.

## Versioning and Changelog

Use **Semantic Versioning** with **Conventional Commits**:

- `feat:` → MINOR, `fix:` → PATCH, `BREAKING CHANGE:` → MAJOR
- `docs:, chore:, refactor:` → no bump unless behavior changes

### On Every Change

1. Add entries under **Unreleased** in `CHANGELOG.md` using **only** the standard [Keep a Changelog](https://keepachangelog.com) categories: **Added**, **Changed**, **Deprecated**, **Removed**, **Fixed**, **Security**. Do not invent custom categories (e.g., "Improved").
2. Determine MAJOR/MINOR/PATCH from change scope

### When Cutting a Release

1. Move Unreleased items to `## [X.Y.Z] - YYYY-MM-DD`
2. Update `VERSION` and `package.json` version
3. Create annotated tag: `git tag -a vX.Y.Z -m "Release vX.Y.Z: <summary>"`
4. Push: `git push && git push --tags`

## Code Style

- **Comment sparingly** - only when behavior is unclear
- **Type everything** - avoid `any`
- **Prefer explicit over implicit**
- **Follow existing patterns** in the codebase

### Naming Conventions

- **Files**: kebab-case (`translation-service.ts`)
- **Classes/Types/Interfaces**: PascalCase (`TranslationService`)
- **Functions/variables**: camelCase (`translateText`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_CACHE_SIZE`)

### File Organization

```
src/
├── cli/              # CLI interface and commands
├── services/         # Business logic
├── sync/             # Continuous localization engine (scan, diff, translate, write, lock)
├── formats/          # i18n file format parsers (JSON, YAML, PO, XLIFF, Android XML, etc.)
├── api/              # DeepL API client
├── storage/          # Data persistence (SQLite cache, config)
├── data/             # Static data (language registry)
├── utils/            # Utility functions
└── types/            # Type definitions
```

## Testing

### Stack

- **Jest** + **ts-jest** - Test runner with TypeScript support
- **nock** - HTTP request mocking for API tests
- **memfs** - File system mocking

### Test Types

All three are **required** for new features:

- **Unit tests** - Individual functions/classes in isolation, mock all dependencies. File: `tests/unit/<component>.test.ts`
- **Integration tests** - Component interactions with nock for HTTP mocking and `DEEPL_CONFIG_DIR` for isolation. File: `tests/integration/<component>.integration.test.ts`
- **E2E tests** - Complete CLI workflows, exit codes, stdin/stdout, error messages. File: `tests/e2e/<workflow>.e2e.test.ts`

### Test Structure

```typescript
describe('ComponentName', () => {
  describe('methodName', () => {
    it('should do something specific', () => {
      // Arrange → Act → Assert
    });
  });
});
```

### Required Scenarios for New Features

**Integration tests must cover:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeepLcom/deepl-cli](https://github.com/DeepLcom/deepl-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
