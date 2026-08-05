---
trigger: always_on
description: This document contains rules, conventions, and best practices for AI agents working on the splat-transform codebase.
---

# Agent Guidelines for splat-transform

This document contains rules, conventions, and best practices for AI agents working on the splat-transform codebase.

## Project Overview

splat-transform is a library and CLI tool for 3D Gaussian splat format conversion and transformation. It runs both in the browser (as a library) and on Node.js (as a CLI).

- **Language**: TypeScript (ES2022)
- **Module System**: ES Modules (`"type": "module"`)
- **Node Version**: >=22.0.0 (per `package.json` `engines`)
- **Build System**: Rollup
- **Testing**: Node.js built-in test runner (`node:test`)
- **Linting**: ESLint with `@playcanvas/eslint-config`
- **API Docs**: Typedoc
- **License**: MIT

## Code Style and Formatting

### Linting

- Run `npm run lint` before committing
- Only fix lint issues in code you are actively modifying
- Auto-fix available via `npm run lint:fix`

### ESLint Configuration

Base: `@playcanvas/eslint-config` with TypeScript overrides:

- **Relaxed rules**: `@typescript-eslint/ban-ts-comment`, `@typescript-eslint/no-explicit-any`, `@typescript-eslint/no-unused-vars` are off
- **JSDoc types relaxed**: `jsdoc/require-param-type`, `jsdoc/require-returns-type` off (TypeScript provides types)
- **Other relaxations**: `lines-between-class-members`, `no-await-in-loop`, `require-atomic-updates` off
- **Globals**: Both `node` and `browser` for `.ts` files; `node` only for `.mjs` files

### Naming Conventions

- Classes: PascalCase (`DataTable`, `Column`, `NodeFileSystem`)
- Functions: camelCase (`readFile`, `processDataTable`, `getInputFormat`)
- Types: PascalCase (`InputFormat`, `ReadFileOptions`, `TypedArray`)
- Constants: UPPER_SNAKE_CASE

### Import Order

1. Node built-in modules (`node:fs/promises`, `node:path`, etc.)
2. External packages (`playcanvas`)
3. Internal modules (relative paths)

### TypeScript

- Target: `es2022`
- `noImplicitAny: true`
- Generates declarations (`declaration: true`)
- Use TypeScript `type` and `interface` keywords for type definitions
- JSDoc comments are used for API documentation (Typedoc), not for type definitions

## File Organization

### Directory Structure

```
src/
├── lib/                    # Platform-agnostic library (browser + Node)
│   ├── index.ts            # Public API exports
│   ├── read.ts             # High-level read orchestration (readFile → ChunkSource[])
│   ├── write.ts            # High-level write orchestration (writeSource / compat writeFile)
│   ├── process-source.ts   # processSource / processSourceBridged (actions over a source)
│   ├── process.ts          # processDataTable and the shared action types (compat)
│   ├── source-info.ts      # --info/--stats text/JSON formatting over source metadata
│   ├── stats.ts            # computeStats over a source or table
│   ├── types.ts            # Options, Param types
│   ├── chunk/              # Core data model: ChunkSource contract, layer layouts, pooled buffers
│   ├── ops/                # Source combinators (bake-transform, concat, filter, select-lod, stack-lods, permute, morton, stats)
│   ├── decimate/           # Chunk-native decimation (partition → knn → priority → select → merge-stream)
│   ├── compat/             # DataTable ↔ ChunkSource bridges
│   ├── data-table/         # Legacy whole-scene data model (compat)
│   │   ├── data-table.ts   # DataTable and Column classes
│   │   ├── combine.ts      # Merge multiple DataTables
│   │   ├── transform.ts    # Geometric transforms
│   │   └── morton-order.ts  # Morton code sorting (legacy copy)
│   ├── io/
│   │   ├── read/           # Read abstractions (FileSystem, streams)
│   │   └── write/          # Write abstractions (FileSystem, helpers)
│   ├── readers/            # Format-specific readers (one per file)
│   │   ├── read-ply.ts
│   │   ├── read-sog.ts     # (read-sog-v1.ts handles the legacy SOG layout)
│   │   ├── read-splat.ts
│   │   ├── read-ksplat.ts
│   │   ├── read-spz.ts
│   │   ├── read-lcc.ts
│   │   ├── read-lcc2.ts
│   │   └── read-mjs.ts
│   ├── writers/            # Format-specific writers (one per file)
│   │   ├── write-ply-streaming.ts  # chunk-native PLY (the streaming hot path)
│   │   ├── write-ply.ts
│   │   ├── write-compressed-ply.ts
│   │   ├── write-sog.ts
│   │   ├── write-csv.ts
│   │   ├── write-html.ts
│   │   ├── write-lod.ts
│   │   ├── write-spz.ts
│   │   ├── write-glb.ts
│   │   ├── write-image.ts
│   │   └── write-voxel.ts
│   ├── workers/            # Cross-platform worker pool (WorkerQueue, tasks)
│   ├── spatial/            # Spatial algorithms (k-means, kd-tree, b-tree, radix sort, quantize-1d)
│   ├── voxel/              # Voxel generation (BVH, octree, GPU voxelization)
│   ├── mesh/               # Mesh generation (collision/marching cubes)
│   ├── render/             # GPU splat rasterizer (for image output)
│   ├── gpu/                # WebGPU compute (k-means, knn, edge cost, voxelize, rasterize)
│   └── utils/              # Logger, math, SH rotation, WebP codec
└── cli/                    # Node.js CLI (NOT platform-agnostic)
    ├── index.ts            # CLI entry, argument parsing
    ├── node-device.ts      # WebGPU device creation for Node
    └── node-file-system.ts # Node.js file system implementations
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [playcanvas/splat-transform](https://github.com/playcanvas/splat-transform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
