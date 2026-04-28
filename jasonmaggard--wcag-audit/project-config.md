---
trigger: always_on
description: > Generated: 2026-02-26
---

# WCAG Audit CLI — Redesign Plan

> Generated: 2026-02-26
> Completed: 2026-02-27
> Status: **All Phases Complete** ✅

---

## Overview

Restructure the monolithic 3-script WCAG audit pipeline (~2,635 lines across 3 files) into a modular Commander.js CLI app with subcommands (`wcag-audit audit`, `wcag-audit report`, `wcag-audit extract-routes`). The existing code moves untouched to `rough_draft/`. The new structure splits code across ~20 focused modules organized by responsibility.

Work proceeds in 6 phases. Each phase ends with an end-to-end verification step and a commit point.

### Key Decisions

| Decision | Choice | Rationale |
|---|---|---|
| CLI framework | Commander.js | Most widely used Node CLI framework, zero build step, mature ESM support |
| File extensions | `.js` with `"type": "module"` | Cleaner than `.mjs`, entire repo is ESM |
| Entry pattern | Subcommands | Single entry point, self-documenting `--help`, consistent option parsing |
| False-positive toggle | `createFilter(enabled)` pattern | Returns real or no-op functions — no `if` checks scattered through engine code |
| Third-party disclaimers | From existing JSONC rule map | `thirdPartyVendors` already has all needed fields |
| Slug style | Clean dashes | `/invoice/:id/payment` → `invoice-id-payment` (no `dyn-` prefix, no `__`) |
| iFrame scanning | Opt-in via `--scan-iframes` | Default false; no explicit Stripe exclusion needed |
| Route extraction | Standalone subcommand | Not part of audit pipeline — run separately or as CI pre-step |
| CSS management | External `.css` file, inlined at generation time | Report stays self-contained; CSS becomes editable |
| Test strategy | Manual verification at each phase | CLI smoke tests and output comparison against rough draft |

### Architectural Principles

These apply across every module:

1. **Thin commands, fat services.** Command handlers are ~20-line adapters that parse CLI input, build a context, and delegate. All domain logic lives in service modules.

2. **Session context object.** A single `AuditContext` / `ReportContext` object is created by the command handler and threaded through the pipeline. Cross-cutting concerns (logger, output dir, timeouts, filter, browser instances) live on this object. Adding a concern means changing one object, not every function signature.

3. **Structured logger.** No bare `console.log`. A `createLogger({ quiet, verbose })` function returns `{ info, debug, warn, error, success }` methods. Lives on the context. Enables `--quiet` for CI and `--verbose` for debugging.

4. **No side effects at module scope.** Every module exports pure functions (or functions that accept a context). No `fs.mkdirSync`, no `setConfig()`, no `throw` at the top level. All side effects happen in command handlers. You can `import` any module without triggering I/O.

5. **Consistent error strategy.** Commands catch at the top and set `process.exitCode = 1`. Everything else throws. Commander's `.exitOverride()` keeps this clean.

6. **Barrel exports.** Each `src/` subdirectory has an `index.js` re-exporting its public API. Consumers write `import { normalizeSlug, esc } from '../utils/index.js'` instead of three separate imports. Internal refactoring stays safe.

7. **Resolve assets via `import.meta.url`.** `styles.css` and other file-relative assets use `new URL('./styles.css', import.meta.url)` — works regardless of `process.cwd()`.

8. **Output writer abstraction.** A shared `writeOutputs(outputDir, manifest, log)` function handles file writing + logging. One place to add dry-run mode, compression, or S3 upload later.

9. **Split defaults by domain.** Audit-time constants (`TIMEOUTS`, `VIEWPORT`) and report-time constants (`LIMITS`, fallback rule maps) live in separate files so the two commands don't cross-load unused config.

10. **Type safety without TypeScript.** A `jsconfig.json` enables VS Code's `checkJs` + IntelliSense from JSDoc annotations. Zero build step, full autocomplete and type checking.

---

## Directory Structure

```
wcag-audit/
├── rough_draft/                    # Backup of all current files (DO NOT DELETE)
│   ├── run-wcag-audit.mjs
│   ├── generate-wcag-conformance-report.mjs
│   ├── extract-routes.mjs
│   ├── run-wcag-pipeline.sh
│   └── package.json.bak
│
├── bin/
│   └── wcag-audit.js               # Commander entry point (~40 lines)
│
├── src/
│   ├── commands/
│   │   ├── audit.js                # ~25 lines: parse opts → build context → delegate
│   │   ├── report.js               # ~25 lines: parse opts → build context → delegate
│   │   └── extract-routes.js       # ~20 lines: parse opts → delegate to extractors
│   │
│   ├── engines/
│   │   ├── ibm.js                  # configureIbm(), runIbmScan(ctx), closeIbm()
│   │   ├── axe.js                  # runAxeScan(ctx)
│   │   ├── lighthouse.js           # runLighthouseScan(ctx), normalizeLighthouseViolations()
│   │   └── index.js                # barrel: re-exports all engines
│   │
│   ├── filters/
│   │   ├── false-positives.js      # createFilter(enabled) + individual filter fns
│   │   └── index.js                # barrel
│   │
│   ├── scanner/
│   │   ├── route-scanner.js        # scanRoutes(ctx) — main loop
│   │   ├── interactive-states.js   # INTERACTIVE_STATES array

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JasonMaggard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
