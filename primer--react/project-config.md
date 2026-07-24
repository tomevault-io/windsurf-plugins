---
trigger: always_on
description: **ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# Primer React Copilot Instructions

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Repository Organization

The project is the React implementation of GitHub's Primer Design System authored as a monorepo with separate npm workspaces.
The primary workspace is `packages/react` which contains the `@primer/react` package that distributes React components for the design system.

**Key Directory Structure:**

- `e2e/`: End-to-end tests running Visual Regression Tests (@vrt) and Accessibility Verification Tests (@avt) using Playwright
- `examples/`: Example applications (nextjs, theming, codesandbox) demonstrating design system usage
- `packages/react/`: Primary package containing all React components and main development workspace
- `packages/mcp/`: Model Context Protocol server for AI tools integration
- `packages/styled-react/`: Legacy styled-components package (being migrated from)
- `script/`: Repository-wide build and utility scripts
- `contributor-docs/`: Contributing guidelines, testing docs, and Architecture Decision Records (ADRs)

## Working Effectively

**Bootstrap and build the repository:**

- Dependencies: Node.js (check with `node --version`), npm (check with `npm --version`)
- `npm install` -- installs dependencies. ~5 seconds with cache, ~2 minutes for clean install. Set timeout to 180+ seconds.
- `npm run build` -- builds all packages. NEVER CANCEL. Takes 90 seconds without turbo cache, ~1 second with cache. Set timeout to 120+ minutes.
- `npx turbo run build` -- builds all packages including example applications. Takes ~33 seconds.

**Node.js version updates:**

- When updating Node.js, update `.nvmrc` and `.devcontainer/devcontainer.json` together. Also check contributor documentation for any hardcoded Node.js version guidance, such as `contributor-docs/CONTRIBUTING.md`.
- GitHub Actions workflows use `node-version-file: '.nvmrc'`; only update workflow files if they stop using `.nvmrc`.

**Run tests:**

- `npm test` -- runs unit tests. NEVER CANCEL. Takes 75 seconds. Set timeout to 90+ minutes. Runs 1500+ tests using Vitest in both node and chromium environments.
- Vitest console enforcement is enabled by default in CI. For local debugging, it is disabled unless you opt in with `VITEST_FAIL_ON_CONSOLE=true npm test`.
- `npm run type-check` -- runs TypeScript type checking across all packages. Takes 42 seconds. Set timeout to 60+ minutes.

**Development workflow:**

- Main component development happens in `packages/react/src/[ComponentName]/`

**Turbo usage in this repository:**

- This repository uses Turborepo for selected workspace orchestration, not for every validation command. Follow the existing root npm scripts first unless you are specifically changing Turbo configuration.
- `npm run build` delegates to `turbo run build --filter='!./examples/*'`, so it builds workspace packages while excluding examples. Use `npx turbo run build` when you intentionally need Turbo's direct CLI behavior, such as building all workspaces including examples.
- `npm run type-check` runs the root TypeScript check first, then `turbo run type-check` for packages that define that task.
- `npm test`, `npm run lint`, `npm run lint:css`, `npm run format`, and `npm run format:diff` are not currently Turbo-orchestrated. Do not replace them with Turbo commands unless you are intentionally updating the repository's task setup.
- Use the `turborepo` skill when creating or changing `turbo.json`, package task scripts that should be orchestrated by Turbo, caching/outputs/env behavior, CI Turbo usage, filters such as `--affected` or `--filter`, or debugging Turbo cache/task execution. For ordinary Primer React component work, validation, or package scripts that already exist, use the commands documented here instead of applying generic Turborepo guidance.

**Linting and formatting:**

- `npm run lint` -- lints JavaScript/TypeScript/Markdown. Takes 73 seconds.
- `npm run lint:fix` -- auto-fixes linting issues where possible
- `npm run lint:css` -- lints CSS files using Stylelint. Takes 6 seconds.
- `npm run lint:css:fix` -- auto-fixes CSS linting issues
- `npm run format` -- formats code using Prettier
- `npm run format:diff` -- checks for unformatted files. Takes 2 seconds.

## Validation Scenarios

**Always validate your changes by:**

1. Building the project: `npm run build` (ensures TypeScript compilation succeeds)
2. Running unit tests: `npm test` (ensures component behavior works correctly)
3. Type checking: `npm run type-check` (ensures TypeScript types are correct)
4. Linting: `npm run lint` and `npm run lint:css` (ensures code style compliance)
5. Formatting: `npm run format:diff` (ensures consistent code formatting)

**For component changes:**

1. Start Storybook: `npm start`
2. Navigate to your component story to visually verify changes
3. Run accessibility tests: `script/test-e2e --grep @avt` if accessibility is impacted
4. Run visual regression tests: `script/test-e2e --grep @vrt` if visual appearance changed

**Before committing:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [primer/react](https://github.com/primer/react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
