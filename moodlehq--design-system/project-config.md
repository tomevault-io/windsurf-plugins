---
trigger: always_on
description: - This is a React + TypeScript component library built with Vite library mode; public entrypoint is `index.ts` and package exports are defined in `package.json`.
---

# GitHub Copilot Instructions

## Project shape (read first)

- This is a React + TypeScript component library built with Vite library mode; public entrypoint is `index.ts` and package exports are defined in `package.json`.
- The build uses Rollup's `preserveModules` mode: every source file is transpiled in isolation into its own stable, hash-free output file. This is intentional — Moodle loads files directly by URL without a consumer-side build step, so output paths must be predictable across releases.
- Runtime styling flow is: `index.ts` imports Bootstrap base CSS, generated design tokens (`tokens/css/index.css`), then component styles (`components/index.css`). Keep this order.
- Components apply Bootstrap CSS classes directly in JSX (no `react-bootstrap` dependency). Each component adds a stable `mds-*` class hook and is styled via `--mds-*` CSS custom properties in a colocated CSS file.
- Token sources are DTCG JSON in `tokens/dtcg/`; generated outputs in `tokens/css/` and `tokens/scss/` are produced by `scripts/tokens.ts` via Style Dictionary.

Published package exports:

| Import                                       | Resolves to                       |
| -------------------------------------------- | --------------------------------- |
| `@moodlehq/design-system`                    | `dist/index.js`                   |
| `@moodlehq/design-system/css`                | `dist/index.css`                  |
| `@moodlehq/design-system/components/<name>`  | `dist/components/<name>/index.js` |
| `@moodlehq/design-system/tokens/css`         | `tokens/css/index.css`            |
| `@moodlehq/design-system/tokens/scss`        | `tokens/scss/_index.scss`         |
| `@moodlehq/design-system/tokens/scss/legacy` | `tokens/scss/_index.legacy.scss`  |

Component subpath imports are auto-discovered at build time — adding a new folder under `components/` is sufficient; no changes to `package.json` or `vite.config.ts` are needed.

## Path-specific instruction files

Three scoped files contain the detailed rules for their areas. They auto-load in VS Code/Copilot when a matching file is open. Other agents should read the relevant file proactively before starting work in that area:

- **All work (quick local map):** `.github/instructions/component-index.instructions.md` — compact component overview and entry points
- **`components/**`** → `.github/instructions/components.instructions.md`
- **`*.stories.tsx`, `*.test.tsx`, `tests/**`** → `.github/instructions/stories-tests.instructions.md`
- **`tokens/**`, `scripts/tokens.ts`** → `.github/instructions/tokens.instructions.md`

Large fallback resource (opt-in only — no `applyTo`, never auto-loaded):

- **Fallback only:** `.github/instructions/design-system.instructions.md` — auto-generated full export from ZeroHeight, intentionally large, do not edit manually

Routing rule for agents:

1. Prefer ZeroHeight MCP and `.github/instructions/component-index.instructions.md` first.
2. Load scoped instruction files based on the file(s) being edited.
3. Load `.github/instructions/design-system.instructions.md` only when the agent cannot access ZeroHeight and still needs missing design context.
4. Ask before loading the large fallback file into the context window.

Component index artifacts:

- `.github/instructions/component-index.instructions.md` is the instruction-friendly quick map for humans and agents.
- `dist/component-index.json` is a build-generated machine-readable index for deterministic lookups.
- Generate with `npm run build-component-index` (also runs as part of `npm run build`).

> When adding a new instruction file, add a pointer entry to both this list and to `.claude/CLAUDE.md` so all agent entry points stay in sync.

## Recommended MCP servers

These MCP servers are useful when working in this repo. Configure them in your AI agent's MCP settings:

| Server                                                                                                          | Purpose                                                                                                                        |
| --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| [Figma](https://mcp.figma.com/mcp) (`https://mcp.figma.com/mcp`, HTTP)                                          | Design-to-code work — `get_design_context`, `get_screenshot`, `get_variable_defs`, `get_metadata`                              |
| [ZeroHeight](https://mcp.zeroheight.com) (HTTP, requires token — see ZeroHeight docs to generate)               | Browsing design system documentation and looking up existing token definitions before requesting new ones                      |
| [Storybook](http://localhost:6006/mcp) (`http://localhost:6006/mcp`, HTTP)                                      | Querying rendered stories — served automatically by `@storybook/addon-mcp` when Storybook is running, no separate setup needed |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moodlehq/design-system](https://github.com/moodlehq/design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
