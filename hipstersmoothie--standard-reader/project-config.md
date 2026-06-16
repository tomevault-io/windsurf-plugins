---
trigger: always_on
description: <!-- intent-skills:start -->
---

<!-- intent-skills:start -->

## Skill Loading

Before substantial work:

- Skill check: run `npx @tanstack/intent@latest list`, or use skills already listed in context.
- Skill guidance: if one local skill clearly matches the task, run `npx @tanstack/intent@latest load <package>#<skill>` and follow the returned `SKILL.md`.
- Monorepos: when working across packages, run the skill check from the workspace root and prefer the local skill for the package being changed.
- Multiple matches: prefer the most specific local skill for the package or concern you are changing; load additional skills only when the task spans multiple packages or concerns.
<!-- intent-skills:end -->

# Project Context

A blank [TanStack Start](https://tanstack.com/start) (React) application. No partner add-ons or
extra feature scaffolding were selected — this is the default blank React starter.

**Package manager: pnpm** (`pnpm@10.26.0`, pinned via `package.json#packageManager`). The project
was scaffolded with npm and later switched to pnpm: `package-lock.json` was removed and
`pnpm-lock.yaml` is the committed lockfile. Use `pnpm` for all installs/scripts.

## Living docs — keep these current as we work

This repo has two source-of-truth planning docs that **must be kept up to date as work happens**:

- [`APP_VISION.md`](./APP_VISION.md) — the product vision (concept, architecture, scope). When a
  decision changes the product direction, data model, lexicons, or architecture, **update
  `APP_VISION.md` in the same change** so it never drifts from reality.
- [`TODO.md`](./TODO.md) — the actionable roadmap derived from the vision. As you complete work,
  **check off the relevant items**; when scope changes or new work is discovered, **add/adjust
  items**. Keep `TODO.md` in sync with `APP_VISION.md`.

Treat updating these docs as part of "done" for any meaningful change, not an afterthought.

## How this project was scaffolded

The project was created with the TanStack CLI, then the generated output was merged into this
repository (which already contained `.git`, `.cursor/`, and a generic `.gitignore`).

```bash
# 1. Scaffold (run in a scratch directory, then merged into this repo root)
npx @tanstack/cli@latest create my-tanstack-app --agent

# 2. TanStack Intent (run in this repo root, after merge + install)
npx @tanstack/intent@latest install   # wires skill-loading guidance into this AGENTS.md
npx @tanstack/intent@latest list      # lists installed, loadable skills

# 3. Switched the package manager from npm to pnpm
rm package-lock.json node_modules -rf
pnpm install                          # generates pnpm-lock.yaml
```

- CLI version resolved: `@tanstack/cli@0.69.0` (via `@latest`).
- Intent version resolved: `@tanstack/intent@0.0.41` (via `@latest`).
- Effective scaffold config (`.cta.json`): `framework: react`, `mode: file-router`,
  `typescript: true`, `tailwind: true`, `packageManager: pnpm`, `intent: true`, no add-ons
  (`chosenAddOns: []`). (`packageManager` was originally `npm`; switched to `pnpm`. `.cta.json`
  records `tailwind: true` as the original scaffold choice, but Tailwind has since been removed in
  favor of StyleX — see "Stack & integrations".)

> Note: the workspace was an essentially empty git repo (not a competing platform template), so
> nothing from the CLI output had to be dropped. Every generated integration, dependency, script,
> and config is represented in this repo.

## Stack & integrations

- **Framework:** TanStack Start + TanStack Router (file-based routing), React 19.
- **Build/toolchain (CLI default):** Vite 8, `@vitejs/plugin-react`, `@tanstack/router-plugin`.
- **Design system:** [hip-ui](https://hip-ui.tngl.io) — a copy-and-own, StyleX + react-aria
  component library vendored into `src/design-system/`. **Build UI from these components and
  tokens** (see "Design system" below).
- **Styling:** StyleX (`@stylexjs/stylex`) is the only styling layer, compiled by
  `@stylexjs/unplugin` in `vite.config.ts`. **Tailwind has been removed** (no `@tailwindcss/vite`,
  `tailwindcss`, or `@tailwindcss/typography`, and `styles.css` is now just a tiny reset). Build all
  UI from design-system components + StyleX tokens.
- **Devtools:** `@tanstack/react-devtools` + `@tanstack/devtools-vite` (stripped from production
  builds automatically).
- **Icons:** `lucide-react` (used by the starter Header/Footer/ThemeToggle).
- **Testing:** Vitest + Testing Library + jsdom (no tests authored yet).
- **Lint/format:** oxlint (oxc) + oxfmt — see "Linting & formatting" below.
- **Agent tooling:** TanStack Intent skill mappings (see the block at the top of this file).

## Design system (hip-ui) — use it for all UI

This project vendors the **hip-ui** design system into `src/design-system/` (a "copy-and-own"
StyleX + [react-aria-components](https://react-spectrum.adobe.com/react-aria/) library). When
building or changing UI, **use these components and tokens instead of hand-rolling markup, raw
HTML elements, or ad-hoc CSS / inline styles.**

### Rules

- **Prefer design-system components.** Import from `src/design-system/<component>` (aliases `#/` and
  `@/` both map to `./src`, e.g. `import { Button } from "#/design-system/button"`). Components are
  named exports built on react-aria, so they're accessible by default (use their props rather than

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hipstersmoothie/standard-reader](https://github.com/hipstersmoothie/standard-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
