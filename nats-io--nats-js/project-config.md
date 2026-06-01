---
trigger: always_on
description: NATS.js v3 monorepo: JavaScript/TypeScript client libraries for NATS messaging.
---

# CLAUDE.md - nats.js

NATS.js v3 monorepo: JavaScript/TypeScript client libraries for NATS messaging.
Supports Deno, Node.js/Bun, and browser (W3C WebSocket) runtimes.

## Repository Structure

This is a **dual workspace** monorepo — Deno workspace (`deno.json`) for
development/testing and npm workspace (`package.json`) for Node.js publishing.

### Modules (under `@nats-io` scope on JSR and npm)

| Directory         | Package                   | Description                                                                          | Dependencies         |
| ----------------- | ------------------------- | ------------------------------------------------------------------------------------ | -------------------- |
| `core/`           | `@nats-io/nats-core`      | Base NATS client (protocol, connection, pub/sub, request/reply, WebSocket transport) | nkeys, nuid          |
| `jetstream/`      | `@nats-io/jetstream`      | JetStream (persistent streaming)                                                     | nats-core            |
| `kv/`             | `@nats-io/kv`             | Key-Value store                                                                      | nats-core, jetstream |
| `obj/`            | `@nats-io/obj`            | Object Store                                                                         | nats-core, jetstream |
| `services/`       | `@nats-io/services`       | Services framework                                                                   | nats-core            |
| `transport-deno/` | `@nats-io/transport-deno` | TCP transport for Deno                                                               | nats-core            |
| `transport-node/` | `@nats-io/transport-node` | TCP transport for Node.js/Bun                                                        | nats-core            |

### Other directories

- `nst/` — Shared Deno test utilities (NatsServer launcher, Lock, setup/cleanup
  helpers)
- `bin/` — Build/release tooling scripts (version checks, CJS import fixer,
  bundler)
- `docs/` — Generated JSDoc documentation

## Development Commands

**Primary toolchain is Deno.** Source is written in TypeScript with `.ts`
extensions. Node.js packages are built via a CJS transpilation step.

### Deno (primary — runs all module tests)

```bash
deno task test              # Clean, lint, then run all module tests
deno task test-core         # Test only core module
deno task test-jetstream    # Test only jetstream module
deno task test-kv           # Test only kv module
deno task test-obj          # Test only obj module
deno task test-services     # Test only services module
deno task test-unsafe       # Test TLS unsafe tests (core/unsafe_tests/)
deno task test-transport-deno  # Test Deno transport

deno task lint              # Lint all modules (runs deno lint per module)
deno fmt                    # Format all code
deno fmt --check            # Check formatting without modifying

deno task clean             # Remove build artifacts and coverage
deno task cover             # Generate and open HTML coverage report
deno task check-versions    # Verify version consistency across modules
```

### Node.js (transport-node only)

```bash
# From repo root:
npm install --workspaces
npm run build --workspaces          # Build all workspace packages for npm
npm run prepack --workspaces        # Same as build (prepack calls build)

# From transport-node/:
npm test                             # Build then run Node.js test suite
npm run coverage                     # Build then run with lcov output
```

### Module npm build process (core, jetstream, kv, obj, services)

Each module's npm build:

1. `npm run pre-process` — runs `bin/cjs-fix-imports.ts` to copy `.ts` sources
   to `build/src/` with import paths fixed for CJS (removes `.ts` extensions)
2. `npm run build-cjs` — runs `tsc` to compile `build/src/` into `lib/`

The `transport-node` package compiles directly from `src/` to `lib/` (no
pre-process step).

## Code Conventions

### Source layout per module

- `src/mod.ts` — Public API exports (what consumers import)
- `src/internal_mod.ts` — Internal/extended API exports (used by other modules
  in the monorepo via `@nats-io/<pkg>/internal`)
- `src/types.ts` — Type definitions
- `src/version.ts` — Auto-generated version string (do not edit manually)
- `tests/` — Deno test files (`*_test.ts`)

### TypeScript style

- **Deno-first**: All source uses `.ts` extensions in imports (e.g.,
  `import { foo } from "./bar.ts"`)
- **Strict mode**: `"strict": true` in `deno.json` compilerOptions
- **Formatting**: `deno fmt` (2-space indent, double quotes implied by Deno
  defaults)
- **Linting**: `deno lint` per module
- **No eslint/prettier** — Deno's built-in formatter and linter are the only
  tools
- **Separate `type` imports**: Use `import type { Foo }` for type-only imports,
  `import { Bar }` for values
- **Error classes**: Custom error classes extend `Error`, set `this.name` in
  constructor (see `core/src/errors.ts`)
- **License header**: All source files must include the Apache 2.0 license
  header (see any `.ts` file)
- **Minimal external dependencies**: Avoid adding new dependencies. The project
  targets multiple runtimes.

### Module cross-references (Deno workspace)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nats-io/nats.js](https://github.com/nats-io/nats.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
