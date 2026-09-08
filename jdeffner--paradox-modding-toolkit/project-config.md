---
trigger: always_on
description: Agent-facing guide for this repo. `CLAUDE.md` is `@AGENTS.md`.
---

# AGENTS.md

Agent-facing guide for this repo. `CLAUDE.md` is `@AGENTS.md`.

## What this project is

The **Paradox Modding Toolkit** (`JDeffner.px-toolkit`): a VS Code extension
plus a standalone LSP server (`@px-lsp/server`) for Paradox/Jomini script
modding across Crusader Kings III, Victoria 3 and Europa Universalis V.
Tolerant parser, scope-aware completion, hover docs, structural diagnostics,
ck3-tiger integration, event graph, GUI editor with a pixel-calibrated layout
engine, DDS tooling, localization workflow. The server also serves bare LSP
clients (neovim) and an external WPF IDE (Sage's Clausewitz Studio, owned by
lennart99v).

**Core design idea:** all language knowledge is *derived from the game
itself* (the user's `script_docs` logs, vanilla files, harvested `_*.info`
docs, real-corpus usage counts), never hand-maintained rule files.

Terms: **mod** = user content package; **vanilla** = shipped game files,
indexed read-only, never diagnosed; **`gameId`** = `ck3`/`vic3`/`eu5`, one
per workspace; **GameProfile** = the boundary behind which ALL game-specific
knowledge lives (`packages/server/src/games/<id>/`); **schema** = per-game
folder-to-definition-kind table; **harvest** = build-time script output in
`data/<id>/*.json`; **tiger** = ck3-tiger/vic3-tiger; **loc** = Paradox
localization (`*_l_<lang>.yml`, UTF-8 with BOM).

## Hard rules (no override; if a task fights one, stop and ask the maintainer)

1. **Never commit or push to `main`.** `main` = one squash commit per PR,
   created only by the maintainer. Check your branch before the first edit; never use
   a worktree with `main` checked out. Old release recipes that push `main`
   directly are superseded.
2. **Never hand-code game knowledge from memory.** Find it in the game
   install, the `_*.info` docs, a `script_docs` dump or a harvest, or don't
   add it.
3. **No Studio-origin content in a commit.** `docs/reference/studio/`
   (gitignored) is C# for human consultation only. Nothing from it may be
   translated, ported or committed into this GPL repo; GUI parity is a
   spec-driven rebuild from `docs/gui-designer/parity-checklist.md` with a
   design-credit header (see `gui/sourceEdit*`).
4. **No machine paths in tracked files.** They live in `dev-paths.json`
   (gitignored) or env vars.

## Invariants

- **AD-5 "annotate, never hide":** scope inference ranks and labels
  completion items but emits zero diagnostics and never removes an item for
  scope reasons. (Server-side word-filtering/capping is fine.)
- **`px` names the product, `ck3` names the game.** Ours: `px-toolkit`,
  `px.*` settings/commands, `@px-lsp/*`, `# px:ignore`. The game's:
  `gameId`, `games/ck3/`, `data/ck3/`, `ck3-tiger`, `ck3-script`,
  `zzz_ck3_modding_edits_*.yml` (`.ck3modding/` is the pre-0.4.0 name of
  the `.px-toolkit/` config dir, read as a fallback). The `paradox*` language ids
  and `paradox/*` wire methods mean "the engine family"; do NOT rename them.
- **Deep validation belongs to ck3-tiger.** Our diagnostics stay structural
  and certain (braces, encodings, folder traps).
- **The games fail silently**, so every writer produces files correct by
  construction: loc yml = UTF-8 **with BOM** + `l_<lang>:` header +
  `_l_<lang>.yml` filename; script `.txt` = UTF-8 with BOM; event files
  START with their `namespace =` line.
- **`localization/replace/` only overrides vanilla keys.** New keys go to
  the mod loc file holding their siblings (`writeLocSmart` /
  `upsertNewModLoc` in `packages/vscode/src/locCommands.ts`).
- **Override rules:** script databases are last-in-wins, `gui/` and
  `localization/replace` are first-in-wins.
- **No `vscode` imports** in `packages/server` or `packages/protocol`
  modules that carry logic; they must be unit-testable in plain Node.
- The parse cache (`packages/server/src/parseCache.ts`) is keyed by
  **uri + version**. In tests and scripts, use a fresh URI per document
  text or you get a stale parse.

## Hit-every-surface checklist

The most common defect: a change that works on the path you tested and is
missing everywhere else. Before calling work done, walk these and say which
applied:

| Axis | Question |
|---|---|
| Games | One decision per GameProfile (`ck3`, `vic3`, `eu5`), even "not supported". Gate on profile data, not `if (gameId === ...)`; the boundary check enforces it. |
| Clients | VS Code is the rich client; bare LSP clients and the Studio get degraded-but-honest behavior via capability gates (`clientCommands`, `snippetSupport`, `fileLinks`), never broken markup or dead links. |
| Entry points | A feature usually also needs: command palette entry, Project-panel row, `when`-scoped keybinding, walkthrough mention. |
| Contracts | Anything on the wire is typed in `packages/protocol` and documented in `docs/PROTOCOL.md`; embedder-visible behavior also in `docs/EMBEDDING.md`. Changing either doc means porting it to its wiki mirror in the same session. |
| Data | Per-game bundled data lives in `data/<id>/`; a new harvest needs a regen script row below and a `--game` flag. |
| Change notes | The changelog bullet ships in the same PR. |

## Repo map

| Path | What lives there |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JDeffner/paradox-modding-toolkit](https://github.com/JDeffner/paradox-modding-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
