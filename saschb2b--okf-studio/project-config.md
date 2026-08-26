---
trigger: always_on
description: A cross-platform **desktop workspace** (Windows + Ubuntu, macOS for free) that autodetects OKF bundles, renders their connected concepts as a graph and reader, and lets the user connect an external ACP agent or Studio Agent to create, curate, and query knowledge through explicit context and reviewed writes. Built with **Tauri 2.0** — a Rust core plus the system webview.
---

# OKF Studio

A cross-platform **desktop workspace** (Windows + Ubuntu, macOS for free) that autodetects OKF bundles, renders their connected concepts as a graph and reader, and lets the user connect an external ACP agent or Studio Agent to create, curate, and query knowledge through explicit context and reviewed writes. Built with **Tauri 2.0** — a Rust core plus the system webview.

## How it's put together

The **Rust core** (`crates/okf-core` + `src-tauri/`) owns filesystem, process, and network mediation: it [scans a folder for bundles](docs/architecture/bundle-detection.md), [parses concepts/links/backlinks](docs/architecture/okf-parsing.md), [validates](docs/features/validation.md), [watches for changes](docs/features/live-reload.md), and hosts explicit [agent connections](docs/architecture/agent-system.md). Its typed [command/event surface](docs/architecture/ipc-and-security.md) hands the **React 19 + TypeScript frontend** (`src/`, Vite, React Compiler enabled — no manual memoization) ready-to-render state. The frontend combines the [graph](docs/features/graph-view.md), [concept reader](docs/features/concept-reader.md), [search](docs/features/search-and-filter.md), [navigation](docs/features/navigation.md), and [Agent Panel](docs/features/agent-panel.md). Folder opening remains read-only; agent processes and network actions start only after explicit user actions.

```
okf-studio/
  src/             # web frontend (React 19 + TS, Vite) — domain-first: features/ + shared/
  src-tauri/       # Tauri shell: tauri.conf.json, capabilities/, commands
  crates/okf-core/ # Rust parsing/validation core (unit-tested against docs/)
  docs/            # the OKF product spec bundle — source of truth AND built-in sample
  design-system/   # ODSF bundle: the marketing site's visual language
  site/            # marketing/landing page (Astro) — see site/README.md
  scripts/         # okf-validate.mjs (vendored conformance checker), gen-icon.mjs
```

**The spec in [`docs/`](docs/) is the source of truth.** Read it before changing behavior, and keep it in sync when you do. Start at [`docs/index.md`](docs/index.md):

- **What it is / why:** [`docs/product/overview.md`](docs/product/overview.md), [`docs/product/principles.md`](docs/product/principles.md), [`docs/product/scope-and-non-goals.md`](docs/product/scope-and-non-goals.md)
- **What it does:** the [`docs/features/`](docs/features/) concepts — folder autodetect, bundle browser, graph view, concept reader, search & filter, navigation, validation, live reload.
- **How it feels:** [`docs/ux/`](docs/ux/) — first run, browsing layout, keyboard shortcuts, theming.
- **How it's built:** [`docs/architecture/`](docs/architecture/) — tech stack, bundle detection, OKF parsing, data model, IPC & security.
- **External specs:** [`docs/reference/`](docs/reference/) — OKF spec summary, Tauri 2.0 notes, glossary.

If anything in this file conflicts with the bundle, the bundle wins; update this file to match.

## Where code lives (folder structure)

Both the frontend and the agent backend are organized **domain-first**: the domain is the top-level unit, and narrower separation (`components/`, sub-parsers) nests *inside* it. Group by feature first, then by kind — never a global `components/` tree beside a parallel utilities tree. When you add a file, place it in its domain, not at the root.

**Frontend (`src/`).** Domains live under `src/features/<domain>/`, each owning a `components/` folder beside its own logic:

- `agent/` — ACP client (connection, catalog, install, threads, local models, custom profiles) + agent-panel components + staged-write review previews
- `git/` — repository snapshot and diff state + Git panel, focus contract, and dedicated diff workspace
- `viz/` — the graph engine (`graph/`), chart helpers, and every graph/chart component
- `reader/` — concept reader, prefs, lineage panel, peek card + lineage derivation
- `bundle/` — Bundle Home, bundle details and sharing, bundle browsing, and open-from-URL (`remoteSource` parser)
- `navigation/` — sidebar, index tree, tag browser, type filters
- `shell/` — window frame and global overlays (top/status/activity bar, tabs, command palette, settings, validation/log panels)

Cross-cutting code lives in `src/shared/`: `ipc`, `store`, `types`, `query`, `selectors`, `odsf`, `theme`, `render/` (markdown/math/mermaid/highlight), `platform/` (native/window/updater/platform), and `styles/` (`baseui.css`, `chrome.css`). Only the composition root stays at the `src/` top level — `App`, `main`, `keys` — plus `mock/` and `test/` infrastructure and the `integration/` lane. Full-app journeys live in `src/integration/`, named for the surface they cover (`connections.test.tsx`, `shell.test.tsx`), because they belong to no single domain; the directory, not a filename infix, is what puts them in that lane. A component's own `.css` and `.test` file sit beside it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saschb2b/okf-studio](https://github.com/saschb2b/okf-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
