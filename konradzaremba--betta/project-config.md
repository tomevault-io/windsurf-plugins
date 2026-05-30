---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Betta** (assembly name) / **GH_AutoCreator** (repo name) is a Grasshopper plugin that auto-generates GH components from attributed service methods — Dynamo-style ZeroTouch for Grasshopper. Developers write plain C# services decorated with `[GrasshopperMethod]` / `[GrasshopperParameter]`; the runtime reflects over them, builds inputs/outputs, handles type coercion, and publishes them as Grasshopper components with no manual `GH_Component` subclasses required.

Tagline (from the brand brief): *"Same silhouette. Every fish is its own. Runtime adaptation for Grasshopper."* Implemented as: each component gets one of **six embedded betta silhouettes** (the Mini PNGs in `media/`, embedded into `Betta.gha` via `LogicalName` resources) picked deterministically from its descriptor GUID. The six silhouettes share the same pose (fan tail left, head right) but were each painted with their own colors and gradients — the visual variety comes from the artist, not from procedural tinting.

Icon render path (in `Betta/Services/IconProvider.cs`):
1. **At first request**, all six fish are rendered once and cached in a `Lazy<Bitmap[]>` field — `PickRendered(d)` then returns `_rendered.Value[d.Guid.ToByteArray()[0] % icons.Length]`. Multiple components landing on the same fish share the same `Bitmap` instance via `descriptor.CachedIcon`. The pick is deterministic per component, stable across sessions/machines because `descriptor.Guid` is MD5 over the method signature.
2. **Per-fish rendering**: `CropToContent(source)` finds the non-transparent bounding box and crops to it (raw PNGs ship with transparent padding around the fish; cropping is what gives the final 24×24 enough pixels to look painted rather than smushed). Then `LetterboxResample` does a two-step bicubic downscale (interim ~64px on the longer edge → final 24×24, inset by a `Padding` margin) **preserving aspect ratio** — the cropped fish is centered in the tile with transparent letterbox bars, never stretched.
3. **No tinting / recoloring.** The hand-drawn source PNGs already carry rich gradients and detail; per-pixel recolor was tried and obliterated the artist's work, so it's gone. If more variety is wanted, drop more PNGs into `media/`.
4. Output is **24×24** — the Grasshopper standard. The Mini source PNGs use bold outlines and solid fills that stay legible at this size, so there's no benefit to shipping a larger bitmap and letting GH downscale. 48×48 was tried and overlapped the parameter labels in compact components; 96×96 was even worse.

Caching: each `ComponentDescriptor.CachedIcon` holds its own rendered `Bitmap` (the downsampled source, no further mutation). Plugin-supplied PNGs via `[GrasshopperMethod(IconResource = "...")]` skip the fish system entirely; they are loaded verbatim from the plugin's own assembly.

The fish PNGs live in `media/` at the repo root. The active set is the **Mini** variants (`NBTabBettaMini_*.png`) — bold, cartoon-style silhouettes with thick outlines, designed to scale cleanly to icon sizes. They're wired in via `<EmbeddedResource Include="..\media\NBTabBettaMini_*.png" LogicalName="Betta.Resources.Fish_*.png" />` in `Betta.csproj`. The legacy painted bettas (`NBTabBetta_*.png`, ~250×250 detailed line art) stay in `media/` as artwork but are not currently embedded.

`SessionFish` exposes `IReadOnlyList<Bitmap> All`, `IReadOnlyList<string> Names` (`Amber`, `Aqua`, `Cosmic`, `Forest`, `Lime`, `Rose`), and `int Count` — the bitmaps load lazily on first access via `Lazy<Bitmap[]>`. The startup log line is `Fish library loaded: 6 silhouettes (Amber, Aqua, Cosmic, Forest, Lime, Rose)`. `SessionMorph` and `BettaPalette` exist but are no longer load-bearing — parked for future surfaces (settings panel, doc badges) where flat color won't compete with the source art.

Target: **Rhino 8, .NET 7 (`net7.0-windows`), C# 11**.

End-user facing docs live in [README.md](README.md); the maintainer-facing tiered roadmap + brainstorm is [ROADMAP.md](ROADMAP.md).

## Solution layout

Three projects + one test project:

- `Betta.Abstractions/` — the **public SDK contract** (`netstandard2.0`, no dependencies). Contains only the attributes and `IBettaCollection` marker. Plugin authors reference this package with `ExcludeAssets="runtime"`; never redistribute it.
- `Betta/Betta.csproj` — the runtime `.gha` (`net7.0-windows`, `UseWpf=true`). References Abstractions.
- `Betta.Strings/Betta.Strings.csproj` — sample plugin demonstrating the terse attribute pattern; builds to a DLL copied into `%AppData%\Grasshopper\Libraries\Betta\` (the plugin folder, not the root Libraries folder).
- `TestBetta/TestBetta.csproj` — xUnit tests using `Rhino.Inside` for headless Grasshopper. Must run as x64.

Solution file is still at the (legacy) path `Betta/Betta.sln` — the solution file includes all four projects.

## NuGet packaging (Betta.Abstractions)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KonradZaremba/betta](https://github.com/KonradZaremba/betta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
