---
trigger: always_on
description: A symbolic execution engine for the [K Framework](https://github.com/runtimeverification/k).
---

# haskell-backend — Developer Guide for Claude

## What This Is

A symbolic execution engine for the [K Framework](https://github.com/runtimeverification/k).
Programs in K-specified languages are compiled to `Kore` (an intermediate representation),
and this backend executes/proves them symbolically via a JSON-RPC server.

The primary binary is `kore-rpc-booster` — a two-engine proxy that routes requests
through a fast engine (Booster) first and falls back to the old engine (Kore) when needed.

---

## Repository Layout

```
kore/               Old Haskell backend — complete but slow; three entrypoints:
                      kore-exec (deprecated), kore-repl, kore-rpc (legacy)
booster/            Fast rewrite engine + proxy server
  library/Booster/  Core library: Pattern/, SMT/, LLVM/, JsonRpc/, …
  tools/booster/    kore-rpc-booster server (Proxy.hs is the key file)
  tools/rpc-client/ kore-rpc-client (testing/scripted RPC calls)
  unit-tests/       Tasty unit tests (tasty-discover)
  test/             Integration tests: llvm-integration/, rpc-integration/, …
kore-rpc-types/     Shared JSON-RPC type definitions
dev-tools/          Utilities: kore-parser, pretty, count-aborts, process-logs, …
test/               70+ make-driven integration test directories (require K framework)
docs/               Architecture docs; 2024-10-18-booster-description.md is essential
scripts/            Performance and analysis scripts
deps/k_release      Pins the required K framework version
```

---

## Building

```sh
# Standard build (with optimisations); --copy-bins puts executables on $PATH
stack build --test --copy-bins all
# or
cabal build all

# Fast dev build (no optimisations — slower runtime, faster compile)
stack build --fast all
cabal build -O0 all

# Via Nix dev shell
nix develop .#cabal --command cabal build all
```

`stack.yaml` is the authoritative package resolver (LTS 22.23, GHC 9.6.5).
`cabal.project.freeze` pins versions for reproducible non-Nix builds.
Run `hpack` inside `booster/` if you edit `package.yaml` before cabal sees changes.

---

## Linting and Formatting

The CI enforces both — fix them before pushing.
Both Fourmolu **and** HLint must pass on **every commit**, not just the final one.

```sh
# Format all Haskell files (fourmolu)
nix develop .#style --command scripts/fourmolu.sh

# Format a single file interactively
nix develop .#style
fourmolu path/to/Foo.hs

# HLint (configured via .hlint.yaml)
hlint .
```

Config files: `fourmolu.yaml` (100-col limit, 4-space indent, leading commas),
`.hlint.yaml` (many suppressions — read before adding ignores).

---

## Running Tests (Shortest to Longest)

### 1. Booster unit tests (~5–30 s)
```sh
stack test hs-backend-booster:unit-tests
# or
cabal test unit-tests
```
Pure Haskell, no external dependencies.  Source: `booster/unit-tests/`.

### 2. Kore unit tests (~1–5 min)
```sh
stack test kore
# or
cabal test kore-test
```
Source: `kore/test/`.  The Makefile target is `make test-kore`.

### 3. LLVM integration tests (~minutes, needs LLVM shared lib)
```sh
cabal test llvm-integration
```

### 4. Booster RPC integration tests (~minutes, needs K binaries)
```sh
# Run the full booster integration test suite:
scripts/booster-integration-tests.sh

# Or run a single directory from inside the integration nix shell (see below):
cd booster/test/rpc-integration && ./runDirectoryTest.sh test-<name>
```

### 5. Full K integration tests (~many minutes, needs K framework)
```sh
# First, enter the integration nix shell:
nix develop \
  github:runtimeverification/k/v$(cat deps/k_release)#kore-integration-tests \
  --override-input haskell-backend . --update-input haskell-backend

# Then:
make test-k          # all integration tests
make test-k-simplifierx  # simplifier-specific subset
make -C test/issue-3344 test   # single test
```

---

## Key Architecture: The Two-Engine Proxy

`kore-rpc-booster` runs **two symbolic execution engines in the same process**:

| Engine | Code | LLVM loaded? | Speed |
|--------|------|-------------|-------|
| **Booster** | `booster/library/Booster/` | Yes | Fast |
| **Kore** | `kore/src/Kore/` | No | Slow but complete |

The proxy (`booster/tools/booster/Proxy.hs`) routes each RPC call:

- **`execute`**: Booster runs first. Falls back to Kore (1 step at a time) only when
  Booster returns a reason in `fallbackReasons` (default: `Aborted | Stuck | Branching`).
  Configurable via `--fallback-on`.
- **`simplify`**: Booster runs, then Kore **always** runs on Booster's output
  (see `handleSimplify`, lines 171–214). This is unconditional — even when Booster
  has fully simplified the term, Kore is still invoked.
- **`implies`**: Booster first if `assumeDefined=true`, else delegates to Kore.
- **`get-model`**: Booster first; re-checks with Kore only when result is `Unknown`.
- **`add-module`**: Both engines, always.

The `ProxyConfig` record (lines 66–74) controls proxy behaviour:
```haskell
data ProxyConfig = ProxyConfig
  { forceFallback        :: Maybe Depth   -- force Kore simplification every N steps
  , fallbackReasons      :: [HaltReason]  -- which halt reasons trigger fallback
  , simplifyAtEnd        :: Bool          -- simplify after each execute result

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runtimeverification/haskell-backend](https://github.com/runtimeverification/haskell-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
