---
trigger: always_on
description: Guidance for **Claude Code** (claude.ai/code) and other AI agents working with this repository.
---

# CLAUDE.md

Guidance for **Claude Code** (claude.ai/code) and other AI agents working with this repository.

## Overview

**Video.js 10** is a **Turborepo‑managed monorepo**, organized by runtime and platform.
Refer to **[`CONTRIBUTING.md`](./CONTRIBUTING.md)** for setup, development, and lint/test instructions.

## Package Layout

| Package Path            | Purpose                                                             |
| ----------------------- | ------------------------------------------------------------------- |
| `packages/utils`        | Shared utilities (`/dom` subpath for DOM‑specific helpers).         |
| `packages/element`      | Custom element base class for web components.                       |
| `packages/store`        | State management (`/html`, `/react` subpaths for platforms).        |
| `packages/spf`          | Stream Processing Framework (`/dom` and `/playback-engine` subpaths for DOM bindings and the HLS engine). |
| `packages/core`         | Core runtime‑agnostic logic (`/dom` subpath for DOM bindings).      |
| `packages/icons`        | SVG icon library (private, consumed by html and react).             |
| `packages/skins`        | Shared skin CSS and Tailwind tokens (private).                      |
| `packages/html`         | Web player—DOM/Browser‑specific implementation.                     |
| `packages/react`        | React player—adapts core state to React components.                 |
| `packages/react-native` | React Native player (planned, not yet implemented).                 |
| `packages/cli`          | `@videojs/cli` — CLI for reading docs, and more in the future.      |
| `apps/sandbox`          | Vite‑based dev playground (private, not published).                  |
| `apps/e2e`              | Playwright end‑to‑end and visual snapshot tests (private).          |
| `site/`                 | Astro‑based docs and website.                                       |

### Dependency Hierarchy

```text
utils/*         ← shared utilities
utils/dom       ← DOM-specific helpers

element         ← custom element base

store           ← state management
store/html      ← HTML bindings (controllers, mixins)
store/react     ← React bindings

spf                  ← framework primitives (composition, signals, tasks, actors, reactors)
spf/dom              ← DOM bindings for SPF
spf/playback-engine  ← HLS playback engine + SpfMedia adapter

core            ← runtime-agnostic logic
core/dom        ← DOM bindings

icons           ← SVG icon library (private)
skins           ← shared skin CSS (private)

html            ← Web player (DOM/Browser)
react           ← React player
react-native    ← React Native player (planned, not yet implemented)
```

```text
utils ← element
utils ← store ← core ← html / react
utils ← spf ← core
icons, skins → html / react
```

## Workspace

- Uses **PNPM workspaces** + **Turbo** for task orchestration.
- Internal deps are linked with `workspace:*`.
- Always use PNPM, do not use other package managers.

### Common Root Commands

```bash
# Install workspace deps
pnpm install

# Run all demos/sites in parallel
pnpm dev

# Typecheck across repo (fast - uses TypeScript project references)
# Always run from root, not per-package
pnpm typecheck

# Build all packages/apps
pnpm build
# Build all packages (no apps)
pnpm build:packages
# Build sandbox (and its package deps)
pnpm build:sandbox
# Build specific package
pnpm -F <pkg> build

# Run unit tests across all packages
pnpm test
# Run tests for specific package
pnpm -F <pkg> test
# Run tests matching a name or pattern
pnpm -F <pkg> test -t "test name pattern"
# Run tests for a specific file
pnpm -F <pkg> test src/path/to/file.test.ts
# Run tests matching a glob or filter
pnpm -F <pkg> test src/core

# Run e2e tests (Chromium + WebKit)
pnpm test:e2e
# Run e2e tests (Chromium only, faster)
pnpm test:e2e:vite
# Update visual snapshot baselines
pnpm test:e2e:update

# Lint all workspace packages
pnpm lint
# Lint and fix a single file
pnpm lint:fix:file <file>

# Remove all dist and types outputs
pnpm clean

# Validate workspace consistency (CI coverage, scopes, define imports, etc.)
pnpm check:workspace

# Measure bundle size (SPF only)
pnpm -F @videojs/spf size        # Public API (minified + gzipped)
pnpm -F @videojs/spf size:all    # All exports (minified + gzipped)
```

## Dev Workflow

1. Make changes.
2. If you added/changed **exported types** in a package, run `pnpm -F <pkg> build` first.
   - `pnpm typecheck` uses TypeScript project references against **built** `.d.ts` files.
   - New/changed types won't be visible until `tsdown` builds them.
3. Typecheck, fix all issues.
4. Run test/s, fix all issues. If there are no tests add them.
5. Lint file/s, fix all issues.
6. Run build/s, fix all errors.
7. Run `pnpm check:workspace` — fix any consistency warnings.
8. Before creating a PR `pnpm test`.
9. If your changes introduced new patterns or conventions, ask the user to run `/claude-update`.

Be efficient when running operations, see "Common Root Commands".

## Testing

### File Organization

Tests live in a `tests/` directory next to the implementation they cover:

```text
packages/utils/src/dom/
├── listen.ts
├── event.ts
└── tests/
    ├── listen.test.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [videojs/v10](https://github.com/videojs/v10) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
