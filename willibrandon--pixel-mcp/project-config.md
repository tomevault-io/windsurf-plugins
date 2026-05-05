---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An MCP (Model Context Protocol) server implementation in Go that exposes Aseprite's pixel art and animation capabilities through programmatic access. The server integrates with Aseprite's command-line interface and Lua scripting API to enable AI assistants to create and manipulate sprites.

## Build Commands

```bash
# Build the server binary
make build
# or: go build -o bin/pixel-mcp ./cmd/pixel-mcp

# Run tests
make test
# or: go test -v -race -cover ./...

# Run integration tests (requires real Aseprite)
go test -tags=integration ./...

# Generate test coverage report
make test-coverage

# Run linters
make lint

# Clean build artifacts
make clean

# Install binary
make install
```

## Configuration Requirements

**CRITICAL**: This project requires explicit configuration - no automatic discovery or environment variables.

All operations require a configuration file at `~/.config/pixel-mcp/config.json`:
```json
{
  "aseprite_path": "/absolute/path/to/aseprite",
  "temp_dir": "/tmp/pixel-mcp",
  "timeout": 30,
  "log_level": "info",
  "log_file": "",
  "enable_timing": false
}
```

Configuration fields:
- `aseprite_path` (required): Absolute path to Aseprite executable
- `temp_dir` (optional): Temporary directory, defaults to `/tmp/pixel-mcp`
- `timeout` (optional): Command timeout in seconds, defaults to 30
- `log_level` (optional): Verbosity level (`debug`, `info`, `warn`, `error`), defaults to `info`
- `log_file` (optional): Path to persistent log file, empty means stderr only
- `enable_timing` (optional): Enable request tracking/timing, defaults to `false`

The `aseprite_path` MUST be an absolute path to a real Aseprite executable. The project does NOT:
- Auto-discover Aseprite installations
- Use environment variables for Aseprite path
- Search system PATH for Aseprite

## Testing Philosophy

**ALL tests use real Aseprite** - no mocks, stubs, or fakes for Aseprite integration.

- Unit tests require config file with real Aseprite path
- Integration tests (tagged with `//go:build integration`) also use real Aseprite
- Test utilities are in `internal/testutil/`
- Use `testutil.LoadTestConfig(t)` to load test configuration
- See `docs/TESTING.md` for detailed testing requirements

## Architecture

### High-Level Structure

The server acts as a bridge between MCP clients (AI assistants) and Aseprite:

```
MCP Client → MCP Server (Go) → Lua Script Generation → Aseprite CLI (--batch --script)
```

### Package Organization

- `cmd/pixel-mcp/` - Server entry point and initialization
- `pkg/aseprite/` - Core Aseprite integration
  - `client.go` - Command execution and Lua script execution
  - `lua.go` - Lua script generation utilities (includes palette and dithering generators)
  - `types.go` - Domain types (Color, Point, Rectangle, Pixel, etc.)
  - `palette.go` - K-means palette extraction and color analysis
  - `image_analysis.go` - Brightness maps, Sobel edge detection, composition analysis
- `pkg/config/` - Configuration management (file-based only)
- `pkg/server/` - MCP server implementation
- `pkg/tools/` - MCP tool implementations organized by category:
  - `canvas.go` - Sprite/layer/frame management (create_sprite, add_layer, add_frame, delete_layer with protection, delete_frame with protection, flatten_layers)
  - `drawing.go` - Drawing primitives (pixels, lines, rectangles, circles, fill, contours for polylines/polygons)
  - `selection.go` - Selection and clipboard operations (8 tools)
  - `animation.go` - Animation and timeline operations (frame duration, tags, tag deletion, duplication, linked cels)
  - `inspection.go` - Pixel data inspection and reading
  - `analysis.go` - Reference image analysis (palette extraction, edge detection, composition)
  - `dithering.go` - Dithering patterns for gradients and textures (16 patterns including Floyd-Steinberg)
  - `quantization.go` - Color quantization (median_cut, k-means, octree algorithms)
  - `auto_shading.go` - Automatic geometry-based shading with per-pixel normal calculations
  - `palette_tools.go` - Palette management (set_palette, apply_shading, analyze_palette_harmonies)
  - `transform.go` - Transform operations (flip, rotate, scale, crop, resize canvas, outline, downsampling)
  - `export.go` - Export and import operations (export_sprite, export_spritesheet, import_image, save_as)
- `internal/testutil/` - Testing utilities (no mocks)

### Core Workflow

1. MCP client sends tool request with parameters
2. Server validates parameters via JSON schema
3. Appropriate Lua script is generated from template
4. Script written to temp file with restricted permissions (0600)
5. Aseprite executed in batch mode: `aseprite --batch [sprite] --script [temp.lua]`
6. Output parsed and returned to client
7. Temp files cleaned up (always, even on error)

### Key Design Constraints

- **Stateless**: Each operation is independent, no shared state between requests
- **Batch Mode Only**: All Aseprite operations run in headless `--batch` mode
- **Lua-based**: Operations use Aseprite's Lua API, not GUI automation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willibrandon/pixel-mcp](https://github.com/willibrandon/pixel-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
