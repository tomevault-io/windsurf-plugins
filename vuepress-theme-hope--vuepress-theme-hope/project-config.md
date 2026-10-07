---
trigger: always_on
description: Guidance for AI agents working on this monorepo. Read it before making changes.
---

# vuepress-theme-hope — Agent Guide

Guidance for AI agents working on this monorepo. Read it before making changes.

## Repository Overview

A [pnpm workspaces](https://pnpm.io/workspaces) monorepo holding the VuePress Hope theme and its companion plugins, plus the documentation sites for each of them.

```plain
packages/   # Published packages (components, create, lightgallery, md-enhance, shared, theme)
docs/       # Documentation sites (components, lightgallery, md-enhance, shared, theme)
demo/       # Demo sites for each package
scripts/    # Shared build / release helpers
```

Each package follows the same layout:

```plain
src/
  client/   # Browser-only code (Vue components, composables, styles)
  node/     # Node.js-only code (plugin factory, markdown-it extensions)
  shared/   # Code that runs in both environments (types, constants, utils)
  index.ts  # Re-exports the node entry (consumed by VuePress core)
```

## Essential Commands

| Command                     | What it does                          |
| --------------------------- | ------------------------------------- |
| `pnpm build`                | Build every package with `tsdown`     |
| `pnpm bundle`               | Same, with `NODE_ENV=production`      |
| `pnpm test`                 | Run Vitest with coverage              |
| `pnpm lint`                 | OxLint + Oxfmt + Stylelint (auto-fix) |
| `pnpm lint:check`           | Same checks without auto-fix          |
| `pnpm lint:md`              | Markdownlint over every `*.md`        |
| `pnpm --filter <pkg> build` | Build a single package                |

Documentation sites are built per package, e.g. `pnpm --filter docs-theme docs:vite-build` or, from `docs/theme/`, `pnpm docs:vite-build`.

## Coding Standards

### Import / Export Rules

- Relative imports must use the `.js` extension even though sources are `.ts`.
- No cross-folder imports between `client`, `node` and `shared`.
- No Node.js APIs in `client`, no browser APIs in `node`, and neither in `shared`.
- No bundled external dependency warnings from the `bundle` command.

### Export Requirements

- Plugin and theme factory exports must be named consistently with the package name, and every type used in the exported API must also be exported.
- A file that exports a single function or class must have a filename matching that export.

### CSS / SCSS

- All CSS classes start with `vp-`. Classes integrating third-party content are exempt, e.g. `waline-wrapper` for the Waline comment system.
- Color variables must contain `-c-`; plugin variables are prefixed with the plugin name; theme variables with `vp-`.
- Icon variables inside class definitions must use `--icon`.

### JSDoc

- Required for all user-visible exports. Internal implementations do not need it, but existing comments must stay correct.
- Bilingual: English description first, a blank line, then Chinese.
- `@param` is required for every parameter, written bilingually and separated with `/`.
- `@default` is always included for user-visible things that have a default (including `@default false`); elsewhere only when the value is not obvious from the parameters.
- `@example` only on exported functions.
- `@description` is optional, only when extra explanation is needed.

**JSDoc template:**

````typescript
/**
 * English description
 *
 * 中文描述
 *
 * @description (optional) English detailed description
 *
 * 中文详细描述
 *
 * @param paramName - English description / 中文描述
 *
 * @default defaultValue
 * @example
 * ```ts
 * // Example code in TypeScript
 * ```
 */
````

## Commit Messages

Conventional Commits, single line only (the `commit-msg` hook rejects a body and enforces a 1–50 character subject):

```plain
feat(theme): add markdown field option
```

Allowed types: `feat|fix|docs|style|refactor|perf|test|workflow|build|ci|chore|types|release`.
Allowed scopes: `components|create|lightgallery|md-enhance|shared|theme|deps|demo|release` (or none).

## Documentation

Each package has its own docs site under `docs/<package>/src`, with English at the root and Chinese under `zh/`. Upstream plugin documentation lives in the [`vuepress-ecosystem`](https://github.com/vuepress/ecosystem) repository; when a plugin option or behavior changes upstream, sync the matching page here.

### General Requirements

- Write for developers: concise, clear and essential information only, with no typos or grammar errors.
- Consistent with code behaviors.
- Chinese and English content must be consistent in structure and content.
- Keep content concise and clear, prefer shorter over longer.
- Use "你" instead of "您" in Chinese.
- Ignore errors from the `@[code ...` import grammar and from VuePress components in markdown, which are valid in VuePress but not standard markdown.
- Always keep a blank line before a container closing marker (`:::`, `::::`). Without it Oxfmt treats the marker as a list continuation and drops it, leaving the container unclosed.
- Do not repeat a plugin's built-in locale data in its documentation; link to the shared locales page instead.

### Page Structure

Feature descriptions and option references are separated so the options section stays short:

- `## Usage`: install command and a minimal config example.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vuepress-theme-hope/vuepress-theme-hope](https://github.com/vuepress-theme-hope/vuepress-theme-hope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
