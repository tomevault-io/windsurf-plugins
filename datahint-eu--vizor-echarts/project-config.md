---
trigger: always_on
description: - **Purpose**: Blazor wrapper around Apache ECharts; main library targets .NET 10.
---

# Copilot Instructions for vizor-echarts

## Project Overview
- **Purpose**: Blazor wrapper around Apache ECharts; main library targets .NET 10.
- **Structure**: src/Vizor.ECharts (core), src/Vizor.ECharts.Demo (Blazor server demo), src/Vizor.ECharts.Samples (area-based samples), src/Vizor.ECharts.Tests (unit tests via MSTest), src/Vizor.ECharts.BindingGenerator (code generation utility).
- **Documentation**: Detailed technical documentation is in the [doc/](doc/) folder, covering generator architecture, testing strategy, circular dependency solutions, memory profiling, and implementation guides.

## External References
Official Apache ECharts documentation and resources:
- **Examples Gallery**: https://echarts.apache.org/examples/en/index.html - Official ECharts examples (translate JavaScript to C#)
- **Cheat Sheet**: https://echarts.apache.org/en/cheat-sheet.html - Quick reference for common patterns
- **Option Documentation**: https://echarts.apache.org/en/option.html - Complete API reference for chart options
- **Tutorial**: https://echarts.apache.org/en/tutorial.html - In-depth guides (e.g., Dataset usage)
- **Online Editor**: https://echarts.apache.org/examples/en/editor.html - Test ECharts code snippets before translating to C#

## Core Architecture & Interop Flow

### C# → JS Pipeline
[src/Vizor.ECharts/EChart.razor](src/Vizor.ECharts/EChart.razor) inherits from [src/Vizor.ECharts/EChartBase.cs](src/Vizor.ECharts/EChartBase.cs) and serializes `ChartOptions` to JSON, passing it via `IJSRuntime` interop to [src/Vizor.ECharts/Scripts/vizor-echarts.js](src/Vizor.ECharts/Scripts/vizor-echarts.js). JS functions called: `vizorECharts.initChart` (initial render), `updateChart` (options update), `attachClickEvent` (click handling).

### Chart Options Model
- `ChartOptions` is a partial shell ([src/Vizor.ECharts/ChartOptions.cs](src/Vizor.ECharts/ChartOptions.cs)).
- Actual option/series types in [src/Vizor.ECharts/Options](src/Vizor.ECharts/Options) and [src/Vizor.ECharts/Series](src/Vizor.ECharts/Series).
- **Most files auto-generated** from ECharts option.json; preserve `[JsonPropertyName]` attributes and property naming conventions.

### Series Typing & Polymorphic Serialization
- Each series implements `ISeries` interface (auto-generated in [src/Vizor.ECharts/Series/Generated/ISeries.cs](src/Vizor.ECharts/Series/Generated/ISeries.cs)).
- Uses **[.NET 10 polymorphic serialization](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/polymorphism)**: `[JsonPolymorphic(TypeDiscriminatorPropertyName = "type")]` on ISeries interface, with `[JsonDerivedType]` attributes for each series type (e.g., `LineSeries`, `BarSeries`).
- **No explicit Type property** in series classes—discriminator added automatically by serializer during JSON output.
- Same pattern applies to `IDataZoom` ([src/Vizor.ECharts/Options/DataZoom/Generated/IDataZoom.cs](src/Vizor.ECharts/Options/DataZoom/Generated/IDataZoom.cs)) with `InsideDataZoom` and `SliderDataZoom`.
- Polymorphic series data handled by [src/Vizor.ECharts/Types/SeriesDataConverterFactory.cs](src/Vizor.ECharts/Types/SeriesDataConverterFactory.cs).
- `SeriesData<T>`, `SeriesData<T,U>`, and `SeriesData<T,U,V>` generics serialize correctly via factory pattern.

## Data Loading Patterns

### Server-side (C#)
Use the `DataLoader` callback parameter on `EChart` component—invoked during render to populate options asynchronously.

### Browser-side (JS fetch)
Use `ExternalDataSource` ([src/Vizor.ECharts/Types/ExternalDataSource.cs](src/Vizor.ECharts/Types/ExternalDataSource.cs)) to define fetch URLs, optional path extraction, and `afterLoad` JS functions. **Inside options objects, reference via `ExternalDataSourceRef`** (not `ExternalDataSource`). Cached fetches accessible via `window.vizorECharts.getDataSource(fetchId)`.

### JS Functions in Options
Wrap raw JavaScript as `JavascriptFunction` ([src/Vizor.ECharts/Types/JavascriptFunction.cs](src/Vizor.ECharts/Types/JavascriptFunction.cs)); converter outputs raw function text for JS `eval`.

## Multi-Chart Coordination
Use `ChartGroup` ([src/Vizor.ECharts/ChartGroup.cs](src/Vizor.ECharts/ChartGroup.cs)) to synchronize updates across multiple charts via `UpdateAsync`.

## JSON Serialization & Performance
- **Cached serializer options by default** (`CacheJsonSerializerOptions=true`); reuse across all charts for memory efficiency (see Meziantou post linked in EChartBase comments).
- camelCase JSON convention via shared serializer.
- Custom `JsonConverters` parameter: use consistently or disable caching.

## Diagnostic Report & Pattern Analysis

The generator produces **TypePatternAnalysisReport.md** showing type mapping coverage:

### How to Access
- **Location**: `src/Vizor.ECharts.BindingGenerator/` (same directory as `typeinfo.txt`)
- **Triggered**: Automatically generated when running either:
  - `dotnet run -- option-binding --input <path/to/option.json> --output src/Vizor.ECharts`
  - `dotnet run -- typeinfo --input <path/to/option.json>`

### Report Contents
```
# Type Pattern Analysis Report
Generated: 2026-01-04

## Summary
- Total properties analyzed: 37,348
- ✅ Fully supported: 37,330 (99.95%)
- ⚠️ Partially supported: 2 (0.01%)
- ❌ Unsupported: 16 (0.04%)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datahint-eu/vizor-echarts](https://github.com/datahint-eu/vizor-echarts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
