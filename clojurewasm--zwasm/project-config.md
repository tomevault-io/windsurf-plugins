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
  (3-host SSH fan-out) is an **optional** pre-PR pre-flight mirroring CI
  (ADR-0076 D9) — CI's `ci-required` is authoritative. `--force`
  always forbidden. Root is kept lean (ADR-mirroring the CW layout): this file
  is `.claude/CLAUDE.md`; community-health files (CONTRIBUTING / CODE_OF_CONDUCT /
  SECURITY) are in `.github/`; `THIRD_PARTY.md` is in `legal/`; `examples/` is
  under `docs/examples/`. Only README / LICENSE / CHANGELOG / build+flake files
  remain at root.
- **Release stays user-only (ADR-0156)**: tag / publish / cutover are
  manual. Current line = `v2.0.0-rc.1` (tag-only; Latest = v1.11.1).
  `v2.0.0` final = bump `build.zig.zon` + push `v2.0.0` tag → `release.yml`
  auto-builds + Release + Latest→v2. See `docs/migration_v1_to_v2.md`.
- v1 ABI compatibility is out of scope; the C/Zig/CLI surfaces broke v1 on
  purpose (ADR-0156).

Read-only reference clones: `~/Documents/OSS/` + `zwasm/` (v1) +
`ClojureWasmFromScratch/`. Full list at
[`.dev/reference_clones.md`](../.dev/reference_clones.md); mirrored in
`additionalDirectories` setting. Never edit or commit from these
paths. Pre-redesign investigation: `~/zwasm/private/v2-investigation/`.

## Language policy

Public project. **English by default** for code, comments, identifiers,
commit messages, README, ROADMAP, ADRs, `.dev/`, `.claude/`, all config.
**Japanese** for chat replies only. Enforced by
[`.claude/output_styles/japanese.md`](output_styles/japanese.md)
+ SessionStart hook.

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
  is PR-only, and CI's `ci-required` runs `scripts/ci_gate.sh` (zig fmt +
  `test-all` + extended: lint / build-option DCE / ReleaseSafe JIT smoke / AOT
  cross-compile / **`zone_check`**) on **all 3 OSes** (aarch64-macos +
  x86_64-linux + x86_64-windows) for **every** PR. That IS the merge gate.
  Doc-only PRs auto-skip the heavy legs (still green via `ci-required`). The
  local `scripts/gate_merge.sh` (3-host SSH fan-out) + `scripts/gate_commit.sh`
  (pre-commit) are now **optional pre-PR pre-flight** mirroring CI — no longer
  load-bearing for merge safety. The campaign-era Windows-BATCHED / `--suspend`
  cadence is RETIRED (`should_gate_windows.sh` = deprecation stub; ADR-0174
  superseded-in-part; the `.dev/windows_gate_suspended` sentinel is dead).
  `file_size_check` is **advisory** (ADR-0099 2026-07-03, not a commit block);
  `spill_aware_check` is wired into `gate_commit.sh` + CI `ci_gate.sh` extended
  (D-505 triage done; BASELINE=0). OrbStack retired per ADR-0067 (D-134); scratch only.
- **Context budget**: the **1M** window is in effect (the prior 200K pin
  `CLAUDE_CODE_DISABLE_1M_CONTEXT=1` was removed 2026-05-31 — it made the
  window hit 100% fast and the squeeze, not the working set, was the felt
  pain). The real levers are **structural, not a window cap**: (1) lean
  auto-loaded rules — `.claude/rules/*.md` are injected IN FULL by their

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clojurewasm/zwasm](https://github.com/clojurewasm/zwasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
