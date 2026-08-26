---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kaiord is an open-source health & fitness data framework. A TypeScript monorepo for creating, converting, and managing data across FIT, TCX, ZWO, GCN, and KRD formats.

**Packages:**

- `@kaiord/core` - Domain types, schemas, ports, use cases (no adapter implementations)
- `@kaiord/fit` - FIT format adapter (Garmin FIT SDK)
- `@kaiord/tcx` - TCX format adapter (fast-xml-parser)
- `@kaiord/zwo` - ZWO format adapter (fast-xml-parser, XSD validation)
- `@kaiord/garmin` - Garmin Connect API (GCN) format adapter
- `@kaiord/garmin-connect` - Garmin Connect API client (SSO auth, push/list workouts)
- `@kaiord/cli` - Command-line interface
- `@kaiord/mcp` - Model Context Protocol (MCP) server for AI/LLM integration
- `@kaiord/garmin-bridge` - Chrome extension for SPA-to-Garmin Connect integration (private)
- `@kaiord/train2go-bridge` - Chrome extension for reading Train2Go coaching plans (private)
- `@kaiord/whoop-bridge` - Chrome extension for WHOOP health data via session piggyback (private)
- `@kaiord/trainingpeaks-bridge` - Chrome extension for TrainingPeaks body metrics (private)
- `@kaiord/tanita-bridge` - Chrome extension for MyTANITA body-composition export (private)
- `@kaiord/workout-spa-editor` - React web application (private)

## Commands

```bash
# Install and build
pnpm install
pnpm -r build

# Test
pnpm -r test                    # All tests
pnpm -r test:watch              # Watch mode
cd packages/core && pnpm test   # Single package

# Lint and format
pnpm lint                       # Lint + type check + format check
pnpm lint:fix                   # Auto-fix all
pnpm format                     # Format with Prettier

# Changesets (for version-worthy changes)
pnpm exec changeset             # Create changeset before PR

# Archive maintenance
pnpm lint:archive               # Enforce archive folder-vs-Completed invariant
pnpm lint:archive-index         # Verify archive/README.md is up to date
pnpm archive:index              # Regenerate archive/README.md
pnpm test:scripts               # Run node:test for scripts/*.test.mjs

# NPM optimization (Claude Code skills)
/check-deps                     # Analyze dependencies (unused, duplicates, security)
/analyze-bundle                 # Check bundle sizes and optimization opportunities
/optimize-imports               # Refactor imports for better tree-shaking
```

## Quality Standards

**CRITICAL: Zero Tolerance for Warnings and Errors**

When working on this codebase, ALL problems must be fixed, regardless of whether they were introduced in the current branch or pre-existing:

- ✅ **Zero ESLint warnings** - All linting rules must pass
- ✅ **Zero TypeScript errors** - Strict type checking with no `any` escapes
- ✅ **Zero test warnings** - Clean test output (React act(), accessibility, etc.)
- ✅ **Zero build warnings** - Vite, ESBuild, etc. must produce clean output
- ✅ **Zero IDE warnings** - SonarQube, accessibility, and static analysis warnings must be resolved (treat as lint errors)
- ✅ **Coverage thresholds met** - 80% for core packages, 70% for frontend
- ✅ **All tests passing** - 100% pass rate across all packages
- ✅ **Mechanical guards passing** - `pnpm test:scripts` enforces:
  - `check-no-zustand-writethrough.mjs` — no Zustand store writes Dexie directly (R-DexieImport / R-PersistStateImport / R-AppDexieImport)
  - `check-no-pii-leakage.mjs` — toast and `console.*` first arguments under `packages/workout-spa-editor/src/{components,hooks,lib}/**` are static (bare string literal or top-level SCREAMING_SNAKE_CASE constant referencing a literal); rule R-PIIInterpolation
  - `check-no-library-dual-mount.mjs` — no-dual-mount invariant for the Library content component (spec/spa-routing); only `LibraryPage.tsx` and `TemplatePickerDialog.tsx` may import `organisms/WorkoutLibrary` / `WorkoutLibrary/WorkoutLibrary` / `LibraryDialogContent` (R-LibraryNoDualMount)
  - `check-session-match-id-shape.mjs` — every `coachingActivityId:` literal in a `sessionMatches` write call site, plus every `[profileId+coachingActivityId]` Dexie reader, MUST be constructed via `buildCoachingActivityId(...)`, `toPersistedCoachingActivityId(...)`, or a `CoachingActivityRecord.id` property access; rule R-SessionMatchIdShape (see `.omc/autopilot/bug-trace.md` §H7 for the original SHORT/COMPOSITE divergence)
  - `check-no-barrel-test-suites.mjs` — no `*.test.{ts,tsx}` may target a subject module that is a pure re-export barrel; test the source modules instead (R-NoBarrelTestSuite, spec/test-minimality)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pablo-albaladejo/kaiord](https://github.com/pablo-albaladejo/kaiord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
