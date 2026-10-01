---
trigger: always_on
description: OpenSpindle is an Electron desktop app for preparing and running jobs on a Makera Z1 CNC machine. This file holds the project's conventions, for people and for coding agents. README.md is the user guide; `docs/` holds the feature references, the file formats and the architecture.
---

# OpenSpindle

OpenSpindle is an Electron desktop app for preparing and running jobs on a Makera Z1 CNC machine. This file holds the project's conventions, for people and for coding agents. README.md is the user guide; `docs/` holds the feature references, the file formats and the architecture.

# Commands

- `npm ci` installs everything (and generates the plugin SDK's type declarations); `npm run dev` runs the app with the renderer dev server. The build scripts run TypeScript directly, so they need Node.js 22.18 or later.
- Before handing over a change, `npm run typecheck`, `npm run lint` and `npm run check` (Prettier; `npm run format` fixes what it finds) must pass, and `npm run build` must succeed.
- `npm run sim:z1` starts a simulated Makera Z1 on 127.0.0.1:2222 to connect to without a machine; `npm run sim:z1 -- --help` lists its fault injection options.

# Pull requests

- `main` is the trunk. Work on a short-lived branch and merge it through a pull request, which is squash-merged once CI passes; never push to `main`.
- The pull request's title becomes the commit on `main` and, for `feat`, `fix`, `perf` and `revert`, a line of the release notes installed apps show before updating. Write it as a Conventional Commit whose description says what changed for people using the app, in the present tense: `fix(job): A pause is found by the line the machine reports`. README.md's Pull requests section lists the types.
- Never edit package.json's `version`, CHANGELOG.md or `.release-please-manifest.json`: release-please's release pull request owns them (docs/releasing.md).

# Project preferences

- Do not add or run automated tests unless the user explicitly asks.
- Use the configured shadcn components for standard controls, forms, and dialogs.
- Use shadcn Select for dropdown menus so the popup follows the supplied theme. Do not substitute NativeSelect unless explicitly requested.
- Compose layouts with Tailwind utilities. Keep only the Tailwind/shadcn theme stylesheets: `src/styles.css` and its design tokens in `src/theme.css`, which the plugin SDK's theme imports so plugin views match the app (re-applying the shadcn preset writes tokens to `styles.css`; move them to `theme.css`). Do not add custom CSS files.
- Do not nest ternary expressions. Use early returns, named functions, or explicit branches.
- Preserve the typography: the shadcn preset's Space Grotesk for text, Space Mono for code (`font-mono`), and Space Mono's numerals for numbers shown as values, such as fields, readouts, tables, badges and fact lines (`font-numeric` covers only digits and signs, so words and units stay in Space Grotesk). Numbers in names and sentences stay in the text face. Do not switch fonts or override theme font weights to address rendering issues.
- The supplied preset is `b1tM1sQkM` (Mira / neutral / Space Grotesk, applied with `npx shadcn@latest apply --preset b1tM1sQkM`). Follow the official shadcn skill at https://github.com/shadcn-ui/ui/blob/main/skills/shadcn/SKILL.md and inspect component documentation with the shadcn CLI before composing UI. Use component variants for styling; limit layout classes to spacing, sizing, and positioning.

# Product rules

- OpenSpindle is an Electron app only. Do not add a browser build, browser fallbacks, or null checks for host services the preload always provides.
- Only an explicit user action moves the machine, changes its settings or runs a program. Importing, previewing, editing, saving and plugins never do, and Run lives only on the Job tab. Machine commands are verified by acknowledgement and telemetry and never retried automatically; Stop stays available.
- Keep machine and vendor specifics (anchor ids and names, factory positions, bed registration, firmware codes) in the machine's firmware adapter (`src/machine/firmware/<vendor>`) or its fixture kit (`src/domain/fixtures/<machine>`), never in generic code.
- Tools, tool catalogs and the starter library are data in `public/tool-libraries/`: `catalogs.json` lists them and `provenance.json` records their sources and corrections. Never hard-code tools, brands or vendor special cases in code.
- Reuse the component that already does a job, with the same title, labels and behaviour, instead of making a variant. Keep panels lean: no helper links, explanatory notes or decorative icons unless asked.
- Tools that act on the 3D view's selection go in the Prepare toolbar (`src/features/prepare/prepare-toolbar.tsx`), next to Auto-level; the viewer's left toolbar holds view options only. Prepare toolbar tools show only their icon; hovering one shows its name and what it does (`ToolbarButton` and `ToolbarToggle` in `src/components/workspace/toolbar-button.tsx`).
- When behaviour changes, update the README paragraph or the `docs/` page that describes it in the same change. Describe behaviour as it is now, without the history of earlier versions or plans.

# Code layout

See docs/architecture.md. In short:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openspindle/openspindle](https://github.com/openspindle/openspindle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
