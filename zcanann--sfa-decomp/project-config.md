---
trigger: always_on
description: > **Active compiler experiment (scope corrected 2026-09-06):** Only game-category
---

# AGENTS.md - SFA-Decomp Runbook

> **Active compiler experiment (scope corrected 2026-09-06):** Only game-category
> MWCC C/C++ units use the common GC/1.3 `config.compiler_version`. The older
> math block, including `rand`, belongs to the `MSL_C/PPCEABI/bare/H/` source
> tree and third-party grouping by user instruction (2026-09-20). Its eleven
> math TUs retain their exact GC/1.2.5n profiles; `rand.c` retains GC/1.3.
> These are independent library settings, not game compiler exceptions.
> MSL ownership is provisional; descriptive filenames do not claim recovered
> vendor source names. See `docs/msl_math_rehome.md` for the path mapping.
> Keep `main/zlb.c` on its ProDG toolchain.
> Dolphin SDK, MSL, MusyX, and compiler runtime retain their pre-migration compiler
> profiles: GC/1.2.5n library defaults where applicable and their existing per-unit
> exceptions. Their inclusion in the global migration was a scope mistake; do not
> move them to GC/1.3 as part of game-code work. Optimization profiles remain;
> the linker independently stays GC/1.3.2. Both `ninja all_source` and the strict
> retail checksum target must pass before pushing. Mark regressed game units
> `NonMatching` so the matching link uses retail objects while `all_source` still
> compiles their C/C++. Do not restore game compiler exceptions, change the
> expected checksum, or disable the check to make the build green.

> **Integration workflow (effective 2026-07-29):** High-frequency decomp commits land on the
> permanent `staging` branch, not directly on `main`. Fetch before starting, check out
> `origin/staging`, and rebase local unpushed work onto the fresh remote staging tip before every
> push. Never resolve a rebase by clobbering with `--theirs`; abort and re-derive on conflict.
>
> A maintainer or bot performs one **normal merge commit** from `staging` to `main` per UTC day.
> This preserves the useful per-change history while causing the main Actions build to run once
> for the batch. After that merge is published, fast-forward `staging` to the new `main` merge
> commit before reopening it for work; do not rotate or date the staging branch. If commits land
> directly on `main`, reconcile them into `staging` without dropping either side before the next
> staging push. Larger coherent PRs remain welcome, but `staging` is the default low-overhead
> landing path.

> **Automation note (resolved 2026-06-18):** JackPriceBurns paused the loop because it
> was re-pushing a stale patch series (cherry-pick with `--theirs` conflict resolution
> reverted other contributors' fixes) and only gated on the 30s strict-hash target, so
> it never saw `all_source` breaks (dup `posPtr`, `fn_802ABFBC` mismatch). Both are
> fixed and `main` is green again. Resumed by zcanann with corrected discipline: every
> push **rebases onto the fresh active remote integration branch**, **never
> `--theirs`-clobbers** (abort + re-derive on conflict), and **gates on
> `ninja all_source` exiting 0** — not just the match target.

Keep this light. The project is still in the "recover the game" phase, not the "polish an already-understood decomp" phase.

## Target
- Active target version in this repo is `EN v1.0` (`GSAE01`).
- `orig/` may contain EN, PAL, and JP artifacts. Use them for cross-checking only.
- `resources/ghidra-decomp-4-12-2026/` contains the raw EN v1.0 Ghidra decomp with no code analysis. Use it for basic code structure only.
- `resources/DolphinSymbolExport_GSAE01.txt` contains the active EN symbol export based on Dolphin Signature analysis. This is good for SDK matching, but the addresses need to be translated against the current `config/GSAE01/symbols.txt` anchors before using them for splits.
- Other `resources/*` content should also be treated as rough shape data, not analysis.
- `reference_projects/*` may exist in some checkouts to mine patterns, especially SDK code and common Nintendo/GameCube layouts. Do not edit anything in this folder, as it is reference code from other game decompilation efforts. Known donors: `marioparty4` (2002, closest-era Nintendo title), `tww` (2002–03 Wind Waker), `melee` (2001, oldest SDK baseline), `mkdd` (2003 Mario Kart DD), `pikmin2` (2004, richest Dolphin SDK coverage — `src/Dolphin/` mirrors our layout), `prime` (2002 Metroid Prime, same year as SFA).
- SDK files have already been added to this project, but they are from another game. We will need to assign our splits to use these files, update symbols, and possibly learn from `reference_projects/*` to drive our SDK files to matching.
- `reference_projects/rena-tools*` may exist in some checkouts with Rena's SFA decomp projects, and may have some information that helps.

## What Progress Looks Like
- Recover real functions, data, class boundaries, globals, vtables, and file structure.
- Improve buildability, linkage, and objdiff results.
- Make the source look more like plausible original code, not a pile of coercion hacks.
- Generate new understanding when the repo has none yet.

This repo starts from very little. Expect to do naming, struct recovery, type cleanup, file splitting, and tool building as part of normal work.

## Ground Truth
- `objdiff` is still the final measure of whether a change helped.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zcanann/SFA-Decomp](https://github.com/zcanann/SFA-Decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
