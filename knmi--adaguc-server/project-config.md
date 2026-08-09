---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

Adaguc-server is a C++ WMS/WCS geographical information server for meteorological,
climatological and remote sensing data. Core server code lives in `adagucserverEC/`,
the low-level data model in `CCDFDataModel/`, and shared helper classes in `hclasses/`.

## `adagucserverEC/` — server core

- `adagucserver.cpp` — `main()` entry point.
- `CRequest.cpp/.h` — central request handler (`process_querystring()` dispatches to
  per-operation methods for WMS GetMap/GetCapabilities/GetFeatureInfo/GetLegendGraphic
  and WCS GetCapabilities/DescribeCoverage/GetCoverage).
- `CServerParams.cpp/.h` — parsed request/query params plus server config;
  `CServerConfig_CPPXSD.h` holds the generated classes for `adaguc.dataset.xml`.
- `CDataSource.cpp/.h` — one configured layer/dataset instance to render (style,
  projection, dimensions), built by `CRequest`.
- `CDataReader.cpp/.h` — opens/reads the data for a `CDataSource` via `CCDFDataModel`.
- DB/catalog layer: `CDBAdapterPostgreSQL.cpp/.h` (concrete, no abstract interface)
  wraps `CPGSQLDB.cpp/.h` (libpq); accessed as a singleton through
  `CDBFactory::getDBAdapter()`. `CDBFileScanner.cpp/.h` scans data directories and
  populates the DB. `CDBStore.cpp/.h` holds query result rows/columns.
- `CConvert*` classes translate source-specific formats into the internal CDM model:
  `CConvertADAGUCPoint`/`CConvertADAGUCVector` (ADAGUC point/vector formats),
  `CConvertASCAT` (ASCAT scatterometer wind), `CConvertCurvilinear` (curvilinear
  grids), `CConvertEProfile` (E-PROFILE ceilometer/lidar), `CConvertGeoJSON`,
  `CConvertH5VolScan(Utils)` (HDF5 ODIM radar volume scans), `CConvertHexagon`,
  `CConvertKNMIH5EchoToppen` (KNMI radar echo-top), `CConvertLatLonBnds`/
  `CConvertLatLonGrid` (each split into Header/Data files), `CConvertTROPOMI`
  (Sentinel-5P), `CConvertUGRIDMesh` (unstructured mesh).
- Warping/rendering pipeline: `CImageWarper.cpp/.h` (PROJ-based reprojection),
  `CGenericDataWarper.h` + `GenericDataWarper/` (generic per-pixel warp/draw
  dispatch), `CImgWarpGeneric/` and siblings (`CImgWarpNearestNeighbour`,
  `CImgWarpNearestRGBA`, `CImgWarpBilinear`, `CImgWarpHillShaded`) implement concrete
  warpers, `CDrawImage.cpp/.h` is the canvas, `CCairoPlotter.cpp/.h` is the
  Cairo-based plotting backend, and `CImageDataWriter`/`CGDALDataWriter`/
  `CNetCDFDataWriter` encode output (PNG/GDAL/NetCDF).

Subdirectories:

- `CDataPostProcessors/` — pluggable post-processing steps on read data (unit
  conversion, clipping, Beaufort, UV components, feature filtering, etc.) via
  `CDPPInterface.h` — see "Adding a new data post processor" below.
- `Types/` — shared value types (`GeoParameters`, `ProjectionStore`,
  `LayerMetadataType.h`).
- `CImageOperators/` — raster ops (`drawContour`, `smoothRasterField`).
- `CImgRenderers/` — point/vector style rendering (`getPointStyle`, `getVectorStyle`).
- `CUniqueRequests/` — dedupes/coordinates identical concurrent requests.
- `CLegendRenderers/` — legend image generation (continuous/discrete variants).
- `utils/` — query string parsing, config utils, layer metadata, geometry utils, logging.
- `LayerTypeLiveUpdate/` — live-updating (auto-refreshing) layer type support.

## `CCDFDataModel/` — in-memory data model

An in-memory, CF-NetCDF-like generic data model that represents variables,
dimensions and attributes uniformly regardless of source format.

- `CCDFObject.cpp/.h` — top-level container (like a NetCDF dataset): variables,
  dimensions, global attributes.
- `CCDFVariable.cpp/.h` — a variable (data plus its own attributes/dimensions).
- `CCDFAttribute.cpp/.h` — key/value metadata attribute.
- `CCDFDimension.h` — dimension definition.
- `CCDFStore.cpp/.h` — storage/collection management for objects.
- `CCDFReader.h` — abstract `CDFReader` interface (`open()`, `close()`,
  `_readVariableData()`). Format-specific readers implement it as a strategy so
  `CDataReader` (in `adagucserverEC/`) can read any backing format into the same
  `CCDFObject` model: `CCDFNetCDFIO` (NetCDF), `CCDFHDF5IO`/`CCDFHDF5IO_ODIM.cpp`
  (HDF5, incl. ODIM radar conventions), `CCDFPNGIO`, `CCDFGeoJSONIO`, `CCDFCSVReader`.
- `traceTimings/` — lightweight tracing utility. Public API:
  `traceTimingsEnableAndInit()`, `traceTimingsSpanStart(TraceTimingType)`,
  `traceTimingsSpanEnd(TraceTimingType)`, `traceTimingsCheckEnabled()`.

## `hclasses/` — shared low-level utilities

Project-agnostic utility classes used by both `adagucserverEC/` and
`CCDFDataModel/`.

- `CTString.cpp/.h` — `CT::string`, the custom string class used pervasively
  throughout the codebase. Prefer `std::string` in new/changed code unless you
  specifically need `CT::string`-only API (e.g. `concat`, `printconcat`,
  `equalsIgnoreCase`, `startsWith`, `split`, `substring`) — the two convert
  implicitly, but `CT::string`-only methods don't exist on `std::string`.
- `CDebugger.h` — `CDBDebug`/`CDBWarning`/`CDBError`, printf-style variadic
  logging macros; `%s` arguments need `.c_str()` (passing `std::string` directly
  is undefined behaviour). Don't add `.c_str()` where it's not needed elsewhere
  (e.g. `nlohmann::json::parse()` accepts a `std::string` directly).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KNMI/adaguc-server](https://github.com/KNMI/adaguc-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
