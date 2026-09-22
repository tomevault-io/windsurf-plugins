---
trigger: always_on
description: This guide is the offline development entry point for first-time EAI Simulator developers and coding agents. Hosted documentation is optional supporting material, not a prerequisite for using this guide. `README.md` provides project and user orientation, while `.github/CONTRIBUTING.md` owns contribution policy. When documentation and implementation disagree, source code, configuration, and tests are authoritative.
---

# EAI Simulator Agent Development Guide

## 1. Purpose and Scope

This guide is the offline development entry point for first-time EAI Simulator developers and coding agents. Hosted documentation is optional supporting material, not a prerequisite for using this guide. `README.md` provides project and user orientation, while `.github/CONTRIBUTING.md` owns contribution policy. When documentation and implementation disagree, source code, configuration, and tests are authoritative.

All repository paths in this guide are relative to the repository root. All commands assume the repository root as the working directory unless a section explicitly says otherwise.

"Offline guide" means that the development instructions live in the repository. It does not mean installation or asset use is network-free: dependency installation, gated assets, and large models can still require network access and external services.

The internal GitLab view also contains the hosted website source under `docs/`.
That tree is intentionally absent from the public GitHub export. When `docs/`
is absent, do not recreate it in a public pull request and skip the website-only
commands later in this guide; use the published documentation and the GitHub
documentation issue template instead. Repository guides outside `docs/` remain
part of the public contribution surface.

## 2. Non-Negotiable Development Rules

- Run `git status --short` before editing. Preserve unrelated tracked and untracked work, including changes that overlap files you need to inspect.
- Inspect the relevant code, configuration, and tests before making assumptions about behavior.
- Use `rg` and `rg --files` for repository searches when available.
- Use structured parsers and serializers for structured data such as JSON, YAML, TOML, and USD metadata; do not rely on ad hoc text manipulation.
- Keep each change focused on the requested behavior. Avoid unrelated refactors and formatting churn.
- Use `apply_patch` for deliberate manual file edits.
- Never run destructive Git commands, including `git reset --hard` or `git checkout --`, without an explicit request that identifies the intended scope.
- Never commit secrets, credentials, private notes, runtime snapshots, caches, downloaded assets or weights, experiment outputs, or generated files that a repository workflow does not explicitly designate as tracked source or maintained fixtures. Preserve tracked generated environment JSON and other maintained fixtures when their workflow requires them.
- Prefer lightweight static and unit checks before checks that start Isaac Sim, require a GPU, load ROS2, download assets, or call external services.
- Report the commands actually run, their results, and any validation limitations. Do not imply that an unrun check passed.
- Update documentation in the same change whenever public behavior, configuration, or workflows change.

## 3. System Requirements and Supported Versions

### Core Simulator Environment

- Ubuntu 22.04 is the supported host platform.
- Isaac Sim 5.1 is the simulator baseline.
- Isaac Lab 2.x must be installed with its Conda environment named `env_isaaclab`.
- A CUDA-capable NVIDIA GPU is normally expected for simulator workflows. CPU execution may work only for paths that explicitly support it.

### Development Tooling

- Node.js 20 LTS or a newer LTS release is required for the tracked Env DIY runtime check.

### ROS2 and Python Boundaries

ROS2 is optional for the core simulator and required only for ROS2 or Nav2 workflows. Humble on Ubuntu 22.04 remains the validated system-ROS baseline. The installer and runtime can select the Isaac Sim Humble or Jazzy bridge through `--ros-distro`/`ROS_DISTRO`, but this selection does not install system ROS or establish a validated Ubuntu 24.04/Jazzy Nav2 baseline. ROS2 command-line tools and Python programs that import `rclpy` must use the Python supplied by the selected system ROS rather than the Python interpreter in `env_isaaclab`. Keep the simulator and system ROS Python environments distinct unless a workflow explicitly integrates them.

### Animated Humans and PhysX

Selections containing animated humans force CPU PhysX. Isaac Sim 5.1 cannot safely perform the required animated pose writes with GPU PhysX, so a requested CUDA physics device is replaced with CPU for those selections.

Human rendering remains on the CUDA GPU, but the `UsdHumanStageRuntime` pose/retarget path is CPU work. Schedule it deliberately for large deployments:

- `UsdHumanStageRuntime.update(dt, *, context=None, actor_ids=None, animate_while_idle=False)` updates only the ids passed in `actor_ids`; unselected actors keep their clocks and pending events for later ticks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roso-lab/eai-simulator](https://github.com/roso-lab/eai-simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
