---
trigger: always_on
description: This repo is the canonical working source for reusable Codex infrastructure around future
---

# CppStudio Agent Notes

This repo is the canonical working source for reusable Codex infrastructure around future
C++/CUDA/Vulkan development.

## Required Orientation

- This is not a generated sample project. Do not treat it as a C++ app/library repo.
- The main artifact is the user-level Codex skill source at `skills/cpp-cuda-vulkan-studio/`.
- If available in the session, use the project skill `cppstudio-repo-onboarding` when starting
  work in this repo.
- The installed user-level copy at `${HOME}/.codex/skills/cpp-cuda-vulkan-studio` is a
  deployment target, not the source of truth.

## Source Of Truth

- Edit `skills/cpp-cuda-vulkan-studio/` in this repo.
- Use Planning Harness for durable planned-work state. The legacy
  `important-instruction-ledger` remains available only as a selective Recovery
  diagnostic; do not require it for ordinary CppStudio source edits.
- Use `docs/CODEBASE_ARCHITECTURE_INDEX.md` and `docs/CODEBASE_SUBSYSTEM_MANIFEST.json` as the
  maintained code map for this repo.
- Before changing repo files, use the maintained code map to choose the matching subsystem doc and
  primary paths for the work.
- Publish normal user-level Codex installs with `./scripts/rollout_to_codex.sh`.
- Use `./scripts/sync_to_codex.sh` only for dry runs, diagnostics, or an explicitly scoped
  single-skill sync.
- Do not hand-edit `${HOME}/.codex/skills/cpp-cuda-vulkan-studio` as the long-term source.
- Do not move private app, local workstation, or other project-specific skills back into user-level
  Codex from this repo.

## Evidence Routing

- Use repository facts first. Load donor or peer-tool evidence when a change
  depends on an unfamiliar design, product-shape choice, or external behavior.
- Use official specifications, headers, examples, and tests for standardized
  formats, protocols, SDK schemas, and conformance work.
- Do not require donor research for a bounded change whose owner and contract
  are already clear.

## Supervised Worker Interrogation

- When supervising a tmux, subagent, or other worker and the worker makes, skips, or rejects a
  decision for unclear reasons, interrogate that worker before claiming the cause, patching skills, or
  deciding no CppStudio fix is needed.
- Ask for the exact skill routes, donor routes, web/upstream sources, decision criteria, gaps, and
  verification commands that led to the behavior. Treat the answer as evidence to check against the
  transcript and files, not as authority.
- If the worker is unreachable, state that directly and inspect the transcript, project files, and
  CppStudio rules before drawing conclusions. Do not infer motives or root causes from memory.
- During supervised production lanes, continuously watch for worker behavior that points to a
  reusable CppStudio gap: skipped donor/profile routing, weak or mismatched verification, stale
  binaries, bad OSTM/session evidence, ignored code-map maintenance, unsupported helper commands,
  premature closeout, or drift from the approved slice. When such a signal appears, preserve the
  evidence, interrogate the worker if needed, and fix CppStudio or follow user-level cross-repo
  routing instead of treating it as a one-off worker mistake.

## Vulkan-Leaning Defaults

- If the user asks for a new GPU, 3D, rendering, realtime, simulation-visualization, XR, or
  cross-platform C++ project and does not specify CUDA or Vulkan, recommend Vulkan before the project
  starts because it gives the project an easier path to cross-platform and cross-vendor hardware
  compatibility.
- If the user does not choose after that recommendation, proceed with a Vulkan-first plan and state
  the assumption. Do not stop solely to ask whether they meant CUDA unless the requirements clearly
  imply custom CUDA kernels, NVIDIA-only libraries, or CUDA-specific deployment.
- When Vulkan is chosen by the user or assumed by default, keep the project Vulkan-only by default:
  do not add CUDA, CUDA interop, CUDA tests, CUDA runtime requirements, or CUDA donor routing unless
  the user explicitly asks for CUDA/Vulkan interop or the requirements force a CUDA-specific compute
  path.
- When the user explicitly chooses the CUDA lane, Vulkan may be mixed in when it is needed or clearly
  more convenient for presentation, realtime visualization, graphics, XR, swapchain/display work, or
  CUDA/Vulkan interop. State why the mixed lane is justified and keep the CUDA/Vulkan boundary
  explicit in build options, tests, and documentation.
- When both Vulkan and CUDA are plausible, document and route the Vulkan path first, then keep CUDA
  as a separate alternative lane, not as something automatically mixed into the Vulkan plan.
- For reusable 3D, rendering, realtime visualization, XR, or cross-vendor GPU work, frame the
  default route as Vulkan-first when requirements do not force an NVIDIA-only path.
- Prefer Vulkan-oriented donors, validation, shader tooling, synchronization notes, and CTest labels
  first for graphics/realtime tasks. Escalate to CUDA donors first only for custom CUDA kernels,
  CUTLASS/cuBLAS/cuDNN/TensorRT-style integrations, CUDA graphs, or explicitly NVIDIA-only targets.
- Keep CUDA support intact. Do not remove CUDA options, CUDA tests, CUDA donors, or CUDA companion

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarkansarim/CppStudio](https://github.com/tarkansarim/CppStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
