---
trigger: always_on
description: K-based formal semantics of the EVM, plus the `kevm-pyk` Python toolchain that drives compilation, execution, and proof.
---

# evm-semantics

K-based formal semantics of the EVM, plus the `kevm-pyk` Python toolchain that drives compilation, execution, and proof.

## Repository layout

```
kevm-pyk/src/kevm_pyk/kproj/evm-semantics/   K source (*.md, *.k) — the semantics
kevm-pyk/src/kevm_pyk/                        Python toolchain (kevm_pyk package)
kevm-pyk/src/tests/                           Python test suite
tests/specs/                                  K proof specs, organised by suite
```

## Build toolchain

All Python commands run through `uv` from the repo root or inside `kevm-pyk/`:

```bash
# from repo root:
uv --project kevm-pyk/ run <command>

# equivalent, from inside kevm-pyk/:
uv run <command>
```

Build the Python package (creates wheel/sdist, installs into the local venv):

```bash
make -C kevm-pyk/ build
# equivalent:
uv --project kevm-pyk/ build
```

Before running any proof, the K definitions must be compiled into kdist targets.
These are cached under `~/.cache/kdist-<hash>/`; the hash changes whenever any `.k`/`.md`
file in `kproj/` changes, so a rebuild is required after every semantics edit.

```bash
# via the kevm-kdist CLI (recommended):
uv --project kevm-pyk/ run kevm-kdist build evm-semantics.plugin
uv --project kevm-pyk/ run kevm-kdist build evm-semantics.haskell

# parallel build of all targets:
uv --project kevm-pyk/ run kevm-kdist build --jobs 8
```

The kdist targets compile against the installed K (and its haskell-backend), managed by `kup`.
To install a specific K release, or to override just the haskell-backend with a local checkout or a remote branch/commit (e.g. to test a backend change under review), see [`docs/2026-05-22-kup-override.md`](docs/2026-05-22-kup-override.md); rebuild the kdist targets afterward.

## Linting and static analysis (Python)

```bash
make -C kevm-pyk/ check          # all checks: flake8, mypy, autoflake, isort, black
make -C kevm-pyk/ format         # auto-fix:   autoflake, isort, black

# individual tools via uv:
uv --project kevm-pyk/ run flake8 src
uv --project kevm-pyk/ run mypy   src
uv --project kevm-pyk/ run black  --check src
uv --project kevm-pyk/ run isort  --check src
```

**Every commit must pass `make -C kevm-pyk/ check` and `make -C kevm-pyk/ test-unit`.**

## Unit tests (fast, no K runtime required)

```bash
make -C kevm-pyk/ test-unit
# equivalent:
uv --project kevm-pyk/ run pytest src/tests/unit --numprocesses=8 --dist=worksteal
```

These test pure Python logic (AST helpers, KEVM class methods, etc.) and finish in seconds.

## Integration / proof tests

### Make targets (recommended)

```bash
make -C kevm-pyk/ test-integration PYTEST_ARGS+="-k <filter>"
```

The Makefile defaults to `--numprocesses=8 --dist=worksteal --maxfail=1`.
Override parallelism or failure tolerance:

```bash
make -C kevm-pyk/ test-integration PYTEST_PARALLEL=4 PYTEST_ARGS+="-k test_prove_functional"
make -C kevm-pyk/ test-integration PYTEST_MAXFAIL=10 PYTEST_ARGS+="-k test_prove_functional"
```

### Direct pytest (same options, more control)

```bash
uv --project kevm-pyk/ run pytest src/tests/integration \
    --numprocesses=8 --dist=worksteal \
    -k "test_prove_functional"

# run a single named spec:
uv --project kevm-pyk/ run pytest src/tests/integration \
    --numprocesses=1 \
    -k "test_prove_functional and compute-valid-jump-dests"
```

### Proof test suites

| Make target (from repo root) | pytest `-k` filter | Spec directory | Notes |
|---|---|---|---|
| `make test-prove-functional` | `test_prove_functional` | `tests/specs/functional/` | Pure-function/lemma proofs; fast per spec |
| `make test-prove-rules` | `test_prove_rules` | `tests/specs/{benchmarks,erc20,examples,mcd,mcd-structured,kontrol}/` | Full EVM contract proofs |
| `make test-prove-optimizations` | `test_prove_optimizations` | `tests/specs/opcodes/` | EVM opcode optimisation proofs |
| `make test-prove-dss` | `test_prove_dss` | `tests/specs/mcd/vat*.k` | DSS/MakerDAO VAT proofs (very slow, CI only) |
| `make test-conformance` | `test_conformance` | — | Ethereum execution-spec conformance |

Top-level convenience targets delegate to `make -C kevm-pyk/ test-integration PYTEST_ARGS+=...`.

### Running the prover directly (for timing / debugging a single spec)

Useful when you want wall-time measurements or verbose RPC logs without pytest overhead.
The definition must be kompiled first (≈40 s), then reused across runs:

```bash
uv --project kevm-pyk/ run kevm kompile-spec \
    tests/specs/functional/my-spec.k \
    --main-module VERIFICATION --syntax-module VERIFICATION \
    --output-definition /tmp/my-spec-kompiled \
    --backend haskell \
    -I kevm-pyk/src/kevm_pyk/kproj/evm-semantics \
    -I kevm-pyk/src/kevm_pyk/kproj/plugin

time uv --project kevm-pyk/ run kevm prove \
    tests/specs/functional/my-spec.k \
    --definition /tmp/my-spec-kompiled \
    --spec-module MY-SPEC \
    --use-booster \
    --save-directory /tmp/my-spec-save \
    -I kevm-pyk/src/kevm_pyk/kproj/evm-semantics \
    -I kevm-pyk/src/kevm_pyk/kproj/plugin \
    --verbose
```

`--verbose` shows per-RPC-call timing (simplify / execute / implies).
`--save-directory` persists the proof KCFG; omit `--reinit` on reruns to resume.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runtimeverification/evm-semantics](https://github.com/runtimeverification/evm-semantics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
