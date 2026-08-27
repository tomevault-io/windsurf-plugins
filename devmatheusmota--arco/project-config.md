---
trigger: always_on
description: > Identical in content to [`CLAUDE.md`](CLAUDE.md) in this directory. Keep both in sync.
---

# Arco — working guide (AI)

> Identical in content to [`CLAUDE.md`](CLAUDE.md) in this directory. Keep both in sync.
> Contributing from outside? Start with [`CONTRIBUTING.md`](CONTRIBUTING.md) — setup, project
> layout, house rules, and PR convention.

## 1. What it is

**Arco** is a **Windows-first** desktop app that organizes, operates, and resumes multiple coding
agents (Claude Code, Codex, OpenCode) and shells in parallel, inside a persistent workspace with
real terminals (PTYs), layouts, themes, history, and RAM control.

> Tagline: **Reveal the state of every agent, shell, and project.**
> Status: **v1.3.0**, functional MVP in polish. Identifier: `com.mota.arco`.

## 2. Where you are

At the repository root — the app directory. It contains:

- `src/` — React frontend.
- `electron/` — the application shell: window, command router (`commands/`), PTY and speech hosts.
- `src-tauri/` — the previous Rust/Tauri shell, kept as legacy (not released).
- `docs/` — versioned docs (`FEATURES.md`, `CHANGELOG.md`, `OVERVIEW.md`, `BRAND.md`,
  `DIAGNOSTICO_MATURIDADE_TECNICA.md`).
- `package.json`, `vite.config.ts`, `tsconfig.json`, `tests/`.

## 3. Stack

- **Frontend:** React 18.3 · TypeScript 5.6 · Vite 6 · Zustand 5 · xterm.js 5.5 (`@xterm/addon-fit`, `-search`, `-webgl`) · `react-resizable-panels` · `@dnd-kit/core` · `@radix-ui/react-dialog` · `lucide-react` · `nanoid`.
- **Shell / backend:** Electron 43 (Chromium) · `electron/main.cjs` command router · PTY host on
  `@homebridge/node-pty-prebuilt-multiarch` · speech host on `sherpa-onnx`.
- **Legacy shell:** `src-tauri/` still holds the Rust/Tauri build the app used before v2. It is kept
  for reference and is not part of a release. Do not add features there.
- **Styling:** CSS Modules + CSS custom properties (no Tailwind, no styled-components).

## 4. Commands (from `package.json`)

```bash
npm install
npm run app      # builds the frontend and runs the full app (RECOMMENDED WAY)
npm run dev      # Vite frontend only, at http://localhost:1422 (strictPort)
npm run build    # tsc + vite build — tsc typechecks and VALIDATES i18n (see §5)
npm run package  # AppImage into dist-electron/
npm test         # vitest run over tests/**/*.test.ts (test:node runs via node --test, separately)
```

To iterate with hot reload, run `npm run dev` and start the shell against it:
`ARCO_DEV_URL=http://localhost:1422 npm run app:nobuild`.

The packaged app needs **Node installed on the machine**: the PTY and speech hosts run under the
system Node, because their native bindings target Node's ABI and are not rebuilt for Electron.

When returning the path of a generated installer or bundle, always report the **full absolute path**
(for example, `/home/user/projects/arco/dist-electron/Arco-2.0.0.AppImage`), never just the path
relative to the repository.



## 5. Non-negotiable rules

1. **DO NOT stop or restart the app or the dev server** (the shell / Vite). Do not kill the
   process, do not run `npm run app` "just to test" if it is already running. Apply changes through
   **HMR** and trust the reload.
2. **DO NOT commit / push / tag / release without explicit permission from the owner at that
   moment.** Make changes **in the working tree only** and stop — committing is his call. When he
   authorizes a commit, **DO NOT add a co-author** (`Co-Authored-By: Claude …`) or any tool
   signature to the message — he is the only author.
3. **Strict design system — no gradients, nothing "vibecoded".** No generic template UI. Dashboards
   and widgets show **real data**, never placeholder/mock. Style through CSS Modules + tokens from
   `src/styles/theme.css`; **never** hardcode a color — use the variables (`--bg`, `--fg`,
   `--accent`, `--agent-*`, `--status-*`, etc.).
4. **i18n is mandatory.** Every visible string goes through `t()`. When adding text, register the key
   in `src/lib/i18n/messages/en.ts` (**source of truth**, default EN) **and** in
   `src/lib/i18n/messages/pt-BR.ts`. `pt-BR.ts` is typed against the keys of `en.ts`, so
   `npm run build` **fails** if a translation is missing.
5. **Changelog is mandatory for features.** Every feature addition, change, or removal must update
   [`docs/CHANGELOG.md`](docs/CHANGELOG.md) in the same task, under the **`[Unreleased]`** section
   (top of the file), with a short, objective, user-facing description. Never skip this step — the
   changelog is the source for release notes.
6. **Releasing follows [`docs/RELEASE.md`](docs/RELEASE.md), in order.** The `/release` skill in
   `.claude/skills/release/` runs it end to end. The changelog is not the
   only place a release has to touch: the dialog the user reads lives in `src/lib/changelogData.ts`
   plus the `whatsNew.*` keys in both message files, and a version shipped without an entry there
   tells the user the update never arrived. `main` must contain the release before it is cut —
   cutting from a branch leaves the next version number below what is already installed. Both
   mistakes have shipped; `npm run release` now refuses to run when either is present.

7. **Speed is the product.** Arco is judged by how fast it feels — typing, switching a session,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devmatheusmota/arco](https://github.com/devmatheusmota/arco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
