---
trigger: always_on
description: This file provides guidance to coding agents — Codex, Claude Code (claude.ai/code), and others — when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents — Codex, Claude Code (claude.ai/code), and others — when working with code in this repository.

> **This file is the single source of project instructions for every agent.** Codex and other AGENTS.md-aware tools read it directly; Claude Code reads it through a one-line `@AGENTS.md` import in `CLAUDE.md`. Put project instructions here, never in `CLAUDE.md` — anything added there is invisible to every other agent.

## Working Style

The repository owner develops hands-on and tracks the implementation themselves. By default, **investigate and explain — do not modify code.** When asked about a bug or behavior, report the root cause, the relevant files/lines, and (optionally) how a fix would look, but leave the actual implementation to the owner unless they explicitly ask you to make the change.

## Output Language

**Write all output to the repository owner in Japanese.** This is a hard requirement, not a preference.

- **Code review** — findings, severity reasoning, and suggested fixes are written in Japanese, even though the code and identifiers under review are English.
- **Investigation reports** — root cause, affected files/lines, and how a fix would look.
- **Commit messages and PR titles/bodies.**

Do not translate identifiers, type names, file paths, or code fences — those stay verbatim as they appear in the source.

Rationale: FloatSoda treats **Japanese as the neutral (default) language** — see `docs/Localization.md`. `docs/`, XML doc comments, and exception messages are all Japanese by design. Review output in Japanese keeps the entire surface consistent for the owner and for the LLMs that relay it to end users. If you feel English "should" be the default here, read `docs/Localization.md` before changing anything.

## Project Overview

FloatSoda is a SteamVR Overlay UI framework for .NET 10 / C# 14 that brings Flutter-like declarative UI to VR overlays. It renders via SkiaSharp → OpenGL (GLFW/OpenTK) → OpenVR overlay texture.

## Target Users

FloatSoda targets three personas. Use them as the yardstick for API design, docs tone, and error messages:

1. **VRChatters who vibe-code personal tools** — they barely write code themselves; an LLM does. The real "reader" of `docs/` and the API surface is the LLM, so optimize for "an LLM cannot misuse this API." Concrete wants: a FaceEmo expression switcher (OSC), a VRChat photo album, a friend-online toast notifier.
2. **Booth creators selling overlays** — Unity-native programmers capable of building a game world, but they only know uGUI and have never seen declarative UI. Explain concepts by mapping from Unity/Udon vocabulary. They can handle exe distribution given a guide.
3. **Engineers who avoid uGUI** — their core pain is that uGUI is not text-based (scenes/prefabs don't diff, review, or LLM-generate). The headline value: **the entire UI lives in C# code — no scenes, no prefabs**. Never introduce state that can't be expressed in code (e.g. external asset/config files) without weighing this trade-off.

## Build & Run Commands

```bash
# Run the sample overlay app
dotnet run --project samples/FloatSoda.Samples.OverlayApp

# Build the whole solution
dotnet build

# Run all tests
dotnet test

# Run a specific test project
dotnet test tests/FloatSoda.Test
dotnet test tests/FloatSoda.Rendering.Test

# Run a specific test by name
dotnet test tests/FloatSoda.Test --filter "FullyQualifiedName~AlignmentTest.ComputeOffset"
```

Tests use xunit. `FloatSoda.Test` tests geometry types, RenderObjects, and Widgets. `FloatSoda.Rendering.Test` tests the Layer tree.

### Test naming

Test method names follow **`MemberName_条件_期待結果`**: the leading member name stays English, the condition and expected result are written in **Japanese**. Test class names stay English (they mirror the type under test).

```csharp
[Fact]
public void ComputeOffset_親子が同サイズ_原点を返す() { ... }

[Fact]
public void BorderSide_Widthが負_ArgumentOutOfRangeExceptionを投げる() { ... }
```

Do not use `[Fact(DisplayName = "…")]` — it duplicates the intent and drifts. Do not bulk-rename existing tests; apply this to new tests only. Full rules: `CONTRIBUTING.md`.

## Project Structure

| Project | Role |
|---|---|
| `src/FloatSoda.Abstractions` | Shared engine contracts, geometry types, input events, and frame pacing |
| `src/FloatSoda.Rendering` | Skia layer tree, shared Layer renderer, and bitmap rendering |
| `src/FloatSoda.Engine` | Platform layer: GLFW/OpenGL (`GLView`), `Renderer`, `OverlayWindow`, `RenderThreadRunner`, `FramePacer` |
| `src/FloatSoda.OVR` | OpenVR wrappers, overlay types (`DashboardOverlay`, `WorldSpaceOverlay`, `DeviceTrackedOverlay`), `VREventDispatcher`, and exception types |
| `src/FloatSoda` | Framework core: RenderObject tree, Widget/Element system, `RenderPipeline`, `FloatSodaApp` |
| `src/FloatSoda.Testing` | Headless Widget and RenderObject bitmap renderers for tests and tooling |
| `src/FloatSoda.UI` | Headless UI layer: behavior-only widgets (`ButtonBase`, `InteractionState`), no visuals — see `docs/UILayering.md`. **Planned for Phase 5, not yet usable** |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sumx21t-3310/FloatSoda](https://github.com/sumx21t-3310/FloatSoda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
