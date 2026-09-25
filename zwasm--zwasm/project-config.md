---
trigger: always_on
description: A from-scratch WebAssembly runtime in Zig 0.16.0.
---

# zwasm v2

A from-scratch WebAssembly runtime in Zig 0.16.0.

> Pointers only — detailed plans live in [`.dev/ROADMAP.md`](../.dev/ROADMAP.md),
> runnable procedures in [`.claude/skills/`](skills/). (The autonomous
> `/continue` build-campaign loop is RETIRED post-merge — maintenance mode.)

## Identity

**Project name (in all docs and the published artifact): `zwasm`.**
Binary / package: `zwasm`.

zwasm v2 is a ground-up redesign of zwasm (v1 git history at commit 517cc5a).
**As of 2026-07-01 the from-scratch campaign is COMPLETE**: v2 shipped to
`main` (replace-merge `dbd43f89e`); v1 is frozen at tag `v1.11.1`.

- Working dir: `~/Documents/MyProducts/zwasm/` (unified — the separate
  `zwasm_from_scratch/` working dir is retired).
- **`main` is the trunk.** Dev model: cut a `develop/<slug>` branch from
  `main`, PR to `main`. `main` is **server-side ruleset-protected**: no direct
  push, PR required, and the `ci-required` status check (CI's 3-OS gate) must be
  green to merge; only the repo admin can bypass. Doc-only PRs auto-skip the
  heavy gate (still green via `ci-required`). The local `scripts/gate_merge.sh`
  (3-host SSH fan-out) is an **optional** pre-PR pre-flight that runs
  `test-all` per host, not the CI leg — its header names what it leaves out
  (ADR-0076 D9) — CI's `ci-required` is authoritative. `--force`
  always forbidden; `--force-with-lease` only in the pinned form ROADMAP §14
  permits. Root is kept lean (ADR-mirroring the CW layout): this file
  is `.claude/CLAUDE.md`; community-health files (CONTRIBUTING / CODE_OF_CONDUCT /
  SECURITY) are in `.github/`; `THIRD_PARTY.md` is in `legal/`; `examples/` is
  under `docs/examples/`. Only README / LICENSE / CHANGELOG / build+flake files
  remain at root.
- **Release stays user-only (ADR-0156)**: tag / publish / cutover are
  manual. The release line is the latest `v2.x` tag (see CHANGELOG /
  GitHub Releases — do NOT hardcode it here, it rots). Cut = bump
  `build.zig.zon` + CHANGELOG section + push the `vX.Y.Z` tag →
  `release.yml` auto-builds + publishes. See
  [`.dev/archive/migration_v1_to_v2.md`](../.dev/archive/migration_v1_to_v2.md).
- v1 ABI compatibility is out of scope; the C/Zig/CLI surfaces broke v1 on
  purpose (ADR-0156).

Read-only reference clones: `~/Documents/OSS/` (upstream runtimes + specs).
Full list at [`.dev/reference_clones.md`](../.dev/reference_clones.md).
Never edit or commit from these paths.
v1 is not a separate clone — it is tag `v1.11.1` in this repository.

## Language policy

Public project. **English by default** for code, comments, identifiers,
commit messages, README, ROADMAP, ADRs, `.dev/`, `.claude/`, all config.
**Japanese** for chat replies only — set by the `Japanese` output style
([`.claude/output_styles/japanese.md`](output_styles/japanese.md), via
`outputStyle` in `settings.json`). That single setting is sufficient; the
SessionStart hook injects no language directive. To work in another
language, override `outputStyle` per-machine in `settings.local.json`.

**Bilingual exception**: meta-prose pointers ("詳細は <ref> を参照。")
and culturally-loaded one-word labels (例: 気付いたら即追加, 裏取り)
where they anchor a concept more cleanly. Never in normative rule
text or code identifiers.

## Frozen invariants (read once per session)

- **Release is user-only (ADR-0156)**: never autonomously tag, publish, or
  cut over to a release. Tag / publish / version come only from an explicit
  user message. (The v2 build campaign — Phase 16 完成形 — is complete; the
  project is in maintenance. v2 is on `main`; v1 frozen at v1.11.1.)
- **ROADMAP §18 amendment**: routine `[x]` flips + SHA backfills + next
  phase table expansion = no ADR. Deviation in §1 / §2 (P/A) / §4
  (architecture / Zone / ZirOp) / §5 (layout) / §9 phase scope/exit /
  §11 / §14 forbidden list = file `.dev/decisions/NNNN_<slug>.md` per
  §18.2 FIRST. **Carve-out (ADR-0132)**: re-sequencing/re-scoping the
  ROADMAP because a phase's exit/scope references genuinely-later-phase
  work (§18.1 first bullet) is **AUTONOMOUS** — file the ADR + §18.2
  four-step + forward-ref each deferred item to its true phase, and
  proceed without stopping (no user-flip). Default posture =
  autonomous-with-ADR; surface only for bucket-2/3 genuine blocks.
- **CI gate is authoritative (post-v2.0.0 maintenance; ADR-0076 D9)**: `main`
  is PR-only, and CI's `ci-required` runs `scripts/ci_gate.sh` on **all 3 OSes**
  (aarch64-macos + x86_64-linux + x86_64-windows) for **every** PR. That IS the
  merge gate. A PR run gets the **core** gate — zig fmt + `test-all` +
  `bench-latency-build` (compile-only, ADR-0209) + the test-discovery guard,
  plus `run-rust-host` on the Linux leg only and the unit tests built
  ReleaseSafe on the Linux and Windows legs (#347, #303) — and, as a blocking
  step OUTSIDE `ci_gate.sh`,
  `test-wasi-p1-official` (ADR-0225; the only leg content the script does not
  define). The **extended** checks (lint /
  build-option DCE / ReleaseSafe JIT smoke / AOT cross-compile / `zone_check` /
  `spill_aware`) are gated on `ZWASM_CI_EXTENDED`, which `ci.yml` sets only on
  the **push to `main`** — they are up to ~20 cold-cache ReleaseSafe builds and
  would dominate every PR's wall-clock (rationale in `ci.yml`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zwasm/zwasm](https://github.com/zwasm/zwasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
