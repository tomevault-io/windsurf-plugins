---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Endfield Calc is a production chain calculator for "Arknights: Endfield" — a single-page React + TypeScript app that computes resource requirements, production ratios, and facility needs for potentially circular production loops. Deployed to GitHub Pages at `/endfield-calc/`.

## Commands

```bash
pnpm install          # Install dependencies
pnpm dev              # Start dev server
pnpm build            # Type-check (tsc -b) then Vite build
pnpm lint             # ESLint
pnpm test             # Vitest (run all tests)
pnpm knip             # Detect unused code/exports
```

Tests are in `src/tests/lib/`. Run a single test file with:
```bash
pnpm vitest run src/tests/lib/calculator.test.ts
```

## Architecture

### Core Algorithm (`src/lib/calculator.ts`)

The central piece of the codebase. Implements a graph-based production planner.

**Pipeline** (`calculateProductionPlan`, outer loop runs up to 100 iterations for backtracking):

1. **Build bipartite graph** (`buildBipartiteGraph`) — DFS from each target. Items + recipes as two node types. For each item pick one recipe via `selectRecipe` heuristic: prefer single-output recipes, prefer non-circular (not consuming `visitedPath`), else first. Respects `recipeOverrides` (user-pinned) and `recipeConstraints` (backtrack exclusions). Forced raw materials (`forcedRawMaterials` + `manualRawMaterials`) terminate recursion. Byproducts added to graph as non-raw nodes but NOT marked visited — lets them be re-discovered with external producers.

2. **SCC detection** (`detectSCCs`) — Tarjan on bipartite graph. Successors: item → consuming recipes, recipe → output items. Each SCC captures items, recipes, and `externalInputs` (inputs from recipes outside SCC).

3. **Condensed DAG + topo sort** (`buildCondensedDAGAndSort`) — collapse each SCC into one super-node. Kahn's algorithm topologically orders.

4. **Flow calculation** (`calculateFlows`, reverse topo order so consumers processed before producers):
   - **Plain recipe node**: facility count = max over outputs of `demand / ratePerFacility`. Propagate input demand `= calcRate(input.amount, craftingTime) * facilityCount` to `itemDemands`.
   - **SCC node**: delegate to `solveSCCFlow`. If fails, record in `invalidSCCs`.

5. **SCC solver** (`solveSCCFlow`, 5 phases):
   - Phase 1: external demands for internal items (external consumers + target rates).
   - Phase 2: external-output demands — SCC recipes producing items OUTSIDE the SCC with non-zero demand. Pins those recipes' facility counts to `demand / rate`.
   - Phase 3: build `n × m` matrix of net production coefficients `calcRate(output, t) - calcRate(input, t)` per (item, recipe). Before solving, drop impossible disposal-item rows via `filterImpossibleDisposalRows` (forced-disposal item with all-zero LHS + non-zero RHS — surplus handled later by `injectDisposalRecipes`). If any pinned, substitute their contribution into RHS and solve reduced system on `freeM` free recipes via `solveOverdetermined` (linear-solver.ts): `numVars == 0` → `[]`; overdetermined + 1 free var → `max(b/a)`; overdetermined multi-var → pick `numVars` rows with largest `|RHS|` and solve that square system; else plain `solveLinearSystem`. Fallback on no-solution → `tryExtendSCCWithFeeders`.
   - Phase 4: deficit propagation. For each internal item compute net production, if `externalDemand - netProduction > 0` push deficit into `itemDemands` (via `Math.max`, NOT addition — external demand already counted in Phase 1).
   - Phase 5: propagate consumption back to `scc.externalInputs` (demand += consumption).

6. **Feeder extension** (`tryExtendSCCWithFeeders`) — fallback when SCC unsolvable. For each overridden item in SCC, find alternate non-SCC producing recipe (feeder) with at least one non-SCC input. Adds feeders to graph + SCC, pins the override recipe to its external demand, solves extended system. Validates: if target item still has deficit, rollback all mutations (`addedRecipeIds`, `addedItemIds`, `addedConsumptionEdges`, original `scc.recipes`/`externalInputs` sets) and return false. On success adds `scc.id` to `resolvedSCCIds` (cycle linearized — excluded from `detectedCycles`).

7. **Disposal injection** (`injectDisposalRecipes`) — for each `forcedDisposalItems` with surplus (`production - consumption - targetDemand > 0`), find disposal recipe (empty outputs), add to graph with `facilityCount = surplus / rate`.

8. **Backtracking** (`backtrackRecipeChoices`) — when `invalidSCCs` non-empty, collect items in those SCCs with `recipeChoices` (multiple options), pick one with next untried recipe, extend `recipeConstraints` excluding all recipes up to current index, rerun. Gives up when all combos exhausted → returns best-effort graph with `invalidCycles`.

9. **Build final graph** (`buildProductionGraph`) — assemble `ProductionDependencyGraph` with nodes (sum production across ALL producing recipes for multi-producer items), edges (item↔recipe both directions), `detectedCycles` (excluding resolved SCCs), `invalidCycles`.

**Invariants**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamboChen/endfield-calc](https://github.com/JamboChen/endfield-calc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
