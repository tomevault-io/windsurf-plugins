---
trigger: always_on
description: transforms.ts           AST tree transforms (obfuscateLocals)
---

# Ruam

JS VM obfuscator — compiles JavaScript functions into custom bytecode executed by an embedded virtual machine interpreter.

## Quick Reference

-   **Package manager**: `bun` (bun workspaces)
-   **Build all**: `bun run build` (lib + worker + website)
-   **Build lib only**: `bun run build:lib` (tsup, ESM-only)
-   **Build worker**: `bun run build:worker` (esbuild + manifest generation)
-   **Build website**: `bun run build:web` (Next.js static export)
-   **Typecheck**: `bun run typecheck` (tsc --noEmit)
-   **Test**: `bun run test` (bun:test, 2328 tests)
-   **Test watch**: `bun run test:watch`
-   **Dev server**: `bun run dev` (Next.js dev)
-   **Stats**: `bun run stats`
-   **Clean**: `bun run clean` (remove dist, .next, caches)
-   **Fresh build**: `bun run build:fresh` (clean + build)
-   **Fresh test**: `bun run test:fresh` (clean + test)
-   **Node**: >= 18, **Module**: ESM (`"type": "module"`)

## Project Structure

```
src/
  index.ts                  Public API: obfuscateCode (sync), obfuscateFile, runVmObfuscation (async)
  cli.ts                    CLI entry point (bin: ruam)
  transform.ts              Main orchestrator: parse -> compile -> assemble
  types.ts                  TypeScript interfaces (VmObfuscationOptions, PresetName, BytecodeUnit, etc.)
  constants.ts              Shared constants (parser plugins, globals list, limits, hash/mixing constants, binary tags)
  babel-compat.ts           Babel ESM/CJS compatibility layer (normalized traverse/generate exports)
  presets.ts                Preset definitions (low/medium/max) + resolveOptions()
  tuning.ts                 Centralized tuning parameters: TuningProfile, getTuningProfile(intensity)
  option-meta.ts            Single source of truth for option metadata (labels, categories, CLI flags)
  preprocess.ts             Optional identifier renaming
  structural-choices.ts     Per-build structural variation: dispatch/return polymorphism, statement shuffling, expression noise
  browser-entry.ts          Browser ESM entry point (re-exports obfuscateCode, presets, types)
  browser-worker.ts         Web Worker for playground (message protocol: {id, code, options} → {id, result, elapsed})
  browser-crypto-shim.ts    Polyfill for Node.js crypto.randomBytes() using Web Crypto API

  compiler/
    index.ts                Function compilation entry point
    opcodes.ts              Opcode enum (~317 opcodes in 26 categories) + per-file shuffle map
    capture-analysis.ts     Capture analysis for register promotion (Tier 1)
    optimizer.ts            Peephole optimizer (Tier 2) + superinstruction fusion (Tier 3)
    emitter.ts              Bytecode emitter + constant pool
    scope.ts                Compile-time scope analysis + register allocation
    encode.ts               Bytecode serialization (binary + custom encoding + FNV-1a+LCG cipher) + string constant encoding
    crypto.ts               Build-time FNV-1a+LCG stream cipher + custom alphabet binary encoding
    fingerprint.ts          Build-time fingerprint computation
    rolling-cipher.ts       Build-time rolling cipher encryption + implicit key derivation
    basic-blocks.ts         Shared basic block identification (used by block permutation + incremental cipher)
    block-permutation.ts    Bytecode basic block shuffling via Fisher-Yates with fall-through JMP insertion
    incremental-cipher.ts   Block-epoch keyed instruction encryption (build-time)
    opcode-mutation.ts      MUTATE opcode insertion at pseudo-random intervals with cumulative mutation state
    visitors/
      index.ts              Barrel exports
      statements.ts         Statement compilation (if, for, while, switch, try, etc.)
      expressions.ts        Expression compilation (calls, members, operators, etc.)
      classes.ts            Class compilation (methods, properties, inheritance)

  naming/
    token.ts                NameToken class — opaque handle with fail-fast .name/.toString()
    scope.ts                NameScope class — child scope with own PRNG, claims tokens + deriveSeed()
    registry.ts             NameRegistry class — central coordinator, resolveAll(), alphabet, createDynamicGenerator()
    reserved.ts             JS reserved words + excluded names sets
    claims.ts               Canonical key definitions for RuntimeNames + TempNames scopes
    setup.ts                Bridge: NameRegistry → RuntimeNames/TempNames (backward compat)
    compat-types.ts         RuntimeNames and TempNames type interfaces
    index.ts                Barrel exports

  ruamvm/
    nodes.ts                JS AST node types (~36 node kinds) + factory functions
    emit.ts                 Recursive emitter: AST -> minified JS with precedence-aware parens
    assembler.ts            VM runtime orchestrator: assembles IIFE from AST builders, incl. shielded mode
    transforms.ts           AST tree transforms (obfuscateLocals)
    structural-transforms.ts Per-build AST walk: control flow, declaration style, expression noise transforms
    mba.ts                  Mixed Boolean Arithmetic AST tree transform
    opaque-predicates.ts    Opaque predicate library: 5 always-true/false mathematical families
    handler-aliasing.ts     Handler aliasing: structurally different implementations per build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owengregson/Ruam](https://github.com/owengregson/Ruam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
