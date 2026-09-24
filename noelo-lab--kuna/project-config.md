---
trigger: always_on
description: Guidance for working in this repository. (`CLAUDE.md` and `AGENTS.md` at the repo root are
---

# AGENTS.md

Guidance for working in this repository. (`CLAUDE.md` and `AGENTS.md` at the repo root are
symlinks to this file, `docs/agents.md`.) This file holds only what every task needs;
everything else is one lookup away via the **doc map** at the bottom.

## What kuna is

kuna is an **agent-first decompiler written in Rust**: a decompilation engine plus a SLEIGH
compiler, organized around an explicit phase model whose decision points are exposed as
per-run, flippable options — the LLM control surface is the product. It started as a Rust
port of Ghidra's decompiler (Apache-2.0 — see `LICENSE` and `NOTICE`) and has since
diverged on its own defaults and features; the origin story lives in `docs/history.md`, and
is not needed for day-to-day work.

## Layout

| Path | What |
|---|---|
| `decompiler/` | The engine — a cargo workspace. `kuna-decomp` is the decompiler (`src/` is phase-foldered: `p0_knowledge/`…`p9_emit/`, plus `substrate/` and `infra/`); `kuna-analysis` the loader/analyzer tier (ELF markup, strings, DWARF, function discovery); `kuna-sleigh`/`kuna-slacomp` the SLEIGH runtime/compiler (binary `slacomp`); `kuna-console` the `decomp_dbg`/`decomp_test_dbg` binaries; `kuna-cli` the user-facing `kuna` binary; `kuna-ghidra` and `kuna-wasm` the Ghidra and browser front-ends; `kuna-base`/`kuna-num`/`kuna-harness`/`kuna-lift-diff` support. |
| `tests/datatests/` | Vendored XML regression corpus (83 files / 675 assertions) — what `make test` runs. |
| `tests/stages/` | kuna-owned issue testcases — `make test-stages`. Conventions: `tests/stages/README.md`. |
| `tests/golden/` | Differential golden vectors for the workspace suite (`make rust-test`). |
| `specs/Ghidra/Processors/` | Vendored SLEIGH specs. `.sla` files are built artifacts (gitignored), produced by `slacomp`. |
| `scripts/` + `tools/pipeline/` | Python helpers (`decompile.py` library shim, `paths.py`, `pipeline/`, `decbench/`) + driver for the improvement pipeline (`docs/improvement-pipeline.md`) and the decbench campaign (`docs/decbench-loop.md`). |
| `scripts/repipe/` + `tools/repipe/` | The RE-friction loop (`docs/re-pipeline.md`): Codex Sol-low testers reverse-engineer crackmes with kuna and record where it fails them; Codex Sol-high-or-above builders close those gaps and self-merge. Durable backlog in `docs/re-needs/`; promoted regression probes in `tests/cli/`. |
| `integrations/` | Front-ends embedding the engine: `ghidra/` (kuna as stock Ghidra's decompiler core), `web/` (the project site + in-browser decompiler at `kuna.noelo.org`). |

## Build & test

Only prerequisite: a Rust toolchain. Develop in the workspace directly
(`cd decompiler && cargo build/test ...`); the Makefile is the driver:

```bash
make            # binaries + specs
make binaries   # decomp_dbg / decomp_test_dbg / slacomp / kuna  → decompiler/target/release/
make specs      # compile all .slaspec → .sla with slacomp
```

**Four gates — run all of them before every commit:**

| Gate | Checks | Expect |
|---|---|---|
| `make test` | datatest corpus vs `docs/baseline.json` | **PARITY OK** (675/675) |
| `make test-stages` | stage-issue corpus vs `docs/baseline-stages.json` | **PARITY OK** |
| `make rust-test` | full cargo workspace suite + `docs/options.md` freshness | green |
| `make check-spec` | `docs/spec/` anchors + inline code paths resolve; each phase folder owned by exactly one chapter (`--strict` adds option-mention coverage) | green |

CI runs all four (plus `kuna catalog --check`) on every push to main —
`.github/workflows/tests.yml`. On a **pull request from a branch in this repo** the
workspace suite is skipped and only the parity gates run; **you are the gate for
`make rust-test` on those PRs**, which is why it is on the list above. To demand it from
CI on a particular PR, add the **`full-ci`** label — that label is itself a trigger, so
the suite starts on the label alone. (It also always runs pre-merge on a fork PR, and via
*Run workflow*.) Run all four locally regardless: the workspace suite is the long pole in
CI, so local failures are found far sooner.

- **Never re-pin `docs/baseline.json` to absorb a regression** — fix the code or make the
  change opt-in. The only sanctioned re-pins are an intentional upstream sync or a
  deliberate default change, and the commit message says which (`kuna test --save-baseline`).
  Adding a stage test DOES re-record the stages baseline:
  `kuna test --datatests --datatests-dir tests/stages --save-baseline docs/baseline-stages.json`.
- `docs/options.md` is generated — after touching option metadata:
  `decompiler/target/release/kuna catalog --markdown > docs/options.md`.

## The `kuna` CLI

The user-facing binary (`decompiler/crates/kuna-cli` → `decompiler/target/release/kuna`).
The commands agents use most:

```bash
kuna docs                                          # the embedded manual — cli, options, phases, modes
kuna install-skill                                 # install the embedded agent skill (skills/kuna/SKILL.md)
kuna decompile ./a.out main [--json]               # one function (or an address with --addr)
kuna xrefs ./a.out --to 0x401030 --json            # what references this; --from for the reverse

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noelo-Lab/kuna](https://github.com/Noelo-Lab/kuna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
