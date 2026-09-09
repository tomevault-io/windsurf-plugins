---
trigger: always_on
description: Guidance for AI assistants (and humans) working in this repo. This is the **project harness**:
---

# CLAUDE.md — ZettelFlow harness

Guidance for AI assistants (and humans) working in this repo. This is the **project harness**:
a high-signal map, the conventions, the commands, and the Obsidian-specific rules that keep
changes shippable. It is committed to git on purpose. Deep detail lives in [`docs/`](docs/) —
this file points you there rather than duplicating it.

## What this project is

**ZettelFlow** is an **Obsidian plugin** (TypeScript + React 19 + Zustand + CodeMirror 6, bundled
by esbuild) that turns a native **Canvas** into a note-creation **workflow**: a wizard walks the
canvas graph (root → steps → actions) and generates a note by merging step templates and running
each step's **actions**. It's a monorepo:

- `src/` + `manifest.json` — **the plugin** (the product).
- `docs/` + `mkdocs.yml` — the MkDocs Material site (GitHub Pages). The community gallery is
  **fully static**: `docs/main_template.json` + `docs/systems/*.zftemplate` served over GitHub raw
  (no backend — #294).

Current version: `3.1.0`, `minAppVersion 1.13.1`, desktop **and** mobile (`isDesktopOnly:false`).

## Architecture in 60 seconds

```
main.ts (Plugin)
 → starters/     bootstrap: ZComponents (RibbonIcon, SettingsTab, PluginApi), {{frontmatter}} processors
 → config/       ZettelFlowSettings + DEFAULT_SETTINGS + settings tab (General/Hooks/Developer, chain-of-responsibility)
 → architecture/ the internal framework:
     api/        ActionsStore (Map singleton), CustomZettelAction base, fnsManager (the `zf` script API), ZfVault/ZfScripts
     plugin/     ObsidianApi facade, Lifecycle, canvas/ (CanvasPatcher via monkey-around), services, VaultStateManager
     components/  core (CodeView = CodeMirror .js editor, search), settings modals
     monitoring/  Logger, custom exceptions   ·   patterns/ AbstractChain   ·   lang/ i18n (en/es)   ·   styles/ c() prefixer
 → actions/      11 built-in actions, each a 4-file bundle (Action/Component/Settings/SettingsReader)
 → application/  notes/ (NoteBuilder + ContentDTO + NoteDTO), components/noteBuilder (Zustand wizard), community/ (static GitHub-backed gallery)
 → zettelkasten/ step/flow editor modals (StepBuilderModal, SelectorMenuModal, installed editors)
 → hooks/        vault hooks: folder automation + property hooks + context-menu integrations
```

**The five layers (the organizing principle, epic #144).** Conceptually the code is understood as
**Knowledge Model** (the pure `architecture/knowledge/` idea graph) → **Workflow Engine** (canvas →
note: `main.ts`/`starters`, `architecture/api`, `actions`, `application/notes` engine + `zettelkasten`
+ `architecture/plugin` canvas/services + `hooks`) → **Knowledge State** (analyses over the model:
`architecture/knowledge/{debt,review,balance,discovery,map,traverse,questions,timeline,synthesis,dashboard,home,projects,journal}`)
→ **Experience** (`architecture/components/core` views + `config` settings UX) → **Community Gallery**
(`application/community` — the Systems Gallery), over a cross-cutting **Foundation**
(`lang`/`styles`/`monitoring`/`ai`/`plugin` facade). Every capability has one home layer; the full
inventory is [`docs/architecture/reposition-map.md`](docs/architecture/reposition-map.md). The `src/`
folders are being repositioned to match, incrementally — nothing is deleted or renamed.

Full detail: [`docs/architecture/overview.md`](docs/architecture/overview.md) and the pages it
links (plugin core, actions & note builder, vault hooks, the static community gallery).

## Commands

| Task | Command |
|---|---|
| Dev build + watch | `npm run dev` |
| Dev build + watch + auto-deploy to test vault | `npm run dev:vault` (needs `.vault-path`) |
| One-shot deploy to test vault | `npm run deploy:vault` (needs `.vault-path`) |
| Production build (type-check + minify → `dist/`) | `npm run release` |
| Lint (blocking) | `npm run lint` (oxlint) / `npm run lint:fix` |
| Type-check (blocking) | `npm run typecheck` |
| Test — TDD (blocking) | `npm test` / `npm run test:watch` / `npm run test:coverage` |
| Verify all (pre-push + CI) | `npm run verify` |
| Obsidian-guideline lint (blocking) | `npm run lint:obsidian` — clean (0), part of `verify`/CI |
| Docs preview | `mkdocs serve` |

Build output (`dist/`) is git-ignored. Releases are cut by **pushing a git tag**
(`.github/workflows/releases.yml` uploads `main.js`/`manifest.json`/`styles.css`); docs deploy on
push to `main`.

## Conventions (follow these)

- **Commits:** Conventional Commits, enforced by a commit-msg hook
  (`feat(scope): …`, `fix: …`, `docs: …`). Pre-commit runs `npm run lint`.
  **Do not add a `Co-Authored-By: Claude` (or any AI) trailer to commit messages.**
- **Branches:** work on `feature/*`; open PRs into `main`. Only commit/push when asked.
- **Imports:** bare-specifier aliases via `tsconfig` `baseUrl: src` — `architecture`, `config`,
  `actions`, `application`, `hooks`, `zettelkasten`, `starters`. No `paths` map.
- **Logging:** use `log` from `architecture`, never bare `console.*`.
- **Obsidian API:** go through the `ObsidianApi` facade / the `Vault` API; avoid global `app` and
  the `Adapter` API.
- **DOM:** build with `createEl`/`createDiv`/`createSpan`, clear with `el.empty()` —
  **never `innerHTML`**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RafaelGB/Obsidian-ZettelFlow](https://github.com/RafaelGB/Obsidian-ZettelFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
