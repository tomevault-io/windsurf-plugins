---
trigger: always_on
description: A5 is a TypeScript library that partitions the world into equal-area pentagonal cells at 31 resolution levels. Built on a dodecahedral geometry, it provides millimeter-accurate geospatial indexing (30mm² at highest resolution) encoded as 64-bit integers.
---

# A5 - Global Pentagonal Geospatial Index

## Overview
A5 is a TypeScript library that partitions the world into equal-area pentagonal cells at 31 resolution levels. Built on a dodecahedral geometry, it provides millimeter-accurate geospatial indexing (30mm² at highest resolution) encoded as 64-bit integers.

Website: https://a5geo.org
Docs: docs/api-reference/README.md

## Ports

- A5 is ported to Python and Rust, with each language being treated as an equally valid port.
- The Typescript project contains the docs and website
- The projects are typically checked out in parallel and can be accessed via:
  - `../a5` TypeScript version
  - `../a5-py` Python version
  - `../a5-rs` Rust version
- Each of the ports has its own `CLAUDE.md` file. Whenever you work with another project, read this file to get the additional context.


## Typescript Project Structure
- `/modules` - TypeScript source code (NOT `/src`)
  - `/core` - Core geospatial functionality (cell, hex, hilbert, serialization, etc.)
  - `/geometry` - Geometric calculations (pentagon, spherical_triangle, spherical_polygon)
  - `/projections` - Map projection implementations (dodecahedron, authalic, gnomonic, etc.)
- `/dist` - Built outputs (a5.js, a5.cjs, a5.d.ts)
- `/tests` - Vitest test files
- `/examples/cli` - CLI applications demonstrating A5 usage
- `/examples/website` - Website example applications (source code)
- `/website` - Docusaurus documentation site
  - `/website/static/data` - Static data files for website examples (JSON, Parquet, etc.)
  - `/website/src/examples` - Example page definitions that import from `/examples/website`
- `/scripts` - Build and utility scripts

## Key Concepts
- **Cell**: A pentagonal region at a specific resolution (represented as bigint)
- **Resolution**: 0-30, where 0 is global coverage and 30 is ~30mm²
- **Compaction**: Combining child cells into parent cells for efficient storage
- **Cell ID**: Always a bigint (use `u64ToHex()` for string representation)

## Commands
```bash
yarn build             # Build library
yarn generate-fixtures # Generate fixtures
yarn test              # Run tests (with watch mode)
yarn test --run        # Run tests once
yarn test hex          # Run tests just for a given file, here `hex.text.ts`
```

## Development Guidelines
- Read CONTRIBUTING.md
- **TypeScript**: Source files in `/modules`, compiled to `/dist`
- **Tests**: Use Vitest, run specific tests with `yarn test <filename> --run`
- **Imports**: Run `python3 analyze_imports.py modules --check-only` after changing imports
- **Cell IDs**: Always use bigint internally, convert to hex with `u64ToHex()` / `hexToU64()`
- **Branded Types**: LonLat and other coordinate types are branded - cast with `as LonLat`, not type construction
- **Build**: Run `yarn build` before testing example CLIs
- **Documentation**: When adding new documentation pages, update `docs/table-of-contents.json` to add them to the sidebar navigation
- **Hot loops**: Prefer indexed `for` loops over `.map()` / `.forEach()` / `.filter()` in performance-sensitive paths (cell-fill, line tracing, dense-sample loops, fixture generators). Two reasons: (1) the closure form is measurably slower in V8 for large inputs (10–15% on country-scale polygons in the polygonToCells PR), and (2) `for (let i = 0; i < n; i++)` ports 1:1 to `for i in range(n)` / `for i in 0..n`, while `.map(fn)` does not.

## Website Examples
- Website example data files go in `/website/static/data/` (NOT in `/examples/website/*/public/data/`)
- When regenerating data for examples, compare with deployed version at https://a5geo.org/examples/* to verify resolution
- Use `getResolution()` to check the resolution of cells in existing data files
- Example resolution levels:
  - Road safety example: resolution 13 (~50k cells for UK dataset)

## Using Chrome DevTools MCP for Testing

The website development server is ALWAYS running on `localhost:3000`. Use Chrome DevTools MCP to test changes:

**Screenshot Management:**
- Name all screenshots with prefix `claude-screenshot-` followed by a descriptive suffix
- Save screenshots to `website/` directory (relative to project root)
- Clean up old `claude-screenshot-*.png` files from previous tasks before creating new ones
- Keep only screenshots relevant to the current task for debugging

**Testing guidelines:**
- NEVER attempt to start the dev server with `yarn start` or similar commands
- Assume the user has the website running at `localhost:3000`
- Reload the page after making changes to see updates
- Take snapshots for DOM inspection when needed
- Use screenshots primarily for visual verification and debugging
- Clean up screenshots when done with a task

## Polyglot Mirroring
A5 uses **Polyglot Mirroring** - maintaining functionally equivalent implementations across TypeScript, Python, and Rust. See [docs/ecosystem/polyglot-mirroring.md](docs/ecosystem/polyglot-mirroring.md) for details.

When porting features between languages:
1. **Any language can be the source** - changes can originate in TypeScript, Python, or Rust

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felixpalmer/a5](https://github.com/felixpalmer/a5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
