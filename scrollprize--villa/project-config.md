---
trigger: always_on
description: This file defines how automated agents (Codex, Claude, chat-based coding agents, CI bots, etc.) should operate inside this monorepo.
---

# AGENTS.md

This file defines how automated agents (Codex, Claude, chat-based coding agents, CI bots, etc.) should operate inside this monorepo.

The repo contains multiple subprojects with different languages, runtimes, and constraints. **Do not assume one “global” build/run workflow applies everywhere.** Instead:

1. **Identify the target subproject(s)** from the user prompt and/or file paths you’re asked to touch.
2. Follow the **Monorepo-wide rules** below.
3. Then apply the matching **Subproject playbook** (e.g., `volume-cartographer/`, `vesuvius/`) only if the prompt targets it.

---

## 1) Monorepo-wide rules

### 1.0 Portal startup policy (important)
- Treat discovery/exploration runs as **read-only** unless the user explicitly asks for environment setup.
- **Do not run installation/bootstrap commands by default** when starting work in this repo.
- Skip side-effect scripts until explicitly requested by the user:
  - `build_dependencies.sh`
  - `install_dependencies.sh`
  - `install_repositories.sh`
  - `setup_user.sh`
  - `setup_sudo.sh`
  - `npm install`, `yarn install`, `pip install`, `poetry install`, `conda env` creation, `uv sync`, Docker build/pull
- If dependencies are needed, report the exact minimal install command per target subproject and ask for confirmation.
- For agent-mode runs (Codex/CI), skip install/bootstrap side effects unless explicitly allowed:
  - Set `AGENTS_AGENT_MODE=1` for that session/run.
  - Then explicitly set `AGENTS_ALLOW_INSTALL=1` to run installs.
- For local/manual usage, no extra env var is required; run installs directly.

### 1.1 Scope first, then act
- Treat each top-level folder as an independent product unless proven otherwise.
- Make the smallest change that solves the requested task.
- If the task spans multiple subprojects, split your work into clearly separated commits/patches.
- Do not execute setup, install, or build scripts in non-target subprojects.

### 1.2 Don’t guess build systems or dependencies
Before changing code:
- Look for **subproject-local** docs and scripts:
  - `README*`, `docs/`, `scripts/`, `Makefile`, `CMakeLists.txt`, `pyproject.toml`, `requirements.txt`, `environment.yml`, `package.json`, `Dockerfile`
- Prefer **existing scripts** over inventing new commands.
- If the target subproject is not explicit, ask the user once for scope before running any install/build/discovery script.

### 1.3 Default to correctness and reproducibility
Unless the prompt explicitly says otherwise:
- Preserve behavior and outputs.
- Avoid nondeterminism (race conditions, unordered iteration affecting results, data-loader shuffles without fixed seeds, etc.).
- Avoid changes that silently relax numerical guarantees, precision, or error bounds.

### 1.4 Performance work must be measured
If the prompt is about performance:
- Establish a baseline.
- Use a profiler appropriate for the platform and language.
- Report before/after results with:
  - command line
  - dataset/input
  - build type
  - iteration counts and summary stats (mean + p50/p95 or min/median/max)

### 1.5 Portability is a hard requirement
The repo targets **Ubuntu and macOS**, across **amd64 and arm64** (where applicable).
- Avoid OS-specific code without guards.
- If adding SIMD/intrinsics, gate it and provide a safe fallback.
- Avoid toolchain-specific flags unless justified and documented.

### 1.6 Keep changes reviewable
- Prefer small, focused diffs.
- Avoid large refactors unless the prompt explicitly requests them.
- If you must refactor, do it in two steps:
  1) mechanical refactor with no behavior change
  2) functional/performance change with measurements

### 1.7 Tests are not optional
- Run the subproject’s tests (or at minimum its smoke/run steps) before claiming success.
- If no tests exist for the touched logic, add a minimal regression test or a lightweight validation harness.

---

## 2) How to decide which playbook to use

Use a subproject playbook when **any** of the following is true:
- The user prompt names the folder (e.g., “work on `volume-cartographer`”).
- The files you’re editing are under that folder.
- You’re asked to run a binary/script that clearly belongs to that folder.

If the prompt is ambiguous:
- Start by mapping the repo structure and identifying candidate entrypoints.
- Propose a plan that separates “discovery” from “changes”.
- Avoid risky changes until scope is clear.

---

## 3) Subproject playbooks

### 3.1 `volume-cartographer/` playbook (activate only when prompted)

**What it is (from repo context):**
- A CPU-based computational geometry / volumetric pipeline project.
- **Language:** C++
- **Build:** CMake
- **Key script:** `volume-cartographer/scripts/build_dependencies.sh` is the source of truth for dependencies.
- **Platforms:** Ubuntu + macOS, amd64 + arm64
- **Current optimization constraint (from prompt context):** focus on speedups **without numeric changes**.

#### Build & test (discover actual entrypoint first)
1) Read and follow:
   - `volume-cartographer/scripts/build_dependencies.sh`
2) Locate the correct CMake entrypoint:
   - repo root `CMakeLists.txt` vs `volume-cartographer/CMakeLists.txt`
3) Prefer:
   - `RelWithDebInfo` for profiling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ScrollPrize/villa](https://github.com/ScrollPrize/villa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
