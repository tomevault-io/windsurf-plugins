---
trigger: always_on
description: GeoTessera is a Python library for accessing and working with Tessera geospatial foundation model embeddings. The library provides tools to download, process, and visualize satellite imagery embeddings from Sentinel-1 and Sentinel-2 data at 10m resolution.
---

# GitHub Copilot Instructions for GeoTessera

## Project Overview

GeoTessera is a Python library for accessing and working with Tessera geospatial foundation model embeddings. The library provides tools to download, process, and visualize satellite imagery embeddings from Sentinel-1 and Sentinel-2 data at 10m resolution.

### Core Architecture

- **Two-step workflow**: Retrieve embeddings (numpy arrays) → Export to desired format (GeoTIFF/NPY)
- **Registry system**: Parquet-based metadata registry for efficient tile lookup
- **0.1-degree grid**: Tiles cover ~11km × 11km, named by center coordinates
- **Direct HTTP downloads**: On-demand tile fetching with automatic cleanup
- **Hash verification**: SHA256 checksums ensure data integrity by default

## Technology Stack

### Core Dependencies

- **Python**: 3.12 or 3.13 required (3.12+ due to geozarr-toolkit)
- **CLI Framework**: `typer` with `rich` for interactive output
- **Geospatial**: `rasterio`, `geopandas`, `rioxarray` for GIS operations
- **Data Processing**: `numpy`, `pandas`, `pyarrow` (for Parquet registry)
- **Visualization**: `matplotlib`, `scikit-learn` (PCA), `scikit-image`
- **Storage**: `zarr`, `xarray`, `dask` for chunked data handling

### Build System

- **Package Manager**: Uses `uv` for dependency management (preferred) or `pip`
- **Configuration**: `pyproject.toml` with setuptools backend
- **Lock File**: `uv.lock` for reproducible builds
- **Test Framework**: `cram` (shell-based functional testing)
- **Linting**: `ruff` for code quality

## Coding Standards

### Python Style

- Follow PEP 8 conventions
- Use type hints for function signatures (e.g., `Optional[str]`, `Path`, etc.)
- Use `typing_extensions.Annotated` for CLI argument annotations with `typer`
- Prefer pathlib `Path` over string paths
- Use f-strings for string formatting

### Code Organization

```
geotessera/
├── __init__.py          # Package exports and version
├── core.py              # Main GeoTessera class
├── registry.py          # Parquet registry management
├── cli.py               # Main CLI commands
├── registry_cli.py      # Registry-specific CLI
├── store.py             # Zarr v3 store access (GeoTesseraZarr)
├── tile_transform.py    # Coordinate transforms for Zarr stores
├── tiles.py             # Tile abstraction (GeoTIFF + NPY)
├── visualization.py     # Visualization functions
├── web.py              # Web map generation
├── country.py          # Country bounding box utilities
└── progress.py         # Progress tracking utilities
```

### Key Patterns

1. **Rich Console Output**: Use `rich.console.Console` and `rich.progress.Progress` for user-facing output
2. **Logging**: Configure with `rich.logging.RichHandler` for pretty logs
3. **Temporary Files**: Use `tempfile` for intermediate data, clean up automatically
4. **Error Handling**: Provide clear error messages with context
5. **CLI Structure**: Commands are typer apps with descriptive help text

## Testing Guidelines

### Test Framework: Cram

Tests are written in `.t` files using cram (shell-based testing):

```bash
# Example test structure
Setup environment:
  $ export TERM=dumb
  $ export TESTDIR="$CRAMTMP/test_outputs"

Run command and check output:
  $ geotessera version
  [version number]
```

### Test Structure

- `tests/cli.t` - CLI command tests
- `tests/hash.t` - Hash verification tests
- `tests/viz.t` - Visualization tests

### Running Tests

```bash
# Run all tests
env TERM=dumb TTY_INTERACTIVE=0 uv run cram tests -v

# Run specific test file
env TERM=dumb TTY_INTERACTIVE=0 uv run cram tests/cli.t -v
```

### Testing Best Practices

- Set `TERM=dumb` to disable ANSI output in tests
- Use `$CRAMTMP` for temporary test data
- Override `XDG_CACHE_HOME` for isolated caching
- Check command exit codes and output patterns
- Test both success and error cases

## Build and Development Workflow

### Initial Setup

```bash
# Clone repository
git clone https://github.com/ucam-eo/geotessera
cd geotessera

# Install with uv (preferred)
uv sync --locked --all-extras --dev

# Or with pip
pip install -e .
```

### Development Commands

```bash
# Run tests
env TERM=dumb TTY_INTERACTIVE=0 uv run cram tests -v

# Run CLI locally
uv run -m geotessera.cli --help
python -m geotessera.cli --help

# Lint code
ruff check .
ruff format .
```

### CI/CD

- GitHub Actions workflow: `.github/workflows/ci.yml`
- Multi-platform testing: Ubuntu, macOS (Intel & Apple Silicon)
- Python versions: 3.12, 3.13
- Dependencies: GDAL must be installed before Python packages
- Tests run with `uv run cram tests -v`

## Key Concepts to Remember

### Coordinate System

- Tiles use WGS84 coordinates (longitude, latitude)
- Tile naming: `grid_{lon}_{lat}` (e.g., `grid_0.15_52.05`)
- Bounding box format: `(min_lon, min_lat, max_lon, max_lat)`
- GeoTIFF exports use UTM projection from landmask tiles

### Data Files

1. **Embeddings**: `grid_0.15_52.05.npy` - int8 quantized arrays (H×W×128)
2. **Scales**: `grid_0.15_52.05_scales.npy` - float32 scale factors
3. **Landmasks**: `grid_0.15_52.05.tiff` - UTM projection + land/water masks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ucam-eo/geotessera](https://github.com/ucam-eo/geotessera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
