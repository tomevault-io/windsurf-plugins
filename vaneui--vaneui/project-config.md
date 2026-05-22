---
trigger: always_on
description: VaneUI (`@vaneui/ui`, v0.9.0) is a React component library with 24 customizable UI components. Built with TypeScript, React 19, Tailwind CSS v4, and Rollup. Uses a boolean props API (`<Button primary lg filled>`) and CSS variable-based theming via `ThemeProvider`.
---

# CLAUDE.md

## Project Overview

VaneUI (`@vaneui/ui`, v0.9.0) is a React component library with 24 customizable UI components. Built with TypeScript, React 19, Tailwind CSS v4, and Rollup. Uses a boolean props API (`<Button primary lg filled>`) and CSS variable-based theming via `ThemeProvider`.

## CRITICAL: Verification After ANY Code Change

**After making ANY code changes** (new features, bug fixes, refactoring, import changes, theme updates, etc.), you MUST run the full verification pipeline. Do NOT report work as complete until all checks pass.

```bash
cd C:/GitHub/vaneui

npm run type-check    # TypeScript type validation
npm run lint          # ESLint — zero errors required
npm test              # Jest — all tests must pass
npm run build         # Full build (includes type-check + lint + rollup + CSS)
npm run test:e2e      # Playwright e2e tests (after visual/CSS/theme changes, new components, or before publishing)
```

**`npm run build` is the most comprehensive check** — it runs type-check, lint, rollup bundling, and CSS generation. If build passes, type-check and lint also passed. However, it does NOT run tests, so always run `npm test` separately.

**E2e tests** validate computed CSS styles in a real browser (color inheritance, font-size scaling, border rendering). Run them after any visual/CSS/theme changes, when adding new components, or before publishing.

**Common pitfall**: `tsc --noEmit` (type-check) can pass while runtime tests fail due to circular dependencies. Always run BOTH type-check AND tests.

## Component Implementation Workflow

When creating or modifying components, **ALL steps below must be completed**. Use the `component-implementation` agent for guidance or `pre-commit-checker` agent to verify.

### Required Steps for New Components

1. **Create Component**
   - Component file: `src/components/ui/{component}.tsx`
   - Theme file: `src/components/ui/theme/{component}Theme.ts`
   - Add categories to `src/components/ui/props/keys.ts` if needed
   - **Key Type Pattern**: Define keys in `ComponentKeys`, export Key type from `keys.ts`, theme files import from `../../props`

2. **Integrate with Theme System**
   - Update `src/components/themeContext.tsx` (import, ThemeProps, defaultTheme, ThemeDefaults, ThemeExtraClasses)
   - Update `src/index.ts` (export component and props type)

3. **Write Tests (REQUIRED)**
   - Create `src/components/tests/{component}.test.tsx`
   - Test: default rendering, size variants, appearance variants, variant modifiers, shape variants, ref forwarding, prop leak prevention, className merging, tag switching (if applicable)
   - **Add to `componentThemeCoverage.test.ts`** — every component with categories + theme must have an entry in this test file to validate that all category keys have theme mappers and all boolean defaults have handlers. See `.claude/rules/testing.md` for details.
   - See `.claude/rules/testing.md` for patterns

4. **Add Playground Examples (REQUIRED)**
   - Add visual examples to `playground/src/App.tsx` showcasing the new/changed component
   - Include: default usage, size variants, appearance variants, key props, real-world context
   - Follow the existing section pattern: `Divider` + `SectionTitle` + multiple `Card` examples
   - **Cleanup**: Before adding new sections, remove or condense older examples so `App.tsx` stays focused (~3-5 sections, ~500 lines). It is a living showcase, not an append-only log.
   - See `.claude/rules/playground-examples.md` for patterns

5. **Add E2E Fixtures & Tests**
   - Add test fixtures to `e2e/fixtures/test-harness.tsx` with `data-testid` attributes
   - Add e2e spec in `e2e/` validating computed CSS styles (see `.claude/rules/e2e-testing.md`)

6. **Verify (ALL must pass)**
   ```bash
   npm run type-check    # TypeScript
   npm run lint          # ESLint — zero errors required
   npm test              # Jest — all tests must pass
   npm run build         # Full build
   npm run test:e2e      # Playwright e2e tests
   ```

**Work is NOT complete until tests are written and all verification passes.**

### Agent Delegation (REQUIRED)

When a task matches an agent's trigger below, you **MUST** delegate to that agent. Do not perform the work inline when a matching agent exists.

| Task Pattern | Agent | Why |
|-------------|-------|-----|
| Creating new components or full implementation workflow | `component-implementation` | Guides all required steps end-to-end |
| Before committing any code changes | `pre-commit-checker` | Runs type-check → lint → test → build gate |
| After writing or modifying tests | `test-runner` | Runs Jest + reports results efficiently (haiku) |
| Auditing component completeness (theme, tests, exports) | `component-auditor` | Validates all layers are wired correctly |
| Debugging component rendering, prop, or styling issues | `debugger` | Traces 3-layer prop system end-to-end |
| Debugging theme/CSS variable issues specifically | `theme-debugger` | Specializes in CSS variable chain and theme inheritance |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vaneui/vaneui](https://github.com/vaneui/vaneui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
