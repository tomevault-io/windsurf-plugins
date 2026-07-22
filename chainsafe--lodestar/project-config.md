---
trigger: always_on
description: - **Target branch:** `unstable` (never `stable`)
---

# AGENTS.md

## Critical rules

- **Target branch:** `unstable` (never `stable`)
- **Pre-push:** run `pnpm lint`, `pnpm check-types`, `pnpm test:unit` before every push
- **Relative imports:** use `.js` extension in TypeScript ESM imports
- **No `any`:** avoid `any` / `as any`; use proper types or justified `biome-ignore`
- **No `lib/` edits:** never edit `packages/*/lib/` — these are build outputs
- **Follow existing patterns** before introducing new abstractions
- **Structured logging** with specific error codes (not generic `Error`)
- **Incremental commits** after review starts — do not force push unless maintainer requests it
- **Communication style:** DO NOT use `—`. Keep communication succinct and human-friendly. NO AI SLOP VERBOSITY.

## Project overview

Lodestar is a TypeScript implementation of the Ethereum consensus client
(beacon node and validator client). It is maintained by ChainSafe Systems
and serves as:

- **Production beacon node** for Ethereum's proof-of-stake consensus layer
- **Validator client** for stakers running validators
- **Light client** implementation with browser support
- **Reference implementation** for TypeScript/JavaScript ecosystem

## Directory structure

```
/packages/
  api/              # REST API client and server
  beacon-node/      # Beacon chain node implementation
  cli/              # Command-line interface
  config/           # Network configuration (mainnet, sepolia, etc.)
  db/               # Database abstraction (LevelDB)
  era/              # Era file handling for historical data
  fork-choice/      # Fork choice implementation (proto-array)
  logger/           # Logging utilities
  params/           # Consensus parameters and presets
  reqresp/          # libp2p request/response protocol
  spec-test-util/   # Test harness for consensus spec tests
  state-transition/ # State transition functions
  test-utils/       # Shared utilities for testing
  types/            # SSZ type definitions
  utils/            # Shared utilities
  validator/        # Validator client

/configs/          # Network configuration files
/docs/             # Documentation source
/scripts/          # Build and release scripts
/dashboards/       # Grafana dashboard JSON files
```

## Build commands

All commands use `pnpm` as the package manager.

```bash
# Install dependencies
corepack enable
pnpm install

# Build all packages
pnpm build

# Build a specific package (faster iteration)
pnpm --filter @lodestar/beacon-node build

# Run linter (biome)
pnpm lint

# Fix lint issues automatically
pnpm lint:fix

# Type check all packages
pnpm check-types

# Type check a specific package
pnpm --filter @lodestar/beacon-node check-types

# Run unit tests (fast, minimal preset)
pnpm test:unit

# Run specific test file with project filter
pnpm vitest run --project unit test/unit/path/to/test.test.ts

# Run tests matching a pattern
pnpm vitest run --project unit -t "pattern"

# Run spec tests (requires downloading first)
pnpm download-spec-tests
pnpm test:spec

# Run a single spec test file (located in packages/beacon-node) from the repository root — use spec-mainnet
# for the mainnet preset
pnpm vitest run --project spec-minimal test/spec/presets/transition.test.ts

# Filter spec tests within a file by pyspec test name pattern
pnpm vitest run --project spec-minimal test/spec/presets/transition.test.ts \
  -t "transition_with_voluntary_exit_right_after_fork"

# Download nightly artifacts from ethereum/consensus-specs CI instead of a
# stable release. Useful when testing against unreleased spec changes.
# Requires GITHUB_TOKEN in the env or a repo-root .env file.
pnpm download-spec-tests latest                                # latest scheduled master run
pnpm download-spec-tests 2026-04-14                            # latest successful run on that date
pnpm download-spec-tests latest <owner>/consensus-specs        # fork
pnpm download-spec-tests latest <owner>/consensus-specs <ref>  # fork + branch

# Run e2e tests (requires docker environment)
./scripts/run_e2e_env.sh start
pnpm test:e2e
```

**Tip:** For faster iteration, run tests from the specific package directory:

```bash
cd packages/beacon-node
pnpm vitest run test/unit/chain/validation/block.test.ts
```

## Code style

Lodestar uses [Biome](https://biomejs.dev/) for linting and formatting.

### General conventions

- **ES modules**: All code uses ES module syntax (`import`/`export`)
- **Naming**: `camelCase` for functions/variables, `PascalCase` for classes,
  `UPPER_SNAKE_CASE` for constants
- **Quotes**: Use double quotes (`"`) not single quotes
- **Types**: Prefer explicit types on public APIs and complex functions
- **No `any` or `as any`**: Do not use `any` type or `as any` assertions to bypass
  the type system. In production code, find the proper type or interface. In test code,
  use public APIs rather than accessing private fields via `as any`. If genuinely
  unavoidable, add a suppression with the full rule ID and justification:
  `// biome-ignore lint/suspicious/noExplicitAny: <reason>`
- **Private fields**: No underscore prefix (use `private dirty`, not `private _dirty`)
- **Named exports only**: No default exports

### Import organization

Imports are auto-sorted by Biome in this order:

1. Node.js/Bun built-ins

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChainSafe/lodestar](https://github.com/ChainSafe/lodestar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
