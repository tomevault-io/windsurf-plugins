---
trigger: always_on
description: This is a **VS Code extension** called "Remote - Kubernetes" that wraps the [Okteto CLI](https://okteto.com) to provide remote Kubernetes development environments directly from VS Code. It manages the full lifecycle: deploying dev environments, syncing code, connecting via SSH, and cleanup.
---

# AGENT.md - Remote Kubernetes VS Code Extension

## Project Overview

This is a **VS Code extension** called "Remote - Kubernetes" that wraps the [Okteto CLI](https://okteto.com) to provide remote Kubernetes development environments directly from VS Code. It manages the full lifecycle: deploying dev environments, syncing code, connecting via SSH, and cleanup.

**Publisher:** Okteto
**Repository:** https://github.com/okteto/remote-kubernetes
**License:** Apache 2.0

## Quick Reference

```bash
pnpm run compile      # Build (development mode via esbuild)
pnpm test             # Run unit tests (compiles first via pretest hook)
pnpm run test:e2e     # Run e2e tests (compiles with tsc, launches VS Code)
pnpm run lint         # Run ESLint
pnpm run watch        # Build + watch for changes
pnpm run package      # Create .vsix extension package
pnpm run ci           # Full CI pipeline: install + lint + test + package
```

## Architecture

### Source Layout

```
src/
├── extension.ts      # Main entry point. Registers all VS Code commands, orchestrates workflows
├── okteto.ts         # Okteto CLI wrapper. Spawns CLI processes, monitors state via terminal
├── ssh.ts            # SSH readiness checks and port discovery
├── telemetry.ts      # Analytics (Mixpanel) and error tracking (Sentry)
├── manifest.ts       # YAML manifest parsing (Okteto v2, Docker Compose)
├── download.ts       # Downloads Okteto CLI binary per platform
├── machineid.ts      # Platform-specific machine ID generation
├── paths.ts          # Git Bash path conversion (Windows support)
├── typings/          # TypeScript type definitions
└── test/
    ├── mock/
    │   └── vscode.ts            # VS Code API mock for unit tests
    ├── suite/                   # Unit tests (run in plain Node.js)
    │   ├── manifest.test.ts
    │   ├── paths.test.ts
    │   ├── machineid.test.ts
    │   ├── okteto.test.ts
    │   ├── download.test.ts
    │   ├── ssh.test.ts
    │   ├── telemetry.test.ts
    │   └── artifacts/           # Test fixture YAML files
    └── e2e/                     # End-to-end tests (run inside VS Code)
        ├── runTest.ts           # Launcher: downloads VS Code via @vscode/test-electron
        ├── index.ts             # Mocha bootstrap for extension host (TDD interface)
        └── extension.test.ts    # Validates extension activation and command registration
```

### Build Pipeline

- **Bundler:** esbuild (`esbuild.js`)
- **Entry:** `src/extension.ts` → **Output:** `dist/extension.js` (CommonJS, Node.js target)
- **TypeScript:** Strict mode, ES6 target, NodeNext modules
- The `vscode` module is externalized (provided by VS Code at runtime)
- **E2E tests use `tsc`** (not esbuild) via `tsconfig.test.json` because `@vscode/test-electron` needs individual `.js` files, not a bundle. The `tsconfig.test.json` extends the main config and adds `skipLibCheck: true` to avoid transitive type conflicts from `@types/eslint-scope` and `@types/glob`.

### Key Extension Commands

| Command | ID | Description |
|---------|-----|------------|
| Okteto: Up | `okteto.up` | Launch a dev environment |
| Okteto: Down | `okteto.down` | Stop a dev environment |
| Okteto: Deploy | `okteto.deploy` | Deploy dev environment |
| Okteto: Destroy | `okteto.destroy` | Destroy dev environment |
| Okteto: Test | `okteto.test` | Run tests in dev environment |
| Okteto: Install | `okteto.install` | Install Okteto CLI |
| Okteto: Set Context | `okteto.context` | Configure Okteto context |
| Okteto: Set Namespace | `okteto.namespace` | Configure Okteto namespace |

### Extension Configuration Options

Defined in `package.json` under `contributes.configuration`:
- `okteto.binary` - Path to Okteto CLI executable
- `okteto.remoteSSH` - Use VS Code Remote-SSH (default: true)
- `okteto.telemetry` - Enable analytics (default: true)
- `okteto.gitBash` - Windows Git Bash path support (default: false)
- `okteto.upArgs` - Extra args for `okteto up` (default: `--log-level=warn`)
- `okteto.upTimeout` - Timeout in seconds (default: 100)

## Testing

### Unit Tests

**Framework:** Mocha + Chai + Sinon + ts-node

```bash
pnpm test    # Runs: mocha -r ts-node/register -r src/test/mock/vscode.ts src/test/suite/*.test.ts
```

Unit tests are plain TypeScript files in `src/test/suite/`. They do **not** require a running VS Code instance. Test fixtures (YAML manifests) live in `src/test/suite/artifacts/`.

The `vscode` module is mocked via `src/test/mock/vscode.ts`, which intercepts Node's `Module._resolveFilename` and `Module._load` to provide a stub when any module imports `vscode`. This is required because modules like `okteto.ts`, `download.ts`, and `telemetry.ts` use `vscode` at runtime (not just for types).

### End-to-End Tests

**Framework:** @vscode/test-electron + Mocha (TDD interface)

```bash
pnpm run test:e2e    # Compiles with tsc, then launches VS Code with the extension
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okteto/remote-kubernetes](https://github.com/okteto/remote-kubernetes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
