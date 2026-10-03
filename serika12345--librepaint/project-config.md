---
trigger: always_on
description: This file defines the operational contract for coding agents working on
---

# AGENTS.md

This file defines the operational contract for coding agents working on
LibrePaint. Use it with `docs/architecture/TODO.md`,
`docs/architecture/PROGRESS.md`, `docs/architecture/DEVELOPMENT.md`, and the
platform documents relevant to the active task.

## Communication

Use Japanese for plans, progress, blockers, verification results, and final
reports. Preserve repository language for source code, identifiers, commands,
paths, and quoted diagnostics.

Implementation reports include the change purpose, the resulting capability,
verification commands and results, remaining risk, and the next durable action.
Structural implementation reports lead with the purpose and identify every
starting file or directory together with its destination file or directory.
Internal target and class identifiers may support that mapping, but do not
replace it.

Implementation reports and pull request descriptions explain the change in
reviewer-facing domain language before introducing repository identifiers.
Titles state the architectural or behavioral outcome instead of leading with
target names, class names, or migration labels. Use this order, omitting a
section only when it does not apply:

1. State the concrete problem in the previous structure and its effect on
   ownership, dependencies, behavior, or maintenance.
2. Describe the resulting responsibility boundaries and dependency direction
   in conceptual terms.
3. Identify removed compatibility routes, obsolete structure, dead code, and
   unintended dependencies.
4. State the observable behavior and contracts that remain stable.
5. Give reviewers explicit points to inspect in the implementation.
6. Report verification by platform and scope, distinguishing successful
   checks from unrelated baseline failures and remaining risk.
7. Name the next scoped action.

Repository paths, CMake targets, class names, test names, and commands support
the explanation after the purpose and structural result are clear. A list of
internal identifiers is not a substitute for explaining what changed, why it
changed, and how a reviewer can judge correctness.

For a split, extraction, relocation, or ownership transfer, report the exact
starting files or directories and their destination files or directories as a
traceable mapping. File paths are required review entry points even when class,
target, and test identifiers are unnecessary supporting detail.

## Roadmap Order

LibrePaint follows the roadmap in `docs/architecture/TODO.md`.

1. R1 establishes responsibilities, package boundaries, and dependency
   direction.
2. R2 records behavioral, image, input, and performance contracts.
3. R3 optimizes rendering against the R2 contracts.
4. R4 introduces the Vulkan backend through the stable rendering boundaries.
5. R5 optimizes the mobile UI through shared application boundaries.
6. R6 completes C++20 adoption and repository-wide modernization.

Production integration enters each stage after its prerequisite completion
criteria pass. Exploration for later stages records findings in the relevant
roadmap item. R2 compatibility contracts precede R3 changes to painting
algorithms, execution order, scheduling, and synchronization.

## Resume Procedure

Durable project state lives in the repository documents. Architecture,
refactoring, test-foundation, and roadmap sessions begin with this sequence:

1. Read `docs/architecture/PROGRESS.md`.
2. Read the active gate in `docs/architecture/TODO.md` and its linked design
   or platform documents.
3. Inspect the current branch and worktree.
4. Validate the recorded next action against the current files.
5. Continue that action, or select the earliest planned gate whose
   prerequisites are complete.

Roadmap state changes update `docs/architecture/PROGRESS.md` in the same
change. The snapshot records a JST timestamp, state, gate, purpose, completed
work, next action, and verification status. Active work uses `in_progress`;
the next ready action uses `planned`; paused work records its resumption
condition.

Large roadmap items use reviewable gates with an explicit purpose, scope
boundary, completion criteria, verification tier, and stop condition.

## Development Environment

Nix defines development tools and dependencies. Direnv loads the `test` shell
from `.envrc`, adds repository scripts to `PATH`, and assigns the repository
build and compiler-cache roots. After the one-time `direnv allow`, entering the
repository provides these source-iteration commands:

```sh
build-incremental native build [target]
run-test <target> [ctest-regex]
verify-quick
verify
```

`build-incremental` selects persistent Ninja trees and platform-specific
compiler caches. macOS and Linux use the native test preset. iOS uses its
pinned device environment. Android and Windows use source-independent pinned
profiles on the x86_64 Linux build host. `path`, `configure`, `plan`, `build`,
`bootstrap`, and `cache-stats` expose each supported platform cycle.

Direct Nix entry remains available through `nix develop .#test`.
Documentation work may use `nix develop .#docs`. Required tool additions target
the narrowest relevant shell.

Nix expressions preserve small inputs and reusable cache boundaries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serika12345/librepaint](https://github.com/serika12345/librepaint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
