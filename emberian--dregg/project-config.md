---
trigger: always_on
description: *(For any agent — Claude, Codex, grok-build, whoever. The single most common way
---

# AGENTS.md — how to test (and build) dregg without melting your session

*(For any agent — Claude, Codex, grok-build, whoever. The single most common way
to waste an hour here is running the whole test gauntlet in debug mode. Don't.
Read this first. Deeper state lives in `HORIZONLOG.md`; this is just "how do I run
things.")*

## Orient before you answer — the durable record, not the summary

The other expensive mistake (beside the debug-mode gauntlet): in a fresh or
post-compaction window, answering a *vision-level* or "how's X?" question from the
**compaction summary + a shallow shell probe**, instead of reading the durable
record. It flattens this (deeply layered, mature) project into a wrong one-liner —
which then gets written back as if it were authoritative.

**Read order — at window-start, and before any vision-level claim:**
1. `HORIZONLOG.md` — the live named-follow-up burn-down, newest entry first. This is
   the continuity anchor.
2. the memory index whole, then its flagged (⚑) topic files.
3. `docs/OVERVIEW.md` for shape.
4. the relevant `docs/` + the **captured artifacts** for the question.

⚠ **There is no `REORIENT.md`.** Three lines of this file pointed at it until
2026-07-26 — it was deleted and the pointers were not, so the very first instruction a
fresh agent read named a file that does not exist. An orientation step that silently
resolves to nothing is worse than no orientation step: it is spent as if it happened.
If you are looking for it, you want `HORIZONLOG.md` plus the memory index.

**Probes lie.** `which microkit` ("not found") + `rustup target list` ("no sel4
target") once "proved" the seL4 toolchain absent — while the Robigalia v0 demo
BOOTS in QEMU (the SDK is at `~/sel4-sdk`, the target is a `-Z build-std` JSON
spec, boot logs are at `/tmp/sel4-boot-*.log`). Absent-on-PATH ≠ absent; a probe ≠
a doc. Read the doc + the captured artifact (boot/test logs), and verify a dated
memory's `file:line` against HEAD before asserting it. The summary is a map drawn
from a moving train; the record is the territory — and the reading IS the job.

## Two traps grep + memory hide

- **`grep -c "sorry\|admit"` over `metatheory/` lies** — `admit` ⊂ `admitGuard`/`admits`,
  `sorry` ⊂ "sorry-free" docstrings. Proof-completeness = `lake env lean
  metatheory/Dregg2/Claims.lean` + the `Verify/*Lint.lean` gates. If you grep, `-nw` and
  read each hit.
- **A memory/doc note about STATE is a hypothesis — verify at HEAD**, especially
  pessimistic ones ("X is undone / a hole"). Truth = `CLAIMS.md`, the `Dregg2.lean`
  annotations, `lake … Claims.lean` — not a memory's mood. A named residual ≠ a hole.

## Use `cargo nextest`, not bare `cargo test`

`cargo-nextest` is installed. It gives per-test timing, parallelism, and — the
important part — **profiles that keep the slow proof tests out of your way**. The
config is `.config/nextest.toml` (the source of truth for the exact profile names
+ filters; read it).

```
cargo nextest run -p <crate>                  # one crate, fast — your default reflex
cargo nextest run -p <crate> -E 'test(/name/)'# a filter expression (one test/pattern)
cargo nextest run                             # the DEFAULT profile = the FAST set
cargo nextest run --profile heavy --release   # the SLOW set, ON DEMAND (see below)
cargo nextest list -p <crate>                 # list tests without running (validate filters)
```

- **The `default` profile is the fast gauntlet.** The >60s tests (the proof /
  recursion / IVC-fold / dispute-timeout suites) are EXCLUDED from it via a
  `default-filter`. So `cargo nextest run` is the everyday green check.
- **The `heavy` profile is the slow set, run on demand only** (CI-full / pre-release /
  when you specifically touched the prover). It is NOT in the normal loop. Run it in
  `--release` — these are proof-heavy and **debug mode is the main reason they crawl**
  (the IVC recursion fold is *minutes* in debug). The wrapper is
  `scripts/test-gauntlet.sh heavy-release` (also `default | ci | full | list-heavy`);
  full detail — the profile table + the heavy-set list — is in `.docs-history-noclaude/TESTING.md`.
  Offload it to the build node: `scripts/pbuild test scripts/test-gauntlet.sh heavy-release`.
- A few tests are `#[ignore]`'d outright (e.g. `t3_ivc_root_k2/k3`, "recursion fold
  is slow (minutes)") — run those with `--run-ignored`/`--ignored` only when needed.

## Where to build — TWO SPARE BOXES, and read the VERDICT not the exit code

Full doctrine, with the measurements: **`docs/BUILD-BUDGET.md`**. The short version:

- **Both persvati and hbox are spare capacity for our work.** hbox was deliberately
  freed on 2026-07-25: `lean-seed.yml` was the only workflow with a self-hosted
  `runs-on`, it fired on every `metatheory/**` commit at 19-25 min of full
  saturation, and it is now `workflow_dispatch` ONLY. hbox load fell 21.72 → 0.08.
  Nothing else in `.github/workflows` can land there. Do not treat hbox as busy.
- **Rust (cargo)** — `scripts/pbuild <lane> cargo nextest run -p <crate> …`. Per-lane
  isolated, rsyncs your WIP. Never a full local `--workspace` build to "check one
  thing": there is ONE target lock, observed held 45+ min with cargos queued behind

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emberian/dregg](https://github.com/emberian/dregg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
