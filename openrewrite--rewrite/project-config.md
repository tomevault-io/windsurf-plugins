---
trigger: always_on
description: This file provides guidance to Claude Code when working with the OpenRewrite TypeScript implementation.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the OpenRewrite TypeScript implementation.

## Module Overview

TypeScript implementation of OpenRewrite for JavaScript/TypeScript source code transformations, plus JSON and YAML support. Includes parsers, AST models, visitors, printers, and an RPC bridge for Java communication.

Self-contained Node.js project, separate from the Java monorepo build system.

## Project Setup

From `rewrite-javascript/rewrite/`:
```bash
npm install
```

Via Gradle (from repo root):
```bash
./gradlew :rewrite-javascript:npmInstall
./gradlew :rewrite-javascript:npm_test
./gradlew :rewrite-javascript:npm_run_build
```

Requires Node.js 18+.

## Running Tests

```bash
# Full suite (typecheck + build + test)
npm test

# Fast iteration (skip typecheck)
npm run testhelper

# Type-checking only
npm run typecheck

# Individual test file
npm run testhelper -- test/javascript/recipes/order-imports.test.ts
```

Available npm scripts: `prebuild`, `build`, `postbuild`, `typecheck`, `dev`, `test`, `testhelper`, `build:fixtures`, `ci:test`, `start`.

### Java RPC Integration Tests

Tests under `test/rpc/` that exercise real Java recipes spawn `org.openrewrite.maven.rpc.JavaRewriteRpc` via `JavaRpcTestServer` (see `src/rpc/java-rpc-client.ts`). They need a classpath file generated from the Java side:

```bash
# From repo root
./gradlew :rewrite-javascript:generateTestClasspath
```

This writes `rewrite-javascript/rewrite/test-classpath.txt` (gitignored). Alternatively set `REWRITE_JAVASCRIPT_CLASSPATH` to override. Tests in `test/rpc/java-recipe-via-rpc.test.ts` skip cleanly with a one-line warning when neither is configured.

## Directory Structure

```
rewrite-javascript/rewrite/
├── src/
│   ├── index.ts                         # Main entry point / re-exports
│   ├── tree.ts, visitor.ts, recipe.ts   # Core framework
│   ├── markers.ts, execution.ts         # Metadata, execution context
│   ├── print.ts, parser.ts              # Base printer, base parser
│   ├── util.ts, uuid.ts                 # Utilities
│   ├── java/                            # Java LST model
│   │   ├── tree.ts                      # J namespace (Java AST)
│   │   ├── visitor.ts                   # JavaVisitor
│   │   ├── print.ts                     # Java-to-source printer
│   │   ├── rpc.ts                       # RPC sender/receiver for Java
│   │   └── type.ts, type-visitor.ts     # Java type system
│   ├── javascript/                      # JavaScript/TypeScript
│   │   ├── tree.ts                      # JS namespace (JavaScript AST)
│   │   ├── visitor.ts                   # JavaScriptVisitor
│   │   ├── print.ts                     # JS-to-source printer
│   │   ├── parser.ts                    # JS/TS parser
│   │   ├── rpc.ts                       # RPC sender/receiver for JS
│   │   ├── assertions.ts               # Test helpers: typescript(), javascript(), jsx(), tsx(), packageJson()
│   │   ├── add-import.ts, remove-import.ts  # Import manipulation
│   │   ├── recipes/                     # Built-in recipes (order-imports, change-import, add-dependency, etc.)
│   │   ├── format/                      # Formatting visitors
│   │   ├── cleanup/                     # Cleanup recipes (add-parse-int-radix, prefer-optional-chain, etc.)
│   │   ├── migrate/                     # Migration recipes (es6/, typescript/)
│   │   ├── search/                      # Search patterns
│   │   └── templating/                  # Template engine
│   ├── json/                            # JSON support (tree, visitor, print, rpc, recipes)
│   ├── yaml/                            # YAML support (tree, visitor, print, rpc, recipes)
│   ├── search/                          # Cross-language search utilities
│   ├── text/                            # Plain text support
│   ├── rpc/                             # RPC infrastructure
│   │   ├── queue.ts                     # Message queue
│   │   ├── rewrite-rpc.ts              # Core RPC protocol
│   │   ├── server.ts                    # RPC server
│   │   ├── recipe.ts                    # Recipe RPC bridge
│   │   ├── trace.ts                     # RPC tracing/debugging
│   │   └── request/                     # Request types (parse, visit, get-object, etc.)
│   └── test/                            # Testing infrastructure
│       └── rewrite-test.ts              # RecipeSpec class, rewriteRun()
├── test/                                # Vitest tests (mirrors src/ structure)
│   ├── javascript/                      # JS/TS tests
│   │   ├── recipes/                     # Recipe tests
│   │   ├── fixtures/                    # Test npm projects
│   │   ├── parser/, format/, cleanup/   # Category tests
│   │   └── search/, templating/, migrate/
│   ├── java/                            # Java model tests
│   ├── json/, yaml/                     # JSON/YAML tests
│   └── rpc/                             # RPC integration tests
├── tsconfig.json
├── vitest.config.mts
└── package.json                         # name: @openrewrite/rewrite
```

## Development Patterns

### Async Visitor Pattern

**All visitor methods are async.** This supports the RPC nature of the framework.

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openrewrite/rewrite](https://github.com/openrewrite/rewrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
