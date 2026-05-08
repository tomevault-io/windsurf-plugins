---
trigger: always_on
description: - **This is a purely open source project.** Never mention any company names, internal projects, proprietary codebases, or employer-related information anywhere — not in code, commit messages, PR descriptions, comments, changelogs, or documentation. All references must remain generic and public.
---

# CLAUDE.md

## Important Rules

- **This is a purely open source project.** Never mention any company names, internal projects, proprietary codebases, or employer-related information anywhere — not in code, commit messages, PR descriptions, comments, changelogs, or documentation. All references must remain generic and public.
- **No dead code.** When replacing an implementation, delete the old one entirely. No deprecation, no keeping old code "for backwards compatibility." The only contract that matters is 100% circe compatibility — internal APIs can change freely. Keep the codebase clean and current.

## PR Workflow

Always create PRs from a fresh branch off latest remote main. Never commit directly to `main`.

```bash
git checkout main && git pull          # sync with remote
git checkout -b <branch-name>          # create feature branch
# ... make changes, commit ...
git push -u origin <branch-name>       # push and create PR
```

## Project

Drop-in replacement for circe's auto/semi-auto/configured derivation. Scala 3.8.2+ only. Uses the "sanely-automatic" approach — Scala 3 macros with `Expr.summonIgnoring` to derive all instances in a single macro expansion, avoiding implicit search chains.

**The Contract (non-negotiable)**: 100% behavioral compatibility with circe's derivation, zero compromise. If an application works with circe-generic or circe's configured derivation, switching to circe-sanely-auto or sanely-jsoniter must produce identical JSON output, accept identical JSON input, and yield identical error messages in every edge case. Any deviation — no matter how minor — is a bug that must be fixed before release. This overrides all other priorities including performance. We never compromise compatibility for speed, convenience, or code elegance. Only the implementation changes; the observable behavior is identical. Experimental status does not weaken this contract.

## Build Commands

Mill 1.1.2. Run from repo root:

```bash
./mill sanely.jvm.compile       # compile (JVM)
./mill sanely.js.compile        # compile (Scala.js)
./mill sanely.jvm.test          # unit tests - JVM (135 tests, munit)
./mill sanely.js.test           # unit tests - Scala.js (135 tests, munit)
./mill compat.jvm.test          # circe compat tests - JVM (192 tests, munit + discipline)
./mill compat.js.test           # circe compat tests - Scala.js (192 tests, munit + discipline)
./mill demo.run                 # run demo
./mill tapir-test.test          # Tapir integration tests (8 tests, munit)
```

**Do NOT run** `./mill __.compile` or bare `./mill` — use targeted module commands to avoid cache invalidation.

### Syncing Circe Upstream Tests

Compat tests are auto-generated from circe's upstream test suite via a git submodule + Python script:

```bash
git submodule update --init      # init upstream/circe/ submodule (pinned to a release tag)
python3 scripts/sync-circe-tests.py  # transform & write to compat/test/src/io/circe/generic/
```

Generated files (DO NOT edit manually — regenerate with the script):
- `AutoDerivedSuite.scala` ← `DerivesSuite.scala`
- `SemiautoDerivedSuite.scala` ← `SemiautoDerivationSuite.scala`
- `ConfiguredDerivesSuite.scala` ← `ConfiguredDerivesSuite.scala`
- `ConfiguredEnumDerivesSuites.scala` ← `ConfiguredEnumDerivesSuites.scala`

### Zinc Incremental Compilation Tests

```bash
bash test-zinc.sh               # zinc incremental recompilation tests (5 scenarios, 21 checks)
```

### Benchmarks

Compile-time benchmarks use [hyperfine](https://github.com/sharkdp/hyperfine) (`brew install hyperfine`).

```bash
bash bench.sh 5                 # auto derivation via hyperfine (~300 types)
bash bench.sh --configured 5    # configured derivation via hyperfine (~230 types)
bash bench.sh --jsoniter 5      # marginal cost of sanely-jsoniter (~199 types, circe-only vs circe+jsoniter)
./mill benchmark.sanely.compile # compile benchmark: our library (auto)
./mill benchmark.generic.compile # compile benchmark: circe-generic (auto)
./mill benchmark-configured.sanely.compile   # compile benchmark: our library (configured)
./mill benchmark-configured.generic.compile  # compile benchmark: circe-core (configured)
bash bench-runtime.sh           # runtime benchmark: circe-jawn vs circe+jsoniter vs jsoniter-scala (quick, hand-rolled)
./mill benchmark-runtime.run    # run runtime benchmark directly (hand-rolled harness)
./mill benchmark-jmh.runJmh                          # JMH runtime benchmark (all 4 libraries, read + write)
./mill benchmark-jmh.runJmh 'Read'                   # JMH read benchmarks only
./mill benchmark-jmh.runJmh 'Write'                  # JMH write benchmarks only
./mill benchmark-jmh.runJmh -prof gc                  # JMH with GC allocation profiler
./mill benchmark-jmh.listJmhBenchmarks                # list detected JMH benchmarks
python3 scripts/analyze_jmh.py results/runtime/runtime.txt  # analyze JMH output into summary tables
python3 scripts/summarize_benchmark.py results               # summarize all results into markdown
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nguyenyou/circe-sanely-auto](https://github.com/nguyenyou/circe-sanely-auto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
