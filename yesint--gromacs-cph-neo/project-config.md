---
trigger: always_on
description: Guidance for Claude Code when developing **in this tree**
---

# CLAUDE.md — constant-pH GROMACS 2026 port

Guidance for Claude Code when developing **in this tree**
(`/home/semen/install/gromacs-2026-cph`). This is the development-facing companion
to the other docs already here — read those for the details this file only points to:

| Doc | What it is |
|---|---|
| `PORT_LOG.md` | The running progress log — chronological, per-work-package, every commit/bug/validation. **Append here as work lands.** |
| `README.md` | User-facing description + limitations (what ships to someone who just wants to *run* cph). |
| `GPU_LAMBDA_UPDATE_PLAN.md` | The **current** GPU roadmap (variant B, GPU-resident λ-dynamics). L0–L3 done; L2/L5 remain. |
| `GPU_PORT_PLAN.md` | **Superseded** older "classic NB-offload" GPU sketch. Kept for reference; don't follow it. |
| `_portref/00_fork_vs_2021_all.diff` | The 2021 constant-pH fork's diff **vs pristine 2021.0**, per touched file (`00_stat.txt` = the ~46-file footprint). This is the source-of-truth for *what the fork changed*; the port re-implements each hunk adapted to 2026 APIs. Gitignored. |

## What this tree is

Pristine **GROMACS 2026.1** with the **constant-pH (λ-dynamics)** method of Aho,
Buslaev *et al.* (originally a GROMACS-2021 fork) ported onto it. The point is a
current-GROMACS cph build to replace the aging 2021 fork that runs the
`full_size_1d/r{1,2,3}_cph` umbrella-sampling campaign.

- **git baseline** = commit `b6b339e` ("pristine GROMACS 2026.1 source, pre-cph-port"). `git diff b6b339e HEAD` is the entire port (~77 src files, ~5400 insertions).
- **Branches:** `master` *is* the full CPU+GPU port (L0–L3 GPU-resident work plus the AA 1-4-pair fix). `gpu-lambda-update` was converged with `master` at `514bb20`; it has **not** followed since, so check before doing GPU work on it. Push only when asked.

### Other trees this one depends on / relates to (NOT part of this repo)

- **Fork oracle (2021, built):** `/home/semen/install/constantph/gromacs-constantph/build/bin/gmx` — the reference implementation every numerical validation compares against (single precision, thread-MPI, gcc-14). Confirmed present.
- **aurum2 — ONE tree, `/home/yesylevskyy/install/gromacs-2026-cph`**, a git clone of `origin` tracking `master`, holding **both** builds side by side (CMake is out-of-source, so one checkout serves any number of configurations):
  | build dir | config | wrapper |
  |---|---|---|
  | `build-sys` | CPU, spack `fftw@3.3.10`, AVX2_256, Release | `build_sys.sh` |
  | `build-cuda` | CUDA sm_86, **own FFTW**, AVX2_256, Release | `configure_cuda.sh` + `build_cuda.sh` |

  Wrappers `spack load` and touch `BUILD_DONE`/`BUILD_FAIL`; they are cluster-local (untracked, in `.git/info/exclude`) along with `*_run/`, `traj1k/`, `_portref/`, `cuda_build_logs/`, `*.tpr`. Build dirs are covered by the repo `.gitignore` (`/build*/` — root-anchored, so `src/external/build-fftw/` survives; that is the `af633fa` fix). See [[reference_aurum2_cuda_build]] and "Building & running on aurum2 (GPU)" below.
- **‼️ History (do not recreate):** CUDA used to live in a **separate** tree `gromacs-2026-cph-gpu` that was an **rsync** copy — an rsync target cannot share a directory with a git checkout, which was the *only* reason for a second tree. On 2026-08-06 it was converted to a git clone and then folded into the tree above; the duplicate is deleted. The rsync is retired because its `--exclude 'build-*'` silently dropped 5 tracked files (`admin/ci-scripts/build-and-test-*.sh`, `build-taf-template.sh`, `docs/dev-manual/build-system.rst`) on top of the known `src/external/build-fftw/` and `src/buildinfo.h.cmakein`. A stale `build-cpu` (byte-identical config to `build-sys`, a month old) was removed at the same time. **One checkout = one HEAD = nothing to drift.**
- **Updating the cluster tree** (once the work is pushed): `git fetch origin master && git merge --ff-only FETCH_HEAD`, then run the relevant build wrapper. If nothing has been pushed yet, use a **git bundle** instead (`git bundle create x.bundle <theirHEAD>..master`, scp, `git fetch /path/x.bundle master`) so you don't have to push unasked. ‼️ **Its `origin/master` ref goes stale, so `git rev-list HEAD..origin/master` can report "0 behind" while the tree is actually behind — always `git fetch` first.** It had silently drifted **29 commits** once (pre-L0.1, no CPU PME reciprocal potential) and **3 commits** again on 2026-08-06; a bug report produced with its stale binary cost an extra debugging round. **Check `git log -1` there before trusting any result.**
- **Repro scripts + cph inputs** live in the *project* dir, not here: `/home/semen/work/Projects/Misha/CG/full_size/cph/` (`m0a_repro.sh`, `m0b_repro.sh`, `cph.mdp.inc`, `molecule_0.itp`, index/toppar, etc.). The M1 single-point validation inputs are assembled from `full_size/` + `full_size/cph/`.

## Repository layout — where the cph code lives

The method's own module (new, self-contained):

```
src/gromacs/applied_forces/constant_ph/
  constant_ph.{cpp,h}            # ConstantPH: the λ-dynamics driver (ODE integrator, groups, thermostat)
  read_params.{cpp,h}            # mdp/tpr parameter parsing
  update_topology_charges.{cpp,h}# baking λ→charges into the topology at grompp
  tests/constant_ph.cpp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yesint/gromacs_cph_neo](https://github.com/yesint/gromacs_cph_neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
