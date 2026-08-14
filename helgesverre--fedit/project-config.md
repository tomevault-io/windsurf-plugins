---
trigger: always_on
description: A small terminal text editor written in F#. Pure-data MVU/Elmish loop.
---

# fedit — agent guide

A small terminal text editor written in F#. Pure-data MVU/Elmish loop.
Read [`README.md`](README.md) for the user-facing intro; this file is
for agents (and humans skimming the project conventions).

## Architecture in one minute

```
KeyPressed / Resize → Msg → Editor.update (pure) → (Model', [Effect])
                                                     ↓
                                            runEffect (impure I/O)
                                                     ↓
                                                    Msg → loop
                       Model → Layout.render (pure) → Screen → Renderer (ANSI)
```

- **Model** is pure data (workspace tree, buffers, cursors, focus, theme, panels).
- **Editor.update** is the only place state transitions live. Returns `(Model', Effect list)`.
- **runEffect** is the only impure path — file I/O, clipboard, config writes. Effects post results as `Msg` into a `ConcurrentQueue` drained each tick.
- **Buffers** use a piece table (`PieceTable.fs`); each buffer owns its undo/redo stack.
- **Hex views** (auto-detected binary files, `:hex`) are ordinary buffers holding the latin1 projection of the raw bytes (char offset = byte offset), marked by an entry in `Model.HexViews`. Row geometry lives in `Hex.fs` (`Hex.layoutFor`) and is shared by the renderer, mouse hit-testing, and cursor movement — the `Dock.metrics` convention. Saves go through `writeAllBytesAtomic` (byte-exact); the first save over an existing file copies the original to `<name>.bak` (`File.backupOnce`, never clobbers an existing backup); highlighting and LSP skip hex buffers.
- **Themes** own the full chrome surface — accent plus an explicit fg/bg per region (editor, gutter, prompt, dock, status, selection, active line). Bundled dark themes set `Default` backgrounds so they keep terminal-default chrome; a light theme supplies real backgrounds.

Source file order (`<Compile>` in `src/Fedit/Fedit.fsproj` is canonical):
`Primitives → Keys → Events → TerminalCapabilities → MouseProtocol → ImageProtocol → KittyImage → PieceTable → Buffer → Hex → Workspace → Screen → Color → Themes → Highlight → Commands → Actions → Keymap → Plugins → PluginWire → PluginProtocol → PluginHostClient → LspTypes → LspWire → LspTransport → LspClient → PickerTypes → PromptTypes → Model → Config → Pickers → KeymapIO → MacroIO → Prompt → Dock → Editor → Status → Renderer → Input → View → Terminal → Runtime → Cli → Cli/Commands/* → Program`.

`Primitives.fs` also holds `Paths` (`norm`/`parent`): fedit uses a **canonical
forward-slash path model** — normalize any path crossing an OS boundary
(tree scan, file open, workspace root) with `Paths.norm`. .NET's file APIs
accept `/` on Windows, so normalized paths still do real I/O. Don't introduce
`Path.GetDirectoryName`/`GetFullPath` on compared/displayed paths (they emit
`\` / drive-anchor on Windows) — use `Paths.parent` and combine+normalize.

`Dock.fs` owns the dock/editor layout geometry (`Dock.metrics`) shared by
`Editor` (mouse hit-testing) and `View.Layout.render` (painting) — change
layout arithmetic there, never in just one consumer.

## Building & testing

The repo ships a pinned `.dotnet` SDK (10.0.x). Recipes prepend it to `PATH` — never `dotnet` directly outside a recipe; use `just` or invoke
the wrapper script `./fedit`.

```
just check        # lint + build + test — pre-commit gate
just dev .        # dotnet watch on src/Fedit
just run .        # one-shot run
just test         # xUnit + FsCheck (Tier 1)
just bench        # BenchmarkDotNet micro suite (Release, ~4 min; filterable)
just bench-manual # frame-pipeline + tree-sitter parse harness (~1-2 min)
just format       # fantomas + oxfmt on **/*.md
just lint         # check-only of the same
```

Website lives under `website/` (Astro 6 + bun) — see `just website::dev|build|check|lint|format`.

## Brand & themes

Source of truth: [`brand/`](brand/). One symbol (caret), one workhorse
mono (Departure for brand, JetBrains Mono for code), one accent
(`#00B86B`), 13 selectable themes. Brand bans purple/magenta and AI-slop
patterns (Inter, gradients, glassmorphism, bento, centered hero — see
`brand/USAGE.md`).

Themes are implemented in `src/Fedit/Themes.fs`; spec mirrors live in
`brand/themes/*.json`. Adding a new theme = add a record to `Themes.all`
and a matching JSON doc; don't reintroduce banned colors.

Voice rules ([`brand/voice.md`](brand/voice.md)) apply to README, CLI
help, error messages, **commit messages**, and release notes. No emoji,
no marketing adjectives, lead with the verb.

## Release process

```
just release 0.1.0    # tags vX.Y.Z, pushes, triggers CI
```

`.github/workflows/release.yml` builds two flavors across 5 RIDs: the
**default NativeAOT** archives (`fedit-<triple>`, ~7 MB, ~10 ms first
paint — what Homebrew and the installers pull) and the **opt-in R2R
fallback** (`fedit-r2r-<triple>`, self-contained ReadyToRun). NativeAOT
can't cross-compile across OS (and Linux not across arch), so the AOT job
uses native-arch runners (`linux-arm64` on a native ARM runner; `osx-x64`
cross-built from the arm64 macOS runner via the universal toolchain). It
uploads archives + SHA256 sidecars to the GitHub Release, renders
[`scripts/fedit.rb.tmpl`](scripts/fedit.rb.tmpl) via

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HelgeSverre/fedit](https://github.com/HelgeSverre/fedit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
