---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project intent

A Unity package of Roslyn **source generators** plus a Unity project that consumes and tests them. Authoritative intent lives in `docs/Directives.md` — read it before structural decisions. `README.md` is the user-facing pitch and is the closest thing to a spec for the `[Singleton]` / `[Dependency]` semantics.

Currently shipped: `[Singleton]` (with optional interface arg, `[SingletonInclude]`, `[SingletonIgnore]`) and `[Dependency]`. Each generator has a companion analyzer that surfaces misuse as diagnostics.

## Repository layout

Two **independently-versioned** but tightly-coupled trees:

- `src/generators-src/EngineRoom.Generators.sln` — the Roslyn generator project (`netstandard2.0`). Source lives under `EngineRoom.Generators/{Singleton,Dependency,Helpers,Templates}/`. Templates are embedded `.cs.txt` resources, loaded by `Helpers/TemplateLoader`.
- `src/generators-unity/` — Unity 6 project (`6000.4.0f1`, URP). Two roles:
  - `Packages/games.engine-room.generators/` — the **UPM package** that ships to consumers. Contains `Runtime/Singleton/Attributes/*.cs` (the public attribute types, plus `ISingleton<T>`), and `Runtime/Plugins/EngineRoom.Generators.dll` (the built generator, with checked-in `.dll.meta`).
  - `Assets/Demo/Singletons/` — demo + tests that exercise the generators against the package. Two asmdefs: `EngineRoom.Demo.Singletons` (the demo MonoBehaviours) and `EngineRoom.Demo.Singletons.Tests` (NUnit, Editor-only, gated by `UNITY_INCLUDE_TESTS`).
- `tests/generators-tests-2022-3-62/` — a **second Unity project on Unity 2022.3.62** purely to verify the package still loads on the older supported version (README declares Unity 2022.3+).
- `docs/Directives.md` — the non-obvious rules:
  - Templates ship as **embedded text resources** inside the dll (not string literals in code).
  - Placeholders use `%%Name%%` (double percent), e.g. `%%ClassName%%`.
  - When something a generator needs looks reusable, extract it to `Helpers/` **immediately** — don't wait for the second caller.
- `docs/Coding Conventions.md` — naming/formatting rules for **both** generator C# and Unity C#. The ones that bite:
  - Allman braces, always present (even one-liners).
  - Private fields `_camelCase`; serialized private fields `[SerializeField] private`; **no public fields ever** — expose serialized values via `[field: SerializeField]` properties, never auto-properties.
  - File member order: constants → static → events → properties → fields → readonly → ctors → public methods → private methods → Dispose/OnDestroy → destructor → nested types. Within each, public → protected → private.
  - Flags enums: plural name, `1 << n`, `0 = None`. Non-flags: singular, reserve `0` for None/Empty.
  - No `#region`. No abbreviations (except math). `nameof()` for member references; interpolation over concatenation.
  - **Comments are rare** — default to none. Only when the WHY is non-obvious.

## Two halves, one source of truth for attributes

The generator project and the Unity runtime package both need to know about the attribute types (`SingletonAttribute`, `DependencyAttribute`, `SingletonIncludeAttribute`, `SingletonIgnoreAttribute`). To avoid drift, the attributes live **once**, as plain `.cs` in the UPM package under `Packages/games.engine-room.generators/Runtime/Singleton/Attributes/`, and the generator csproj **links them into its own compilation** via `<Compile Include="..\..\generators-unity\Packages\...\*.cs" Link="..." />`.

Consequence: **adding a new public attribute type means adding a new `<Compile Include>` line** in `src/generators-src/EngineRoom.Generators/EngineRoom.Generators.csproj`. Otherwise the generator references it textually only (via constants in `*Constants.cs`) and you lose the `typeof()` safety the csproj is set up for.

`ISingleton<T>` is intentionally NOT linked: it uses static interface members that `netstandard2.0` can't compile. The generator references it textually in templates only.

## Build & run

No top-level build script. Two independent toolchains:

- **Generator:** `dotnet build src/generators-src/EngineRoom.Generators.sln -c Release`. A post-build target (`CopyToUnityPackage`, Release-only) drops the dll into `src/generators-unity/Packages/games.engine-room.generators/Runtime/Plugins/EngineRoom.Generators.dll` automatically. Debug builds do not copy — if you're iterating on the generator and want Unity to pick it up, build Release. After the copy, Unity needs an asset refresh (it usually happens on focus; if not, right-click → Reimport on the dll, or use Assets → Refresh).
- **Unity project:** open `src/generators-unity/` in Unity `6000.4.0f1` (match `ProjectSettings/ProjectVersion.txt` or Unity will silently upgrade). The `.sln`/`.csproj` files in this folder are **Unity-generated** — never hand-edit.
- **2022 compat check:** open `tests/generators-tests-2022-3-62/` in Unity 2022.3.62 to confirm the package still loads/compiles on the older floor.

## Unity source-generator requirements (do not deviate)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Engine-Room-Games/MGenerators](https://github.com/Engine-Room-Games/MGenerators) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
