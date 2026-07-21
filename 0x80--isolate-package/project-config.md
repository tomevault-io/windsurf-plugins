---
trigger: always_on
description: `isolate-package` is a tool for isolating monorepo workspace packages into
---

# Isolate Package - Development Rules

## Project Overview

`isolate-package` is a tool for isolating monorepo workspace packages into
self-contained directories with their internal dependencies and adapted
lockfiles. It's particularly useful for deploying to platforms like Firebase
from a monorepo without complex scripting.

## Core Concepts

- **Package Isolation**: Extracts a package from a monorepo with all its
  internal dependencies
- **Lockfile Adaptation**: Generates or adapts lockfiles for the isolated
  package structure
- **Multi-Package Manager Support**: Works with NPM, PNPM, Yarn, and partially
  with Bun
- **Firebase Integration**: Special support for Firebase deployments

## Code Style and Conventions

### TypeScript

- Use TypeScript for all source files
- Target ES modules (ESM) - the project uses `"type": "module"`
- Use path aliases: `~` maps to `src/` directory
- Prefer `node:` prefix for Node.js built-in modules (e.g., `node:path`,
  `node:assert`)
- Use `satisfies` operator for type-safe object literals where appropriate

### Functional Programming

- Prefer functional approaches over class-based ones (as seen in the recent
  Config refactor)
- Use pure functions where possible
- Avoid unnecessary state management
- Use `remeda` for functional utilities (not lodash)

### Error Handling

- Use `assert` from `node:assert` for critical validations
- Use the `getErrorMessage` utility for consistent error message extraction
- Provide clear, actionable error messages

### Logging

- Use the centralized logger from `lib/logger`
- Follow the log level hierarchy: debug < info < warn < error
- Use `debug` for detailed implementation info
- Use `info` for important user-facing information
- Include context in log messages (file paths, package names, etc.)

### File Organization

```
src/
├── lib/
│   ├── config.ts          # Configuration management
│   ├── logger.ts          # Logging utilities
│   ├── types.ts           # Shared type definitions
│   ├── lockfile/          # Lockfile processing logic
│   ├── manifest/          # Package.json manipulation
│   ├── output/            # Output generation
│   ├── package-manager/   # Package manager detection/handling
│   ├── registry/          # Package registry management
│   └── utils/             # General utilities
├── isolate.ts             # Main isolation logic
├── isolate-bin.ts         # CLI entry point
└── index.ts               # Library export
```

### Testing

- Use Vitest for testing
- Place test files next to the source files with `.test.ts` suffix
- Write focused unit tests for utility functions
- Use descriptive test names with `describe` and `it`

### Dependencies

- Use `fs-extra` instead of native `fs` for enhanced file operations
- Use `consola` for logging (via the centralized logger in `lib/logger`)
- Use `yaml` package for YAML parsing/writing
- Use `glob` for file pattern matching

## Key Implementation Patterns

### Package Manager Detection

```typescript
// Always use the centralized detection
const packageManager = detectPackageManager(workspaceRootDir);
// or for singleton usage within a module
const packageManager = usePackageManager();
```

### Path Handling

- Always use `path.join()` for cross-platform compatibility
- Never use string concatenation for paths
- Use utility functions for consistent path formatting:
  - `getRootRelativeLogPath()` for logging paths relative to root
  - `getIsolateRelativeLogPath()` for paths relative to isolate dir

### Manifest Manipulation

- Read manifests with `readTypedJson<PackageManifest>()`
- Write manifests with `writeManifest()`
- Always preserve unknown fields when modifying manifests
- Strip `devDependencies` and `scripts` from internal package manifests

### Async Operations

- Use async/await consistently
- Parallelize operations where possible with `Promise.all()`
- Handle file system operations carefully with proper error handling

## Configuration Philosophy

- Zero-config by default - most users shouldn't need configuration
- Configuration file: `isolate.config.json`
- Environment variables for debugging: `DEBUG_ISOLATE_CONFIG=true`
- Validate configuration keys and warn about unknown options

## Package Manager Specific Handling

### PNPM

- Preserve `workspace:*` specifiers in isolated output
- Copy or generate `pnpm-workspace.yaml`
- Handle Rush workspaces specially (generate workspace config)
- Prune lockfiles before writing

### NPM

- Replace workspace specifiers with `file:` paths
- Preserve original resolved versions and integrity by reading the root
  `package-lock.json` via Arborist `loadVirtual` and copying matching
  entries into the isolated lockfile
- Fall back to Arborist's `buildIdealTree` (generating from node_modules)
  when no root `package-lock.json` exists (`forceNpm` from non-npm
  monorepos, or modern Yarn v2+)

### Yarn

- Replace workspace specifiers with file paths
- Yarn v1: copy `yarn.lock` and run a local `yarn install` to prune
- Yarn v2+: fall through to the NPM generator

## Pull Request Guidelines

- Keep changes focused and atomic
- Update tests for any logic changes
- Update README.md for user-facing changes
- Include clear commit messages
- Consider backward compatibility

### PR Summary Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0x80/isolate-package](https://github.com/0x80/isolate-package) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
