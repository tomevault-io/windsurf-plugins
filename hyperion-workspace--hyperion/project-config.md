---
trigger: always_on
description: Hyperion is a single, flat Tauri 2 + Next.js application — the desktop/mobile agentic workspace client. There is no monorepo: one `package.json`, one `src/`, one `src-tauri/`, all at the repo root.
---

# Hyperion

Hyperion is a single, flat Tauri 2 + Next.js application — the desktop/mobile agentic workspace client. There is no monorepo: one `package.json`, one `src/`, one `src-tauri/`, all at the repo root.

The app is a standard Next.js 16 codebase built as a static HTML export (`output: "export"`), which Tauri 2 loads into a Rust-backed system webview for Windows, macOS, Linux, Android, and iOS. Pages, components, hooks, stores, config, and the design system all live under `src/`.

The project includes 40+ OKLCh color themes with light/dark variants, type-safe i18n via next-intl, a command palette (Cmd+K), keyboard shortcuts, and a sidebar dashboard layout. State management uses Zustand with localStorage persistence. Styling uses Tailwind CSS v4 with shadcn/ui components built on Radix UI.

CI/CD runs through GitHub Actions with Release Please for Conventional Commits-based versioning and automated changelogs.

## Setup and commands

```bash
# Install: always use pnpm, never npm/yarn
pnpm install

# Development
pnpm dev                   # Next.js dev server (http://localhost:3000)
pnpm tauri dev             # Desktop
pnpm tauri android dev     # Android
pnpm tauri ios dev         # iOS

# Quality gates (CI runs all of these on every PR)
pnpm check                 # Biome/Ultracite check
pnpm typecheck             # tsc --noEmit
pnpm build                 # Full production build (next build)

# Utilities
pnpm fix                   # Auto-format and fix lint issues
pnpm clean                 # Remove build artifacts
pnpm shadcn add <name>     # Add a shadcn/ui component to src/components
pnpm deps:check            # Check for outdated deps
pnpm deps:update           # Interactive update
```

Run everything from the repo root — there are no workspace packages or filters to worry about.

## Project structure

```
src/
  app/            Next.js App Router pages ([locale] routing via next-intl)
  components/     shadcn/ui primitives + feature components (auth, kanban, layout, navigation, panels, terminal...)
  hooks/          Shared React hooks
  i18n/           next-intl routing, navigation, plugin, messages/*.json
  lib/            Utilities (cn, motion, storage, AI provider factory, ...)
  pages/          Top-level page components rendered by app routes
  providers/      Context providers (theme, auth)
  stores/         Zustand stores
  config/         Site, navigation, hotkeys, notifications config
  styles/         Tailwind v4 global styles + theme tokens
  scripts/        Client-side init scripts

src-tauri/        Rust backend, Tauri config, capabilities, platform gen/
public/           Static assets
```

Everything is imported via the `@/*` path alias (e.g. `@/components/button`, `@/lib/utils`, `@/i18n/routing`) — never deep relative paths across top-level `src/` folders.

## Coding standards

### TypeScript

- Strict mode is on globally (`strict: true`, `noUncheckedIndexedAccess: true`).
- Target: `ES2022`. Module: `ESNext` / `Bundler` resolution.
- All new code must be TypeScript. No `.js` files in `src/`.

### Formatting and Linting (Biome/Ultracite)

- This project uses **Ultracite** as a zero-config preset over Biome.
- Formatting is strictly enforced (double quotes, 2-space indent).
- Linting catches issues but favors warnings or auto-fixing over failing builds locally.
- Tailwind class sorting is handled natively by Biome (`npx @biomejs/biome check`).
- Stylesheet reference: `src/styles/globals.css`.
- Run `pnpm fix` to automatically resolve formatting and lint issues.

### Commit messages

Follow [Conventional Commits](https://www.conventionalcommits.org/). Release Please parses these to auto-generate changelogs.

| Prefix      | Purpose                             |
| ----------- | ----------------------------------- |
| `feat:`     | New feature                         |
| `fix:`      | Bug fix                             |
| `docs:`     | Documentation only                  |
| `style:`    | Formatting, no logic                |
| `refactor:` | Refactor, no behavior change        |
| `perf:`     | Performance improvement             |
| `test:`     | Adding/fixing tests                 |
| `deps:`     | Dependency updates                  |
| `ci:`       | CI config changes                   |
| `chore:`    | Maintenance (hidden from changelog) |

## Architecture boundaries

### Do

- Add new UI primitives to `src/components/`. Use shadcn/ui patterns.
- Add shared pages to `src/pages/`, hooks to `src/hooks/`, stores to `src/stores/`.
- Add new translations to `src/i18n/messages/*.json`.
- Use the `@/*` path alias for all imports, never deep relative paths across top-level `src/` folders.

### Do not

- Do not modify `release-please-config.json`, `.release-please-manifest.json`, or GitHub workflow files without explicit approval.
- Do not use `npm` or `yarn`. This repo uses pnpm exclusively (v10+, corepack-managed).
- Do not add `"use server"` directives. The app must stay static-export compatible (`output: "export"`).
- Do not edit auto-generated directories: `.next/`, `out/`, `dist/`, `src-tauri/gen/`, `src-tauri/target/`, `node_modules/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hyperion-Workspace/Hyperion](https://github.com/Hyperion-Workspace/Hyperion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
