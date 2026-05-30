---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the setup-liquibase GitHub Action repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the setup-liquibase GitHub Action repository.

## Project Overview

**setup-liquibase** is a production-ready GitHub Action that installs Liquibase for CI/CD workflows. This action **replaces** the legacy `liquibase-github-actions` organization's individual command actions with a single, flexible setup action following GitHub Actions best practices.

### Project Status
- **Status**: Production (v2.x active)
- **Replaces**: `../github-action-generator/` (deprecated)
- **Users**: Public GitHub Actions marketplace + internal Liquibase teams
- **Quality**: 83 comprehensive tests, multi-platform support
- **Current Version**: 2.0.0 (supports Community, Secure editions; OSS and Pro for backward compatibility)

## Build and Development Commands

### Core Commands
- `npm ci` - Install dependencies (use instead of npm install)
- `npm run build` - Build TypeScript to JavaScript bundle (dist/index.js)
- `npm run test` - Run all tests with memory optimization
- `npm run test:ci` - Run tests in CI mode (limited workers, no coverage)
- `npm run lint` - Run ESLint on TypeScript files
- `npm run format` - Format code with Prettier
- `npm run package` - Build and test in one command

### Testing Commands
- Run a single test file: `npm test -- __tests__/unit/installer.test.ts`
- Run tests with pattern: `npm test -- --testNamePattern="should validate"`
- Run tests with debug logging: `npm run test:debug`
- Memory optimization for tests: Use `npm run test:memory` with `--max-old-space-size=4096`
- CI tests use `--maxWorkers=1` to prevent resource exhaustion and cross-platform issues
- Tests run serially by default (`--maxWorkers=1`) with forced exit to avoid hanging processes

## Architecture Overview

### Primary Purpose
Single GitHub Action that installs Liquibase (Community or Secure editions) and adds it to PATH, allowing users to run any Liquibase command. This replaces the previous approach of having individual actions for each command.

### Key Components

1. **Entry Point** (`src/index.ts`)
   - Reads action inputs (version, edition)
   - Validates inputs using type guards
   - Proactively transforms environment variables (path safety)
   - Calls installer and sets outputs

2. **Installer** (`src/installer.ts`)
   - Core installation logic
   - Platform detection (Windows/Unix/macOS)
   - Download URL construction with edition-specific templates
   - Installation validation with timeout protection
   - Cross-platform tar/zip extraction

3. **Configuration** (`src/config.ts`)
   - Central location for all URLs and constants
   - Download URL templates for Community/Pro/Secure editions (Scarf-tracked for analytics)
   - Scarf packages: `liquibase-community-gha`, `liquibase-pro-gha`, `liquibase-secure-gha`
   - Minimum version enforcement (4.32.0)

### Important Implementation Details

- **Minimum Version**: 4.32.0 (enforced due to download endpoint compatibility)
- **Editions**: 'community' (Community edition, formerly OSS), 'secure' (Secure), 'oss' (backward compatibility), or 'pro' (backward compatibility)
- **Secure License**: Required at runtime via `LIQUIBASE_LICENSE_KEY` environment variable (not during installation)
- **Platforms**: Supports Linux (.tar.gz), Windows (.zip), and macOS (.tar.gz)
- **Build Output**: TypeScript compiles to single `dist/index.js` with source maps
- **Path Transformation**: Automatically converts absolute paths in Liquibase environment variables to workspace-relative paths for GitHub Actions compatibility and security
- **URL Selection Logic**: For Pro/Secure editions, versions > 4.33.0 use Secure download URLs; versions <= 4.33.0 use legacy Pro URLs
- **Pre-release/RC Versions**: Non-semver version strings (e.g., `5.1.0-RC114`, `5-secure-release-test`) are accepted when `download-url-base` is provided. A safety regex `/^[a-zA-Z0-9][a-zA-Z0-9._\-+]*$/` validates the version string to prevent path traversal and injection.

### Testing Structure

- **Unit Tests** (`__tests__/unit/`): Test individual modules
- **Integration Tests** (`__tests__/integration/`): Test real installations
- **Performance Tests** (`__tests__/performance/`): Verify memory/speed constraints
- **Error Handling Tests**: Validate failure scenarios
- Tests use Jest with ts-jest for TypeScript support
- CI workflow tests across Ubuntu, Windows, and macOS

### GitHub Action Configuration

- **action.yml**: Defines inputs (version, edition) and outputs (liquibase-version, liquibase-path)
- **Node Runtime**: Uses Node.js 24
- **Icon**: Database icon with blue color for marketplace

## Release Automation

### Single Workflow Architecture
**File**: `.github/workflows/release-drafter.yml`

**Process**:
1. **PRs merged** → Draft release updated automatically
2. **Manual dispatch** → Multi-platform tests + Release published
3. **Major tag automation** → v1 points to latest v1.x.x

**Key Features**:
- Multi-platform testing (Ubuntu, Windows, macOS)
- Dynamic changelog generation from commits
- GitHub App token security
- Automatic major version tag updates (v1 → v1.x.x)

### Release Process
1. **Development**: PRs automatically update draft releases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liquibase/setup-liquibase](https://github.com/liquibase/setup-liquibase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
