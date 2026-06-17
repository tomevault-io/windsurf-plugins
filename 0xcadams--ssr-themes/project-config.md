---
trigger: always_on
description: - Bun workspace for the `ssr-themes` library, the `landing/` app, and the `examples/` apps.
---

# AGENTS.md

## Scope

- Bun workspace for the `ssr-themes` library, the `landing/` app, and the `examples/` apps.
- Package manager is `bun@1.3.11`; runtime baseline is Node `>=20`.
- CI runs from the repo root and covers unit tests plus Playwright.
- GitHub Actions CI and release jobs run on Node `24`.

## Repo Map

- `ssr-themes/` - published library source, Vite library build, Vitest tests.
- `landing/` - TanStack Start marketing/docs site.
- `examples/next/` - Next.js App Router example used by Playwright.
- `examples/solid/` - SolidStart example.
- `examples/tanstack-start/` - TanStack Start example.
- `test/` - root Playwright specs.

## Repo-Specific Rules

- No existing root `AGENTS.md` was present when this file was created.
- No `.cursorrules`, no `.cursor/rules/`, and no `.github/copilot-instructions.md` were found.
- There are no extra Cursor or Copilot instruction files to merge beyond this document.

## Tooling Notes

- Run commands from the repo root unless a package-local run is clearer.
- Root scripts use Bun workspace filters; there is no Turbo/Nx layer.
- There is no ESLint, Biome, or Rome config today.
- Formatting is controlled by `.prettierrc`; standalone typecheck uses `tsc --noEmit`.
- npm publishing uses `.github/workflows/release.yml` with npm trusted publishing; keep that filename aligned with the npm trusted publisher config.

## Commands

Verified from the repo root:

```bash
# install / clean
bun install
bun run clean
# dev
bun run dev
bun --filter ssr-themes dev
bun --filter @ssr-themes/next dev
bun --filter @ssr-themes/solid dev
bun --filter @ssr-themes/tanstack-start dev
bun --filter @ssr-themes/landing dev
# build
bun run build
bun --filter ssr-themes build
bun --filter @ssr-themes/next build
bun --filter @ssr-themes/solid build
bun --filter @ssr-themes/tanstack-start build
bun --filter @ssr-themes/landing build
# format / lint-equivalent / typecheck
bun run format
bun run check-format
bunx prettier . --write
bun run check-types
# underlying commands
bunx prettier . --check
bunx tsc --noEmit -p ssr-themes/tsconfig.json
bunx tsc --noEmit -p landing/tsconfig.json
bunx tsc --noEmit -p examples/next/tsconfig.json
bunx tsc --noEmit -p examples/solid/tsconfig.json
bunx tsc --noEmit -p examples/tanstack-start/tsconfig.json
# tests
bun run test
bun --filter ssr-themes test
bun run test:e2e
# single unit test
bunx vitest run ssr-themes/tests/header.test.ts
bunx vitest run ssr-themes/tests/index.test.tsx -t "setSelected"
# single Playwright test
bunx playwright test test/system-theme.test.ts
bunx playwright test test/system-theme.test.ts -g "should render dark theme if preferred-colorscheme is dark"
bunx playwright test --list test/system-theme.test.ts
```

- `bun run format` is the write-mode formatter command.
- `bun run check-format` is the repo's lint-equivalent formatting check.
- `bun run check-types` runs the repo's typecheck suite.
- `bun run test` mainly runs the library's Vitest suite.
- `bun run test:e2e` first builds `ssr-themes`, then runs Playwright.
- Playwright boots `@ssr-themes/next` on port `4041`.
- Use Vitest `-t` and Playwright `-g` to target a single test; use Playwright `--list` to inspect titles.

## Publishing

- Create releases from a tag that matches `ssr-themes/package.json` as `v<version>`, for example `gh release create v0.2.1 --target main --notes "..."`.
- `.github/workflows/release.yml` runs on GitHub Release publish and optional manual dispatch; it checks the tag/version match before publishing.
- The release workflow installs with `bun install --frozen-lockfile`, runs unit tests, Prettier check, workspace type checks, Playwright, a full workspace build, dry-runs `npm pack`, and publishes from `ssr-themes/`.
- npm authentication is handled by trusted publishing with GitHub-hosted runners and `id-token: write`; no `NPM_TOKEN` should be added for normal releases.
- If the workflow filename, repository, or optional environment changes, update the npm trusted publisher settings to match exactly before the next release.

## Code Style

### Formatting

- Follow `.prettierrc`: single quotes, trailing commas, no bracket spacing, `arrowParens: avoid`, `quoteProps: consistent`.
- Prettier print width is `55`; aggressive wrapping is expected.
- Keep semicolons and let Prettier own line breaks.

### Imports

- Keep imports at the top of the file.
- Put external packages before internal alias or relative imports.
- Use `import type` or inline `type` specifiers for type-only imports.
- Use `import * as React from 'react'` when the file uses the `React.` namespace; otherwise named imports are fine.
- CSS side-effect imports usually come after value imports.
- Match the surrounding file's grouping style instead of imposing a new one.

### TypeScript

- Prefer explicit props and return types when they improve readability or public API safety.
- Avoid `any`; use generics, unions, `unknown`, `Partial<Record<...>>`, template literal types, `as const`, and `satisfies`.
- Preserve strong public typing in `ssr-themes/` exports.
- The library is not fully `strict`, but it still relies on `strictNullChecks`, `exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`, `noUnusedLocals`, `noUnusedParameters`, and `noImplicitReturns`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xcadams/ssr-themes](https://github.com/0xcadams/ssr-themes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
