---
trigger: always_on
description: - Don’t change any code or run any non-read-only-commands without informing me and discussing it first.
---

## Guidelines

- Don’t change any code or run any non-read-only-commands without informing me and discussing it first.
- Be extremely methodical, performing one step at a time and ensuring it works properly before continuing.
- Commit and push cohesive progress once the scope is understood; the user prefers a clean baseline over dangling local files.
- Never commit unrelated changes silently. Summarize what was committed/pushed and keep commits intentional.

## Project Tracking (Required)

- `SCRATCHBOOK.md` is retired. Do not recreate it or use append-only per-turn session memory.
- Keep durable project state in focused docs instead:
  - `docs/RECOVERY_TO_GAMEPLAY.md` — live gameplay-recovery tracker. Update it whenever recovery work changes the baseline, hypotheses, experiments, milestones, validation status, or next actions.
  - `FUNCTION_MAP.md` — identified functions and naming notes.
  - `docs/controller-pak-approaches.md` — Controller Pak/PFS-specific notes.
- Update focused docs only when durable project knowledge changes, not on every interaction.
- Keep tracking entries concise, factual, and actionable:
  - Current baseline and validation status.
  - What was attempted, what worked, and what failed/regressed.
  - Current hypotheses, safe/unsafe patterns, and next recommended steps.
- Prefer introducing risky or experimental behavior behind compile-time feature flags (`#ifdef`/macros).
- Default policy for experiments: flag-gated, off by default, easy rollback path, avoid direct regression risk.

## Reverse Engineering Guidelines (Required)

### When to change recompiled / external code
- Only change recompiled code or external libs if:
  1. Adding logs (fprintf etc)
  2. It is a temporary hack to test something
  3. It is absolutely necessary and will be required for the final code to work

### Symbol naming workflow
- `castlevania2.syms.toml` is the **single source of truth** for function names.
- `symbol_addrs.txt` is secondary (feeds splat for asm output). Keep in sync manually.
- When adding/renaming functions, update `castlevania2.syms.toml` then run `./tools/regen_recomp.sh`.
- The regen script: saves patches, runs N64Recomp, fixes truncation bugs, re-applies hand-written patches.
- CV64 decomp (`../references/cv64_decomp/`) is the primary reference for shared engine function names.
- `tools/match_cv64.py` matches CV64 functions to LoD by instruction fingerprinting (requires both ROMs).
- **ALWAYS update function names whenever progress is made.** Every debugging session, code trace, or investigation that identifies a `func_XXXXXXXX` must result in a rename. Do not leave identified functions unnamed — name them immediately in `castlevania2.syms.toml` and `symbol_addrs.txt`.

### Hand-written patches
- Patches live in `tools/apply_patches.py`, keyed by MIPS VRAM address (never changes across regens).
- Mark patches with `// --- PATCH: description ---` and `// --- END PATCH ---` markers.
- To add a new patch: add a `@patch("description")` function in `apply_patches.py`.
- `patches/recompiled_funcs.patch` is a backup diff — the real patches are in `apply_patches.py`.

### RecompiledFuncs regeneration (CRITICAL)
- **NEVER run N64Recomp by itself without the fixer chain.** On macOS/Linux always use `./tools/regen_recomp.sh`. It verifies binary integrity, backs up before overwriting, fixes truncation bugs, re-applies patches, and fixes cross-function gotos. On Windows, follow the step-by-step equivalent documented in README "Regenerating RecompiledFuncs on Windows" — run N64Recomp **and then the full post-processing chain in order**; skipping the fixers produces broken output.
- **NEVER modify N64Recomp submodule source** (`lib/N64ModernRuntime/N64Recomp/src/`). On macOS the known-good binary is at `../lod_recomp_failed/tools/N64Recomp/build/N64Recomp` and `regen_recomp.sh` verifies its SHA-256 hash before use. On Windows that binary does not exist; building `N64RecompCLI` from the **clean, pinned** submodule (out-of-tree, e.g. `build-n64recomp/` at the repo root) is the validated alternative — its output was verified byte-identical to the macOS generation. Always confirm the submodule is clean (`git -C lib/N64ModernRuntime/N64Recomp status`) before regenerating.
- **NEVER create build directories inside the N64Recomp submodule.** This makes the submodule appear dirty and risks using a modified binary. Build out-of-tree at the repo root instead.
- `RecompiledFuncs/` is gitignored — if corrupted, there is no git revert. The only recovery is re-running the regen pipeline from a clean state.
- Past incident: a locally-modified N64Recomp binary (turning a fatal error into a warning) was used to regenerate, producing corrupted overlay tables that caused `Failed to find function at 0x3C198000` at runtime.

### N64Recomp truncation bug
- N64Recomp crashes on the overlay_system section, truncating `funcs.h`, `recomp_overlays.inl`, and the last `.c` file being written.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fliperama86/cvlod_recomp](https://github.com/fliperama86/cvlod_recomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
