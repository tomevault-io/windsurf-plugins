---
trigger: always_on
description: This is the canonical entry point for **both human contributors and AI coding agents** working on Vortex. It captures the *rules* — invariants, conventions, and footguns — that every change should respect. Build/test/debug *recipes* live in the topic docs linked below.
---

# AGENTS Guide for Vortex GPGPU Development

This is the canonical entry point for **both human contributors and AI coding agents** working on Vortex. It captures the *rules* — invariants, conventions, and footguns — that every change should respect. Build/test/debug *recipes* live in the topic docs linked below.

---

## 1. Documentation Map

### Setup & build
- [docs/install_vortex.md](docs/install_vortex.md) — initial install + build setup
- [docs/building_toolchain.md](docs/building_toolchain.md) — building Verilator, RISC-V GNU, LLVM, compiler-rt, musl, POCL from source
- [docs/environment_setup.md](docs/environment_setup.md) — environment variables and toolchain layout
- [docs/fpga_setup.md](docs/fpga_setup.md) — FPGA target setup

### Codebase orientation
- [docs/codebase.md](docs/codebase.md) — repo file-tree map
- [docs/microarchitecture.md](docs/microarchitecture.md) — pipeline & cache architecture
- [docs/cache_subsystem.md](docs/cache_subsystem.md) — cache subsystem details
- [docs/hardware_library.md](docs/hardware_library.md) — `hw/rtl/libs/` reference

### Coding conventions
- [docs/coding_guidelines_cpp.md](docs/coding_guidelines_cpp.md) — C++ style
- [docs/coding_guidelines_verilog.md](docs/coding_guidelines_verilog.md) — Verilog/SystemVerilog style

### Simulation & test
- [docs/simulation.md](docs/simulation.md) — driver modes (simx, rtlsim, opae, xrt) and blackbox usage
- [docs/testing.md](docs/testing.md) — test and regression flow
- [docs/debugging.md](docs/debugging.md) — debug traces (`--debug`), VCD, scope, trace_csv
- [docs/debug_mode.md](docs/debug_mode.md) — debug-mode hardware support
- [docs/perfetto_analysis.md](docs/perfetto_analysis.md) — Perfetto trace and analysis
- [docs/synthesis_analysis.md](docs/synthesis_analysis.md) — synthesis/PPA analysis

### Process
- [CONTRIBUTING.md](CONTRIBUTING.md) — public fork/PR contribution flow
- [docs/bug_fixes.md](docs/bug_fixes.md) — bug-fix discipline (root-cause vs patch)
- [docs/continuous_integration.md](docs/continuous_integration.md) — CI pipeline
- [docs/proposals/](docs/proposals/) — design and migration proposals (drafts and in-progress)
- [docs/designs/](docs/designs/) — accepted designs (post-proposal, post-implementation)

---

## 2. Build & Toolchain Rules

See [docs/install_vortex.md](docs/install_vortex.md) for the full recipe. The non-negotiable rules:

- **Out-of-tree.** From the repo root:
  ```bash
  mkdir -p build && cd build
  ../configure --xlen=32 --tooldir=$HOME/tools   # or --xlen=64
  ./ci/toolchain_install.sh                # first time only
  make -s
  ```
- **Separate build dirs per major variant** (`build32/`, `build64/`, `build_fpga64/`, ...) to avoid config/tool contamination. Never reuse one build dir for incompatible configurations.
- **`configure` generates a runnable tree** by copying and instantiating `ci/`, `runtime/`, `sim/`, and `tests/` into `build/`. For execution and test automation, *always* prefer the generated scripts/Makefiles under `build/` over the source-tree `.in` files.
- **Re-`../configure` from `build/`** whenever you `git pull`, edit source Makefiles, edit `VX_config.toml` / any `*.toml`, or add/remove a build-participating directory. Symptom of forgetting this: stale binaries, missing targets, or "I edited this Makefile and nothing happened."
- **Always ensure the build is current before running any test or app** — re-run `../configure` from `build/` first. `configure` regenerates `<build>/sw/VX_config.h` and `<build>/hw/*.vh` from `VX_config.toml`, but only when the toml is newer (it guards on mtime). The simx/RTL **cores `#include` this generated header**, so a stale header makes a core compile against old config values and silently diverge from the toml — and from the runtime/RTL, which re-expand the config every build. This is a real footgun: a stale `VX_config.h` once made SimX run a write-back D-cache while the toml/RTL were write-through, producing SimX-only wrong results. **Never** work around such a divergence by injecting `-DVX_CFG_*` overrides into a Makefile — that masks the stale artifact and fights the config system. The toml is the single source of truth; fix it by re-`configure`-ing.
- **`ccache` can serve stale objects on `fmt`-version mismatches** (typical symptom: `fmt::v8` undefined-reference link errors in sim builds). Before deep-diving, retry with `CCACHE_DISABLE=1`.

---

## 3. Bug-Fix Rules

See [docs/bug_fixes.md](docs/bug_fixes.md) for the full rationale and examples. The rules:

- **Fix root causes, not symptoms.** Diagnose before patching.
- **Don't paper over upstream regressions** or mask bugs with fallback paths and suppressed warnings.
- **Don't add legacy API compatibility or fallback code** unless it is explicitly required for the change.
- **If a patch is genuinely unavoidable** (e.g. blocked by an external dep), label it as a patch explicitly *in the commit message* and pair it with a follow-up to do the proper fix.

---

## 4. Testing & Verification Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vortexgpgpu/vortex](https://github.com/vortexgpgpu/vortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
