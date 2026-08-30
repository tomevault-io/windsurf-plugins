---
trigger: always_on
description: EncDotNet.S100 is a set of .NET libraries and a cross-platform desktop viewer for reading, portraying, and rendering [IHO S-100](https://iho.int/en/s-100-edition-5-2-0) nautical chart data. The solution targets .NET 10 with `<Nullable>enable</Nullable>` and `<ImplicitUsings>enable</ImplicitUsings>` throughout.
---

# Copilot Instructions for EncDotNet.S100

## Project overview

EncDotNet.S100 is a set of .NET libraries and a cross-platform desktop viewer for reading, portraying, and rendering [IHO S-100](https://iho.int/en/s-100-edition-5-2-0) nautical chart data. The solution targets .NET 10 with `<Nullable>enable</Nullable>` and `<ImplicitUsings>enable</ImplicitUsings>` throughout.

## Relevant specifications

| Spec | Role in this codebase |
|---|---|
| **S-100 Ed 5.2.1** (see `docs/specs/S-100 Ed 5.2.1_FINAL.pdf`) | Overarching framework; Parts 1–10 define exchange sets, HDF5 encoding, feature catalogues, portrayal catalogues, and the Lua portrayal engine |
| **S-101** | Electronic Navigational Charts — ISO 8211 encoded vector datasets; portrayal via S-100 Part 9A Lua pipeline |
| **S-102** | Bathymetric Surfaces — HDF5 encoded depth/uncertainty grids |
| **S-104** | Water Level Information — HDF5 encoded water-level time-step grids |
| **S-111** | Surface Currents — HDF5 encoded current speed/direction grids |
| **S-122** | Marine Protected Areas — GML encoded (S-100 Part 10b), XSLT portrayal |
| **S-124** | Navigational Warnings — GML encoded (S-100 Part 10b), XSLT portrayal |
| **S-125** | Marine Aids to Navigation — GML encoded (S-100 Part 10b), XSLT portrayal |
| **S-127** | Marine Resources and Services — GML encoded (S-100 Part 10b), XSLT portrayal |
| **S-128** | Catalogue of Nautical Products — GML encoded (S-100 Part 10b), XSLT portrayal |
| **S-129** | Under Keel Clearance Management — GML encoded (S-100 Part 10b) |
| **S-131** | Marine Harbour Infrastructure — GML encoded (S-100 Part 10b), Lua portrayal (Part 9A) |
| **S-201** | Aids to Navigation Information (IALA, authority-to-authority exchange) — GML encoded (S-100 Part 10b), XSLT portrayal |
| **S-411** | Sea Ice Information — GML encoded (S-100 Part 10b), XSLT portrayal |
| **S-421** | Route Plans (IEC 63173-2) — GML encoded (S-100 Part 10b), XSLT portrayal |
| **ISO 8211** | Record format used by S-101 datasets; read via `EncDotNet.Iso8211` NuGet package |
| **ISO 19110** | Feature Catalogue schema; parsed by `EncDotNet.S100.Features` |
| **HDF5** | Binary container used by S-102, S-104, S-111; accessed via the `IHdf5File`/`IHdf5Group` abstraction backed by PureHDF |

## Repository layout

```
src/
  EncDotNet.S100.Core/               # Core abstractions (IAssetSource, IHdf5File, ILuaEngine, pipelines, shared types)
  EncDotNet.S100.Features/           # Feature Catalogue XML parser (ISO 19110 / S-100 Part 5)
  EncDotNet.S100.ExchangeSets/       # Exchange Set CATALOG.XML parser
  EncDotNet.S100.Portrayals/         # Portrayal Catalogue XML parser (S-100 Part 9)
  EncDotNet.S100.Specifications/     # Bundles official FCs and PCs as embedded resources
  EncDotNet.S100.Hdf5.PureHdf/       # IHdf5File implementation using PureHDF (no native deps)
  EncDotNet.S100.Scripting.MoonSharp/ # ILuaEngine implementation using MoonSharp (Lua 5.2)
  EncDotNet.S100.Datasets.S101/      # S-101 ENC reader + Lua portrayal pipeline
  EncDotNet.S100.Datasets.S102/      # S-102 bathymetry reader + coverage pipeline
  EncDotNet.S100.Datasets.S104/      # S-104 water level reader + coverage pipeline
  EncDotNet.S100.Datasets.S111/      # S-111 surface currents reader + coverage pipeline
  EncDotNet.S100.Datasets.S122/      # S-122 marine protected areas reader + GML/XSLT portrayal
  EncDotNet.S100.Datasets.S124/      # S-124 navigational warnings reader + GML/XSLT portrayal
  EncDotNet.S100.Datasets.S125/      # S-125 marine aids to navigation reader + GML/XSLT portrayal
  EncDotNet.S100.Datasets.S127/      # S-127 marine resources & services reader + GML/XSLT portrayal
  EncDotNet.S100.Datasets.S128/      # S-128 catalogue of nautical products reader + GML/XSLT portrayal
  EncDotNet.S100.Datasets.S129/      # S-129 under keel clearance reader
  EncDotNet.S100.Datasets.S131/      # S-131 marine harbour infrastructure reader + GML/Lua portrayal
  EncDotNet.S100.Datasets.S201/      # S-201 aids to navigation information (IALA) reader + GML/XSLT portrayal
  EncDotNet.S100.Datasets.S411/      # S-411 sea ice reader + GML/XSLT portrayal
  EncDotNet.S100.Datasets.S421/      # S-421 route plans reader + GML/XSLT portrayal
  EncDotNet.S100.Renderers.Skia/     # SkiaSharp coverage + vector rasteriser (no map projection)
  EncDotNet.S100.Renderers.Mapsui/   # Mapsui layer renderer with CRS projection (ProjNet/EPSG:3857)
  EncDotNet.S100.Viewer/             # Avalonia cross-platform desktop viewer (macOS, Windows, Linux)
tests/
  EncDotNet.S100.Datasets.S104.Tests/
  EncDotNet.S100.Datasets.S111.Tests/
  EncDotNet.S100.Datasets.S124.Tests/
  EncDotNet.S100.Datasets.S125.Tests/
  EncDotNet.S100.Datasets.S131.Tests/
  EncDotNet.S100.Datasets.S201.Tests/
  EncDotNet.S100.Datasets.S421.Tests/
  EncDotNet.S100.ExchangeSets.Tests/
  EncDotNet.S100.Pipelines.Tests/    # Pipeline integration tests (S-101, S-102, coverage, vector, Skia)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philliphoff/EncDotNet.S100](https://github.com/philliphoff/EncDotNet.S100) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
