---
trigger: always_on
description: Silo keeps **all your projects alive at once** — for developers juggling coding
---

# Silo — project guide for Claude

Silo keeps **all your projects alive at once** — for developers juggling coding
agents (Tauri + React + TypeScript). Open many workspaces and switch instantly;
each keeps its terminals, agents, panels, and layout intact in the background.
**100% open source**, free forever. **Extensible**: modeled on VS Code / Obsidian
with a small stable core, a public extension SDK, and first-party features built
as extensions — so the bar for boundaries, documentation, and a clean public
surface is high.

Product positioning for humans lives in `README.md` / `apps/docs/index.md` /
`context7.json`. Prefer those over inventing a tagline. This file is agent
orientation (architecture, boundaries, commands), not marketing copy.

Orientation docs (read when relevant):

- `docs/decisions/` — ADRs: the architecture decisions of record (the durable "why").
- `docs/proposals/` — RFCs: forward-looking designs not yet decided.
- `docs/ui-terminology.md` — high-level UI component naming.
- `docs/silo-extensions-repo.md` — the **external** `silo-code/silo-extensions`
  repo (cloned at `../silo-extensions`): how its third-party extensions relate to
  this repo — published-SDK lag, npm (not pnpm) build commands, runtime trust /
  `silo.permissions`, and how to install a branch without merging. They
  **publish into** the registry; discovery is Browse /
  [extensions.getsilo.dev](https://extensions.getsilo.dev), not cloning that repo.
- `docs/extensions-registry-repo.md` — the **external**
  `silo-code/extensions-registry` repo (cloned at `../extensions-registry`):
  git-backed catalog behind
  [extensions.getsilo.dev](https://extensions.getsilo.dev) (humans) and
  [registry.getsilo.dev](https://registry.getsilo.dev) (app/CLI index). How it
  relates to in-app Browse / install / update in this monorepo.
- `apps/docs/guide/extension-checklist.md` — pre-flight checklist for any
  extension (bundled or third-party): boundaries, permissions, styling,
  lifecycle, packaging, stability.

## Engineering principles

- Choose the simplest implementation that fully meets the current
  requirements. Avoid speculative abstractions, configuration, and indirection
  in implementation code. This doesn't apply to the public SDK surface
  (`ctx`, `@silo-code/sdk`) — that's designed ahead of full usage on purpose
  (see the roadmap's `planned` → `stable` flow above); the bar there is
  deliberate API design, not premature abstraction.
- Grow the system in layers. Start from the smallest version that works end
  to end, and add each new capability on top of a product that already
  works. Never trade a working product for unfinished complexity.
- Keep components modular and concerns clearly separated.
- Prefer established, well-maintained libraries over reimplementing common
  functionality, and lean on dependencies already in the project before
  adding new ones or writing your own. Don't assume a library lacks a
  capability without checking its docs and types.
- Make architectural decisions for the long term. Don't accept a stopgap
  that only works for now and is meant to be replaced later.

## Self-documentation — keep docs in sync AS YOU BUILD

The API reference is **generated from the source**, so touching the public
extension surface (the `@silo-code/sdk` barrel `packages/sdk/src/index.ts` and
everything it re-exports) means doing the docs in the **same change** — never as
a follow-up. When you add or change a public symbol — a new `ctx` method, type,
or field — run the full workflow (TSDoc, `@public`/`@internal` + `@category`
tags, barrel re-export, the hand-authored `ctx` member page, `pnpm docs:api`,
flipping the roadmap badge) and the Context7 indexing contract for
`context7.json`: the **`silo-docs-sync`** skill.

**Docs-driven development:** the public **Roadmap** (`apps/docs/roadmap.md`) is the
source of truth for what's real; design decisions live as ADRs (`docs/decisions/`)
and proposals (`docs/proposals/`). Design a new primitive by adding it to the
roadmap as `planned` (with its sketched surface) _first_, then implement it and
flip it to `stable`. Expanding `ExtensionContext` (`ctx`) is the main ongoing
work — and the moment to run `silo-docs-sync`.

## Architecture boundaries — enforced, don't regress

The repo is a pnpm workspace; the boundary is now expressed by the **package
graph**. The relevant packages:

- `@silo-code/sdk` (`packages/sdk`) — the public, types-first leaf. The only
  surface `silo.*` and third-party extensions may import.
- `@silo-code/extension-host` (`packages/extension-host`) — the workbench host
  runtime (state, services, layout, docked, panels, components, the registry +
  loader that provide `ctx`). Owns the **privileged**
  `@silo-code/extension-host/internal` subpath, importable by `core.*` only.
- `@silo-code/extensions-core` (`core.*`) / `@silo-code/extensions-silo`
  (`silo.*`) — the bundled first-party extensions.

Extensions must touch the app **only** through `ctx` and `@silo-code/sdk` types —
never the host's `state`/`services`/`layout`/`panels`/`docked`/`components`. This
is enforced **first by package visibility**: a package can only resolve what it
declares as a dependency. `@silo-code/extensions-silo` depends on `@silo-code/sdk`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silo-code/silo](https://github.com/silo-code/silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
