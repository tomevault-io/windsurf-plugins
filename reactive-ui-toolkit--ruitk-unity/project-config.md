---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ReactiveUIToolkit brings a React-like component model (function components, hooks, a virtual node tree, typed props, a Fiber reconciler) to Unity UI Toolkit — all in C# on top of UI Toolkit. The repo is more than a Unity package: it's a monorepo that also contains a Roslyn source generator for the `.uitkx` markup language, a shared language library, a cross-editor LSP server, four IDE extensions, and a docs website. Full docs: http://reactiveuitoolkit.info/

## The `~` folder convention (read first)

Unity's Asset Database ignores any folder whose name ends in `~`. This is deliberate and load-bearing: everything that is *tooling* rather than *shipped runtime code* lives under a `~` folder so Unity never tries to compile it. When editing, know which world a file is in — the same code is often **linked** into multiple projects (see below), so a change in `Shared/` can affect the generator, the LSP server, and the Unity runtime at once.

| Path | Unity sees it? | Build world |
|---|---|---|
| `Runtime/`, `Shared/`, `Editor/`, `Samples/`, `Diagnostics/`, `CICD/` | yes (asmdefs) | Unity C# |
| `Analyzers/` | yes (committed generator DLLs) | consumed by Unity as an analyzer |
| `SourceGenerator~/` | no | Roslyn generator, `netstandard2.0` (tests: `net10.0`) |
| `ide-extensions~/language-lib/` | no | shared language lib, `netstandard2.0` |
| `ide-extensions~/lsp-server/` | no | LSP server, `net8.0` |
| `ide-extensions~/{vscode,visual-studio,rider}/` | no | IDE extensions (Node/VSIX) |
| `ReactiveUIToolkitDocs~/` | no | Vite + React docs site |

## Architecture

### Runtime rendering pipeline (the Unity side)

The render path is React-shaped: **`V.*` factory calls → `VirtualNode` tree → `FiberReconciler` → `IElementAdapter`s → Unity `VisualElement`s.**

- **`Shared/` is the core** and is host-agnostic. It contains the whole reactive engine: `V.cs` (element/component factories), `VNode.cs` (pooled virtual nodes), `Hooks.cs` + `HookRegistry.cs`, the `Core/Fiber/` reconciler (`FiberReconciler`, `FiberNode`, `FiberRoot`, child reconciliation, commit/effect phases, time-slicing at ~2 ms), `Core/Signals/`, `Core/Router/` (React-Router-style routing), `Core/Refresh/` (Fast Refresh families), `Elements/` (one `*ElementAdapter` per UI Toolkit control, resolved through `ElementRegistry`), and `Props/` (the typed `Style` system + appliers).
- **`Runtime/` is a thin MonoBehaviour adapter** — `RootRenderer` (the `MonoBehaviour` host that mounts a fiber tree onto a `UIDocument`/`VisualElement`) and `RenderScheduler`. Keep host-specific Unity glue here; keep engine logic in `Shared/`.
- **`Editor/` is Unity Editor integration** — HMR (`Editor/HMR/`), the `.uitkx` change watcher/generator trigger, console navigation, csproj post-processing. HMR recompiles `.uitkx` in-editor and hot-swaps delegates/modules without a domain reload (~50–200 ms).

Editor-only defenses (e.g. `RootRenderer`'s `UIDocument` host-rebuild polling for Unity 6.3 `UUM-127851`) are gated behind `#if UNITY_EDITOR` so they compile out of player builds. Preserve those gates.

### UITKX language pipeline (the tooling side)

`.uitkx` is JSX-like markup that a Roslyn source generator compiles to a C# partial class at build time (zero runtime overhead). The pipeline (`SourceGenerator~/UitkxPipeline.cs`) is four stages: **DirectiveParser** (preamble + declarations) → **UitkxParser** (recursive-descent markup → AST) → **PropsResolver** (tag names → `V.*` call patterns) → **CSharpEmitter**/**ExportsEmitter**. Diagnostics use `UITKX####` codes.

Since 0.9.0 (ES-modules redesign): a file IS a module. Plain typed `export` declarations (`export VirtualNode Name(...) {…}`, `export (ret) useX(...) {…}`, `export Style x = …;`) replace the deprecated `component`/`hook`/`module` wrapper keywords (UITKX2320 window; classification is signature-driven). Namespaces are FILE-keyed (folder segments + file stem) for new-syntax files; values/utils/hooks emit onto a per-file `public static partial class __Exports`; the full ES import surface is live (`import { a as b }`, `import * as X` + dotted tags, default imports, `export { … }` lists). Companion partial-merging is deprecated (UITKX2107). Emitted usings for new-mode units go INSIDE the namespace block, `global::`-qualified (file-level aliases are shadowed by sibling file-stem namespaces). The codemod is `UitkxMigrateImports --es-modules`.

The actual parser/lexer/AST/lowering/formatter/IntelliSense lives in **`ide-extensions~/language-lib/`** (`Ruitk.Language`), *not* in the generator. Both the generator and the LSP server reference it, so parsing behaves identically in a Unity build and in every editor. `HookRegistry.cs` is the single source of truth for hook metadata and is **`<Compile Include>`-linked** from `Shared/Core/` into the language lib — the generator, analyzer, LSP hover, and Unity runtime all read the same table. There are explicit *parity/contract* tests (`Hmr*ContractTests`, `AsmdefResolverParityTests`) guarding that the HMR emitter and generator emitter stay in lockstep; if you change one emitter, expect to update the other.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactive-ui-toolkit/ruitk-unity](https://github.com/reactive-ui-toolkit/ruitk-unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
