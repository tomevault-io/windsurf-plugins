---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is the **development project** for **Velvet**, a React-style declarative UI framework for Unity UI Toolkit. The actual product is the embedded package at `Packages/com.velvet.core/` (the source of truth); the surrounding Unity project exists only to build and test it. The package is published to a separate `upm` branch (package-at-root) by CI — never edit there; edit in place under `Packages/com.velvet.core/`.

Guiding principle (from the README): **reproduce React's semantics as faithfully as possible**, deviating only where a C#/Unity constraint makes the deviation a clear improvement. When unsure whether a behavior is "correct," the answer is "what React does."

- **Unity 6000.3.11f1** (Unity 6.3 LTS) is the validated/floor version (`ProjectSettings/ProjectVersion.txt`). Bundled USS uses 6.3-only properties.
- C# root namespace is `Velvet` for the runtime. Namespaces are declared per-file and do NOT track folders — moving a file does not change its namespace.

## Running tests (headless / CLI)

Unity test runs require the editor to be **closed** (it holds the project lock). On macOS the editor binary is `/Applications/Unity/Hub/Editor/6000.3.11f1/Unity.app/Contents/MacOS/Unity`.

```bash
UNITY=/Applications/Unity/Hub/Editor/6000.3.11f1/Unity.app/Contents/MacOS/Unity
"$UNITY" -runTests -batchmode -projectPath "$PWD" -testPlatform EditMode \
  -testResults /tmp/results.xml -logFile /tmp/run.log
```

- **Run a subset / single fixture:** add `-testFilter "Velvet.Tests.SomeFixture"` (semicolon-separates multiple; matches fully-qualified class or method names).
- **PlayMode:** `-testPlatform PlayMode`.
- **Do NOT pass `-nographics`.** Tests that mount a real panel (an `EditorWindow.rootVisualElement`, or anything reading `resolvedStyle` / firing pointer/focus events) fail with "No graphic device is available" under `-nographics`. Graphics-free tests still pass with graphics on, so just always omit the flag.
- Results land in the JUnit-style XML (`grep -o 'passed="[0-9]*"\|failed="[0-9]*"'`); compile errors appear only in the `-logFile` (`grep "error CS"`).
- Interactively, the same suites run from **Window ▸ General ▸ Test Runner**.

### Source generators (separate .NET solution)

The Roslyn analyzers/generators under `Packages/com.velvet.core/Generators~/` are a standalone dotnet solution (not part of the Unity build), pinned by `Generators~/global.json`. Their compiled DLLs in `Runtime/Plugins/` are **committed**.

```bash
cd Packages/com.velvet.core/Generators~
dotnet test Velvet.SourceGenerators.sln -c Release   # generator unit tests (no Unity license needed)
./build.sh                                            # rebuild + redeploy DLLs to Runtime/Plugins, then commit them
```

CI is split by what a change can affect: `.github/workflows/generators.yml` runs `source-generators` (no license) only for `Generators~/**` changes, and `.github/workflows/test.yml` runs `unity-tests` (EditMode/PlayMode, **skipped unless a `UNITY_LICENSE` secret is set** — see CONTRIBUTING.md) only for package/project changes; docs and markdown trigger neither. Docs (`docs/`) are DocFX-generated from XML comments via `docs/build.sh`.

## Architecture (the parts that span many files)

The render pipeline, in dependency order under `Runtime/`:

1. **`Component/`** — the `V.*` factories build an immutable **`VNode`** tree (`VNodeTypes.cs`); `V.Mount` attaches it. `V.Component(Foo.Render)` wraps a `[Component] static VNode Foo()`. `V.List` / `V.When` / `V.Fragment` / `V.Provider` are the JSX-construct equivalents. `VNodePool.cs` pools the poolable primitives.
2. **`Reconciler/`** — diffs the new VNode tree against the live fiber tree and patches the underlying `VisualElement`s. Key seams: `FiberRenderer` (mount/flush/dispose), `ChildReconciler` (keyed/positional diff + `FlattenAndFilter`, which drops `null` children — so `cond ? node : null` is the idiomatic "render nothing"), `FiberBatchScheduler` (lane-based, coalesced frame-boundary drains), `FiberNodeFactory`/`FiberNodePatcher` (create/patch + attach styling manipulators), `FiberElementCleaner` + `FiberPrimitiveElementPool`/`FiberElementPoolReset` (resource cleanup + reset-before-pool — a recurring bug class is state ghosting across pool reuse, so a reset helper must scrub **every** field a node may have set).
3. **`Hooks/`** — `Hooks.cs` is the public surface (`UseState`/`UseEffect`/`UseStore`/`UseRef`/…). Hook state lives on the `ComponentFiber`; `StateUpdater<T>` (the setter) is reference-stable across renders and supports the functional form `setX.Invoke(prev => next)`.
4. **`Store/`** — `Store<T>` (Zustand-style immutable state + subscribers); `UseStore(store, selector)` subscribes synchronously at render and unsubscribes on unmount.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s4k10503/velvet](https://github.com/s4k10503/velvet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
