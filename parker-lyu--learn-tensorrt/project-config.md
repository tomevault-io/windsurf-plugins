---
trigger: always_on
description: transforms, resource ownership, synchronization, and non-obvious API choices.
---

# Project Instructions For Codex

This repository is a TensorRT deployment learning project. ISO C++17 is the primary implementation
language for inference and systems lessons, while Python and shell scripts support model export,
validation, profiling, and report generation. It should teach good engineering habits from the
beginning, not quick demo shortcuts.

## Required Development Container

- Use the persistent `learn-tensorrt` container for all project code execution unless a lesson or
  the user explicitly requires a different environment. This includes dependency commands,
  configuration, compilation, tests, Python and shell scripts, inference, profiling, and report
  generation; do not run these commands directly on the host by default.
- The container uses the `learn-tensorrt:25.11` image built from `docker/Dockerfile.dev`, with
  `nvcr.io/nvidia/pytorch:25.11-py3` as its pinned upstream image. The repository is bind-mounted at
  `/workspace/Learn-TensorRT`.
- Before running project code, start the existing container if necessary and execute the command in
  its repository directory, for example:

  ```bash
  docker start learn-tensorrt >/dev/null
  docker exec learn-tensorrt bash -lc \
    'cd /workspace/Learn-TensorRT && <command>'
  ```

- Run only host and container-management work on the host, such as `git`, file editing, Docker image
  and container management, driver checks, and NVIDIA Container Toolkit checks. A documented
  host-native lesson or an explicit user request is an exception.
- Reuse the persistent container instead of creating an ad hoc container. Follow
  `00_environment_check/agent_env_setup.md` when the image or container must be built, recreated, or
  repaired.

## Course Baseline

- Use `nvcr.io/nvidia/pytorch:25.11-py3` as the single upstream development image.
- Target TensorRT 10.14 (`10.14.1.48` in the pinned image) and CUDA Toolkit 13.0.
- Compile host C++ and CUDA C++ as ISO C++17 without GNU language extensions.
- Use the PyTorch and ModelOpt stack supplied by the development image for model export and
  explicit-Q/DQ workflows.
- Build engines, timing caches, references, and benchmark evidence with TensorRT 10.14 in the pinned
  development environment.

## Course Style

- Keep `README.md`, `docs/learning_roadmap.md`, and `docs/coverage_matrix.md` written for third-party
  learners taking the course; keep agent-only implementation instructions in `AGENTS.md`.
- Each implementation lesson should produce a runnable artifact and one concise README. Reporting
  checkpoints should produce a reproducible report and document how it was generated.
- Lesson code should be easy to read, but still structured like code that can evolve.
- Prefer small, focused files over one large file when a lesson has multiple concepts.
- Shared images, models, and reusable resources belong in the root `assets/` directory.
- When a lesson needs an input image, use `assets/img.jpeg` by default.
- Do not display images in GUI windows (for example, with `cv::imshow`); save images learners need
  to inspect in the lesson's `output/` directory instead.
- Transient build products, TensorRT engines, profiling captures, generated images, and local
  benchmark outputs should go to ignored output directories.
- Files under the root `reports/` directory are generated local evidence and must remain ignored.
  Small test fixtures, manifests, and reproducibility metadata outside that directory may be
  committed when they are intentional lesson deliverables.

## Lesson Modules

- Keep lesson directories as complete runnable implementations.
- Course 00 documents the shared runtime environment; do not repeat it in every later lesson.
- Design each lesson so a third party can reproduce it from scratch using only the repository at
  that revision and its documented external prerequisites. A lesson may depend on earlier lessons,
  but must not depend on files, generated artifacts, undocumented local state, or other resources
  that existed during development and were later deleted. Document any cross-lesson dependencies
  and the commands needed to reproduce the lesson in its README.
- Do not add separate `_practice` lesson folders or TODO-only starter copies unless explicitly
  requested.
- When a lesson benefits from hands-on guidance, put concise checkpoints or experiments in that
  lesson's README without duplicating the lesson directory.
- Do not replace a complete lesson with a TODO-only version unless explicitly requested.
- Do not create, switch, or push solution branches for the user unless explicitly requested.

## Lesson README Structure

- Treat `docs/learning_roadmap.md` as the course-level contract. A lesson README must implement the
  roadmap's purpose, deliverables, and acceptance boundary without silently expanding or narrowing
  them.
- Use these learner-facing sections in this order:
  1. `Purpose`
  2. `Prerequisites`
  3. `Deliverables`
  4. Optional `Setup` for lesson-specific dependency, data, or environment preparation
  5. Optional `Build` when the lesson compiles C++, CUDA, a plugin, or another native artifact
  6. `Run` for executable lessons or `Generate the Report` for reporting checkpoints
  7. `Outputs`
  8. Optional `Tests` when automated checks exist

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Parker-Lyu/Learn-TensorRT](https://github.com/Parker-Lyu/Learn-TensorRT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
