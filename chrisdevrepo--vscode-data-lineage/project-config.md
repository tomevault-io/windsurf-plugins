---
trigger: always_on
description: VS Code extension for visualizing SQL database object dependencies from .dacpac files or database import (via MSSQL extension). React + ReactFlow frontend, graphology for graph data, dagre for layout.
---

# Project Context

VS Code extension for visualizing SQL database object dependencies from .dacpac files or database import (via MSSQL extension). React + ReactFlow frontend, graphology for graph data, dagre for layout.

## Architecture

| Runtime | Bundler | Entry | Output |
|---------|---------|-------|--------|
| Node.js (extension host) | esbuild | `src/extension.ts` | `out/extension.js` (CJS) |
| Browser (webview) | Vite | `src/index.tsx` | `dist/assets/index.js` (ESM) |

### Key Directories

- `src/engine/` — dacpac extraction, database import, SQL parsing, graph building
- `src/components/` — React UI (ReactFlow canvas, toolbar, filters, modals)
- `src/hooks/` — Graph state (`useGraphology`), trace state (`useInteractiveTrace`), loader (`useDacpacLoader` — handles both dacpac and DB sources), dropdown state (`useDropdown` — shared open/close + outside-click), overview mode (`useOverviewMode` — schema-level view state machine, auto-trigger guard, `resetUserChoice`)
- `src/extension.ts` — VS Code API, webview lifecycle, message routing

### Extraction Pipeline (DRY principle)

Both dacpac and database import produce `ExtractedObject[]` + `ExtractedDependency[]` + `ConstraintMaps`.
Post-extraction logic (schema aggregation, constraint enrichment, catalog building) lives exclusively
in `modelBuilder.ts` and `types.ts`. The extractors are thin format adapters — they translate their
source format into the shared intermediate types and nothing more.

### Key Files

| File | Purpose |
|------|---------|
| `src/engine/connectionManager.ts` | MSSQL extension API wrapper, connection management |
| `src/engine/dmvExtractor.ts` | Build DacpacModel from database import via DMV queries |
| `src/types/mssql.d.ts` | Type declarations for MSSQL extension API |
| `assets/defaultParseRules.yaml` | Built-in parse rules (17 rules, 5 categories) |
| `assets/dmvQueries.yaml` | Built-in DMV queries (7 queries) |
| `docs/PARSE_RULES.md` | Custom parse rules guide |
| `docs/DMV_QUERIES.md` | Custom DMV queries guide |
| `docs/PROFILING_PATTERNS.md` | Table profiling SQL patterns reference |
| `src/ai/tools.ts` | AI tool pure functions (8 tools): 7 read-only queries + `validateEnrichView`. `shouldInline()` gates catalog/detail delivery mode (not CT/BB). Zero VS Code imports. |
| `src/ai/tokenBudget.ts` | Token budget: `INLINE_TOKEN_BUDGET`, `shouldInline()`, `estimateTokens()`, `CONTEXT_PRESSURE_THRESHOLD`, `REGEX_MAX_LENGTH`. CT/BB always use state machine; budget applies to catalog/detail tools only. Zero VS Code imports. |
| `src/ai/prompts.ts` | Central prompt dispatcher: `buildSystemPromptBase()`, `CT_MODE_PROMPT`, `CT_DEP_MODE_PROMPT`, `BB_MODE_PROMPT`. Zero VS Code imports. |
| `src/ai/aiPresenter.ts` | Compact LLM presentation layer: `strip()`, `presentNode/Column/Schema/Neighbor/Filter()`, `edgeApiType()` (explicit type map with 'read' fallback). Zero business logic, zero VS Code imports. |
| `src/ai/graphUtils.ts` | `buildBareGraph()` — connection-only graphology graph for BFS in AI tools |

## Build & Test

```bash
npm run build         # Build extension + webview
npm run watch         # Watch extension only
npm test              # Unit tests (996 tsx + 112 vitest + snapshot)
npm run test:internal # AI SM tests (184+98+66 = 348 tsx) — separate suite
```

Press F5 to launch Extension Development Host.

## Test Suite

| File | Tests | Purpose |
|------|-------|---------|
| `test/dacpacExtractor.test.ts` | 109 | Dacpac extraction, filtering, edge integrity, Fabric SDK, security, constraints, `parseDspPlatform`, `dbPlatform`, `pkOrdinal`, Phase 1→2 bridge flow |
| `test/graphBuilder.test.ts` | 218 | Graph construction, layout, BFS trace, directional edge filtering, cycle filtering, bidirectional correctness, determinism, virtual external nodes, CLR method suppression, buildSchemaEdges, buildSchemaGraph |
| `test/parser-edge-cases.test.ts` | 204 | Syntactic parser tests: all 17 rules + edge cases + cleansing pipeline + regression guards |
| `test/graphAnalysis.test.ts` | 81 | Graph analysis: islands, hubs, orphans, longest path, cycles, external refs |
| `test/dmvExtractor.test.ts` | 193 | DMV extractor: synthetic data, column validation, type formatting, fallback body direction, constraints, external tables, schema placeholder expansion, `dbPlatform` via `mapEnginePlatform`, `pkOrdinal` from columns query |
| `test/tsql-complex.test.ts` | 55 | SQL pattern tests: targeted SQL files covering each parser pattern; expected results in `-- EXPECT` comments |
| `test/projectStore.test.ts` | 136 | Project store: createProject, updateProject, deleteProject, migrateProjectStore, generateProjectName, addFilterProfile, deleteFilterProfile, serializeFilter, deserializeFilter |
| `test-internal/ai-tools.test.ts` | 184 | AI tool pure functions: getContext, searchObjects, getObjectDetail, runBfsTrace (level + path mode), runAnalysis, searchDdl, getDdlBatch, validateEnrichView, autoFixEnrichView, validateQuery, safeRegex, validateMarkdownFormat |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChrisDevRepo/vscode_data_lineage](https://github.com/ChrisDevRepo/vscode_data_lineage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
