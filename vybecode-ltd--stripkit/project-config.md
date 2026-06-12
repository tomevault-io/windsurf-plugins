---
trigger: always_on
description: > Version 1.0.0 · last-updated 2026-06-06 · last-audit 2026-06-06
---

# CLAUDE.md — StripKit

> Version 1.0.0 · last-updated 2026-06-06 · last-audit 2026-06-06

Context for any Claude Code / agent session working on this repo. Keep this file
short, current, and instruction-shaped. Update the **Last completed task** section
at the end of every working session.

## What this is

StripKit is a C#/Avalonia desktop tool that turns a single transparent PNG
into an animated **filmstrip** (sprite sheet) for audio-plugin GUI controls:
rotary knobs, vertical faders, and horizontal sliders. The output PNG is consumed
by a JUCE-style LookAndFeel filmstrip loader (one frame shown per parameter value).
It is the asset-production companion to the GUI skinning system / VybeForge.

## Start here

1. Read `docs/SOURCE_MAP.md` (where everything lives) and this file.
2. `docs/ARCHITECTURE.md` is the deep design reference; `docs/HANDOFF.md` is the
   current state + next task; `docs/ROADMAP.md` is the full phased plan;
   `docs/KICKOFF.md` is the paste-in prompt for a new session.
3. Other managed docs: `docs/TESTING.md`, `docs/CHANGELOG.md`, `docs/BUGS.md`,
   `docs/AUDIT-LOG.md`. Packaging + release flow: `docs/PACKAGING.md`.
4. The skills in `.claude/skills/` are scoped to this repo — use them.

## Stack

- .NET 9, Avalonia 11.3, CommunityToolkit.Mvvm 8.4 (source generators),
  SkiaSharp 3.119.2.
- Layered-source import (app-only): **Svg.Skia** 5.0.0 (MIT, SVG layers) + **Magick.NET-Q8-x64**
  14.13.1 (Apache-2.0, PSD/PSB layers). Both permissive; not in `FilmstripEngine.cs`.
- AI SVG generation (Generate tab, app-only): OpenAI / Gemini / Claude via their text APIs over a
  shared `HttpClient`; user API keys encrypted at rest with **System.Security.Cryptography.ProtectedData**
  9.0.0 (Windows DPAPI). Not in `FilmstripEngine.cs`.
- MVVM + DI (Microsoft.Extensions.DependencyInjection), compiled bindings.
- Tests: xUnit + NSubstitute + FluentAssertions, `Avalonia.Headless` for view
  tests, golden-image regression for the renderer (`tests/StripKit.Tests`).
- Packaging: self-contained `win-x64` publish → **Inno Setup** installer
  (`installer/StripKit.iss`); distributed as a **GitHub Release download** (no in-app
  auto-update). Release pipeline: `scripts/Invoke-Release.ps1` +
  `.github/workflows/auto-release.yml` — see `docs/PACKAGING.md`.
- CI: `.github/workflows/ci.yml` runs build + full test suite on every push and PR.

## OSS / Contributing

- License: **MIT** (`LICENSE` at repo root).
- Public README with badges (`README.md`).
- `CONTRIBUTING.md` — contribution guide.
- `.github/ISSUE_TEMPLATE/` — `bug_report.md` + `feature_request.md`.
- `.github/pull_request_template.md` — PR checklist.

## Run / build

- `dotnet run --project src/StripKit` (needs the .NET 9 SDK).
- If NuGet warns of a SkiaSharp version conflict with Avalonia, align the
  `SkiaSharp` version in the csproj to Avalonia's transitive one
  (`dotnet list package --include-transitive`).
- Use `python -m pip` style invocation in any Python helper scripts (bare
  `pip`/`python` are not on PATH in this environment).

## Release (Inno Setup + GitHub) — full detail in `docs/PACKAGING.md`

Three stages, **one release creator**. **Stage 1** `scripts/Invoke-Release.ps1`:
test-gate → bump `Version` in `src/StripKit/StripKit.csproj`, `MyAppVersion` in
`installer/StripKit.iss`, and `docs/CHANGELOG.md` → `dotnet publish` → `ISCC` builds
`releases/latest/StripKit-Setup-<ver>-x64.exe` → commit + tag `vX.Y.Z` + push. **Stage
2** `.github/workflows/auto-release.yml` (triggered by the tracked `releases/latest/*.exe`,
or `workflow_dispatch`): VirusTotal scan (`VT_API_KEY` secret) → the **sole**
`gh release create` (notes via `--notes-file`, never inline `--notes`). **Stage 3** the
website reads the live GitHub Release. Unsigned for now (SmartScreen / VirusTotal
heuristic FPs until a code-signing cert is added).

## Architecture (one idea, four component types) — full detail in `docs/ARCHITECTURE.md`

Every render is: *for each of N frames, place the source art inside a fixed frame
cell under a per-frame transform, then stack the cells into one PNG.* The four
component types are knob, vertical fader, horizontal slider, and **meter**
(progressive segment fill). The app is a `TabControl` with **five** tabs — **Create**
(make a strip), **Import** (re-use / re-slice / resample one), **Batch** (a whole folder at
once), **Skin** (assemble a multi-control `skin.json`), and **Generate** (AI-generate a layered
knob SVG from your own OpenAI / Gemini / Claude key, then hand it to Create).

- `Models/` — pure data, no UI/Skia deps: `FilmstripSettings` (render contract),
  `FrameTransform`, `StripDetection` (importer output),
  `SkinManifest`/`ManifestControl`/`ManifestBounds`, `BatchModels`, `CodeModels`,
  `RenderLayer` (`LayerBehavior` + per-layer pivot — the layered-knob stack), the
  `ComponentType`/`StackDirection`/`MeterFillDirection` enums.
- `Services/SkiaFilmstripRenderer.cs` — **the heart.** `ComputeTransform` does the
  rotary/linear math; `RenderFrame` composites one frame with supersampling +
  Mitchell cubic resampling (meters fill segments via `RenderMeterFrame` — procedural
  or layered on/off-art reveal; knobs may get a value-tracking fill arc via

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vybecode-LTD/stripkit](https://github.com/Vybecode-LTD/stripkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
