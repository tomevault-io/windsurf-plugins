---
trigger: always_on
description: SPDX-FileCopyrightText: Copyright The Zephyr Project Contributors
---

<!--
SPDX-FileCopyrightText: Copyright The Zephyr Project Contributors
SPDX-License-Identifier: Apache-2.0
-->

# Zephyr RTOS: instructions for AI coding agents

This file is for coding agents, and for the humans driving them, working in this repository.
It is a digest of `doc/contribute/` plus the things CI and maintainers reject most often. Where
it is silent or disagrees with the documentation, the documentation wins:
`doc/contribute/guidelines.rst` (process, DCO, AI-assistant policy),
`doc/contribute/contributor_expectations.rst` (PR shape), `doc/contribute/style/` (C, Kconfig,
CMake, Doxygen and devicetree style) and `doc/contribute/coding_guidelines/index.rst` (the
MISRA-C subset). Read the relevant page before working in an area you have not touched before;
do not rely on this digest or on training data alone.

## Rules for agents

- Never add a `Signed-off-by:` line: only the human submitter may sign off (DCO). Never add
  `Co-authored-by:`. Do not put "Generated with ...", session links or any other mention of AI
  tools in commit messages, PR bodies, issues or comments. The `Assisted-by:` trailer below is
  the only place AI involvement is recorded.
- Add exactly one `Assisted-by: <Agent>:<model-version> [tool ...]` trailer, for example
  `Assisted-by: Claude:claude-opus-4.6 coccinelle`, naming the tool actually used. Replace it
  rather than stacking when a different model amends the commit. `checkpatch.pl` validates the
  format; basic tools (git, gcc, cmake, editors) are not listed.
- The human reviews and tests every change before it is submitted. State what you did not do
  (not built, not run, not run on hardware, no reproducer) instead of implying it was done.
- Review comments written with AI help are verified by the human before posting; never post raw
  model output (`doc/contribute/reviewer_expectations.rst`).
- Commit messages, code comments, Kconfig help texts and docs describe the tree as it is: no
  references to prompts, plans, sessions, "the rework" or "as requested". Write tersely and
  concretely; maintainers reject verbose, hedging or self-congratulatory prose. No emoji.
- Change what the task needs, which can mean refactoring the code a fix touches, but keep
  unrelated reformatting, renames and cleanups out of the change; style is enforced on new or
  modified lines only. A wider cleanup is a separate PR.

## Build and test

- Zephyr builds only inside a west workspace: from the parent directory of the clone,
  `west init -l <zephyr-dir> && west update` (most boards need HAL and module repositories from
  `west.yml`). Build with `west build -b <board> <app-dir>`; `native_sim` runs on the host.
  See `doc/develop/west/workspaces.rst`.
- Tests and samples are run by twister: `west twister -p native_sim -T <test-dir>` or
  `-s <test-dir>/<scenario-id>`; `--build-only` skips execution; `-i` prints failing logs.
  Suites whose `tests.yaml` lists `unit_testing` as platform need `-p unit_testing`. See
  `doc/develop/twister/index.rst`.
- In a `git worktree`, export `ZEPHYR_BASE=<worktree>` for `west build` and twister; otherwise
  the workspace's registered checkout is built silently instead of your tree.
- Every commit in a series must build and pass its tests on its own (bisectability).

## Checks to run before pushing

CI runs the same checks (`.github/workflows/compliance.yml`); each failure costs a full round
trip. Fix the cause, never work around a check.

- `pip install -r scripts/requirements-compliance.txt`, then
  `./scripts/ci/check_compliance.py --parallel -c upstream/main..HEAD` (`upstream` being the
  zephyrproject-rtos remote, `origin` in a plain clone; the same below). `-l` lists the checks
  (Checkpatch, Gitlint, KconfigBasic, CMakeStyle, DevicetreeBindings, Ruff, Pylint, YAMLLint,
  SphinxLint, KeepSorted, ...); `-m <Check>` runs one.
- checkpatch on one commit: `git format-patch -1 --stdout <sha> | ./scripts/checkpatch.pl -`.
  Piping `git show` instead produces bogus `BAD_SIGN_OFF` errors.
- ClangFormat is advisory (`.clang-format`). Apply it to your own hunks only and skip hunks
  where it reflows surrounding macro tables; never run it on whole files.
- CMake style (`doc/contribute/style/cmake.rst`) is checked on touched lines only and old files
  are grandfathered: copying an old `CMakeLists.txt` as a template can fail CI although the
  original passes. The same holds for any in-tree file you copy: check it against the current
  style page, not only against its neighbors.

## Commits

    area: subarea: imperative summary (75 columns max, no trailing period)

    Body wrapped at 75 columns: what the change does, why this approach,
    which assumptions were made and how it was tested. Never empty.

    Fixes #12345

    Assisted-by: Claude:claude-opus-4.6
    Signed-off-by: Full Name <email@example.com>

- `area:` is the prefix the file's recent history uses: `git log --format=%s -20 -- <path>`.
  Examples: `Bluetooth: Host:`, `drivers: i2c: nrfx:`, `dts: arm: st:`, `boards: nordic:`,
  `kernel:`, `doc:`, `.github:`. Never `subsys:` or `treewide:`, never `WIP`.
- Trailers (`Assisted-by`, `Signed-off-by`, `Link:`, `(cherry picked from commit ...)`) form one
  block as the last paragraph. `Fixes #N` goes in its own paragraph directly before that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zephyrproject-rtos/zephyr](https://github.com/zephyrproject-rtos/zephyr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
