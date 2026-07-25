---
trigger: always_on
description: SPDX-FileCopyrightText: 2026 Siemens AG
---

<!--
SPDX-FileCopyrightText: 2026 Siemens AG

SPDX-License-Identifier: MIT
-->

# AGENTS.md

This guide is for humans and AI agents developing inside the Siemens IX monorepo.

## Agent operating principles

- Act as a senior maintainer: inspect existing patterns before changing code, keep changes scoped, and prefer correctness over speed.
- Change source files, not generated artifacts. If generated output is affected, update the source and run the appropriate build/generation step.
- Preserve unrelated user or maintainer changes in the working tree. Do not revert, reformat, or rewrite files outside the requested scope.
- Use the smallest targeted validation that proves the change, then escalate only when needed.
- Prefer explicit, type-safe fixes over broad fallbacks, silent returns, or catch-all error handling.
- Update tests, documentation, examples, and changesets whenever the user-facing behavior, API, styling, accessibility, or package output changes.

## Source of truth and precedence

Use these resources together (in this order when conflicts appear):

1. `CONTRIBUTING.md`
2. `ARCHITECTURE.md`
3. `.github/copilot-instructions.md`
4. `.github/instructions/*.instructions.md`

## Repository architecture (must-know)

- `packages/core` is the source of truth (Stencil Web Components).
- `packages/react`, `packages/angular`, `packages/vue` are wrapper packages generated from Stencil output targets.
- Do not hand-edit generated proxy files (look for auto-generated comments).
- Theme integrations for third-party libraries live in `packages/aggrid` and `packages/echarts`.

## Environment and package manager

- Use the package manager declared in `package.json`: `pnpm@10.17.0`.
- Use Node.js `22.19.0`; Volta is configured for this version.
- Install dependencies from the repository root with `pnpm install`.
- Do not introduce new package managers, lockfiles, formatters, linters, or test runners.
- Add dependencies only when they are required for the implementation, and keep workspace dependencies as `workspace:*` where applicable.

## Development rules

- Prefer changing `packages/core` first for component behavior, API, styling, accessibility, and docs metadata.
- Keep component structure consistent: `<component>.tsx`, `<component>.scss`, tests in `test/`.
- Use SPDX headers for new source files.
- Use design tokens/CSS custom properties instead of hard-coded theme values.
- Keep accessibility parity across frameworks.
- Reuse existing helpers, utilities, tokens, test fixtures, and component patterns before adding new abstractions.
- Avoid arbitrary waits, global side effects, and behavior changes that are not explicitly required.
- For breaking changes, provide migration guidance and update `BREAKING_CHANGES.md`.

## Package-specific guidance

| Package area | Edit guidance |
| --- | --- |
| `packages/core` | Source of truth for Web Components, styles, accessibility, docs metadata, and generated wrapper input. |
| `packages/react`, `packages/angular`, `packages/vue` | Do not edit generated proxy files. Only edit deliberate hand-written helpers, examples, or package-specific integration code. |
| `packages/aggrid`, `packages/echarts` | Keep integrations aligned with IX design tokens and theme classes. Validate visual/theming impact. |
| `packages/documentation` | Update docs generation logic only when documentation output or example extraction changes. |
| `*-test-app` packages | Use for framework examples, previews, and documentation snippets. Keep examples consumer-realistic. |

## Local workflow

```bash
pnpm install
pnpm storybook
pnpm build
pnpm lint
pnpm test
```

Targeted commands:

- Build single package: `pnpm build --filter @siemens/ix`
- Core component tests (watch): `pnpm --filter @siemens/ix test.ct --watch`
- Run visual regression (after build): `pnpm visual-regression`
- Core unit tests: `pnpm --filter @siemens/ix test.spec`
- Core component tests: `pnpm --filter @siemens/ix test.ct`
- Package lint: `pnpm lint --filter <package-name>`
- Package tests: `pnpm test --filter <package-name>`

## Testing expectations

- Build before tests when compiled artifacts, generated wrappers, styles, or visual/component tests depend on build output.
- `packages/core` is the main place for component tests. Core component tests live at `packages/core/src/components/<component>/test/<component>.ct.ts`.
- Use core component tests for component behavior, interaction, keyboard handling, accessibility, slots, events, and state changes.
- Include accessibility coverage (`makeAxeBuilder`) and a basic hydration/render test in component test files.
- Add or update unit tests for pure logic changes, component tests for interaction/accessibility changes, and visual tests only for meaningful UI or theme changes.
- Prefer Playwright locators, user-visible assertions, and deterministic waits over implementation-detail selectors and timeouts.

## Visual regression testing

- Visual regression tests live in `testing/visual-testing`.
- Visual regression tests run inside a Docker container so screenshots are operating-system agnostic and less affected by local fonts, browsers, or rendering differences.
- Visual regression tests are slow. Keep them focused on visual contracts, not every behavior or edge case.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siemens/ix](https://github.com/siemens/ix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
