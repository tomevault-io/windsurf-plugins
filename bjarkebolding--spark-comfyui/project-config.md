---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

This file orients Claude Code (or any contributor) to the project so work can
continue without re-deriving the history. Read it fully before editing.

Style rule for every markdown file in this repo, including release notes:
plain and direct. No dash punctuation in prose (no em or en dashes), no
rhetorical flourishes, short sentences. Hyphens in compound words, flags,
dates and list markers are fine. Console captures stay verbatim even when
program output contains dashes.

## What this is

`spark-comfyui` is a single-entry-point bash tool that installs, runs,
updates, and maintains [ComfyUI](https://github.com/Comfy-Org/ComfyUI) on the
**NVIDIA DGX Spark (GB10 Grace Blackwell)**. The Spark is unusual hardware:
aarch64 Grace CPU, an sm_121 Blackwell GPU most toolchains don't target yet,
and 128 GB of unified CPU/GPU memory. A generic ComfyUI install either fails
or runs in silently degraded states. This tool makes the whole lifecycle
automatic and self-healing.

Author/owner: GitHub `bjarkebolding`. Target repo name: `spark-comfyui`.
Hardware in use: DGX Spark, hostname `sparky`, install root `~/comfyui-spark/`.
Published: https://github.com/bjarkebolding/spark-comfyui.
Current version: **2026.07.16.1** (MIT licensed, shellcheck-clean).

## Versioning and releasing

CalVer as of 2026-07-13: `YYYY.MM.DD`, plus `.N` for a second
behavior-changing release the same day. Semver was dropped because push
cadence made it meaningless (pushing to main IS releasing). A version's only
job is to stamp which behavior state a bug report ran. The 1.4.0 to 2026.x
transition sorts correctly under `sort -V`, and `doctor`'s update probe is
git-ancestry-based, so the format is cosmetic to tooling.

**Self-update pulls main HEAD, so pushing to main IS releasing. Always bump
VERSION in the same push.** Docs-only pushes need no bump.

## Golden rules (do not regress these)

1. **shellcheck-clean is non-negotiable.** Run `shellcheck -S warning
   spark-comfyui.sh mods/*/run.sh mods/_lib/mod_common.sh` before every commit.
   Mod `run.sh` files are sourced fragments and carry `# shellcheck shell=bash`.
2. **The main script must stay relocatable.** All paths derive from
   `BASE_DIR="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"`. Never hardcode
   `$HOME` or absolute install paths (the only legit `$HOME` uses are the
   systemd user unit dir and `loginctl`). `mods/` must sit next to the script.
3. **Every optimization is backed by a functional gate, not a heuristic.**
   Warnings guide; a live test decides. Example: SageAttention is only enabled
   after a real multi-shape GPU kernel run passes, never on the strength of a
   version string or a `--help` grep. Two prior bugs came from trusting
   heuristics: `TORCH_CUDA_ARCH_LIST="12.0"` and grepping `ptxas --help` for
   `sm_121`.
4. **Idempotent and self-healing.** `install` and `update` are safe to re-run;
   they skip or refresh, never duplicate or break. Anything that patches the
   ComfyUI tree must re-apply cleanly after `git pull`.
5. **Fail loud on real problems, quiet when healthy.** No silent degradation,
   and no false alarms: benign platform noise (e.g. the aarch64
   `nvidia-*-cu13 is not supported on this platform` line from `pip check`) is
   filtered, not surfaced as a failure. This rule governs the `--watch`
   dashboard too: rows render only when they carry information (see the watch
   section below).
6. **Test patches against fixtures before shipping.** Source patches edit
   upstream Python; always dry-run the transform on a realistic fixture and
   confirm the result still `ast.parse`s.

## Repository layout

```
spark-comfyui.sh          # The entry point: install/run/update lifecycle +
                          # dispatch, ~1700 lines. GB10 venv-package logic
                          # (torch, SageAttention, onnx) lives in
                          # mods/_lib/mod_common.sh, not here.
mods/                     # discovered, applied and self-healed automatically
  _lib/mod_common.sh      # shared helpers: py_patch_file, py_marker_present,
                          # mod_export, and the GB10 venv-package functions
                          # (need_nvcc, sage_kernel_ok, onnx_gpu_ok,
                          # ensure_onnx_gpu, ensure_setuptools_compat,
                          # repair_torch, build_and_verify_sage). Sourced by
                          # the main script itself at startup, not just mods.
  05-setuptools-compat/   # setuptools pinned within torch's own constraint
    run.sh
  10-unified-memory-free/ # get_free_memory() -> host-available unified pool
    run.sh transform.py
  20-torch-repair/        # torch CUDA verified/repaired; also mod_prerun
    run.sh                # (runs before every `run`, not just install/update)
  30-manager-config/      # ComfyUI-Manager config.ini (network_mode, uv,
                          # downgrade_blacklist). NOT pip_auto_fix.list, see
                          # configure.py's docstring (crashed Manager's own
                          # version parser on every launch, retired 2026-07).
    run.sh configure.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bjarkebolding/spark-comfyui](https://github.com/bjarkebolding/spark-comfyui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
