---
trigger: always_on
description: This is the codebase for Apache Doris, an MPP OLAP database. It primarily consists of the Backend module BE (`be/`, execution and storage engine), the Frontend module FE (`fe/`, optimizer and transaction core), and the Cloud module (`cloud/`, storage-compute separation). Your basic development workflow is: modify code, build using standard procedures, add and run tests, and submit relevant changes.
---

# AGENTS.md — Apache Doris

This is the codebase for Apache Doris, an MPP OLAP database. It primarily consists of the Backend module BE (`be/`, execution and storage engine), the Frontend module FE (`fe/`, optimizer and transaction core), and the Cloud module (`cloud/`, storage-compute separation). Your basic development workflow is: modify code, build using standard procedures, add and run tests, and submit relevant changes.

## Security Threat Model

For security scans, vulnerability triage, security reviews, and changes involving authentication, authorization, network boundaries, external catalogs, cloud tenancy, or other security-sensitive behavior, read `SECURITY.md` first, then follow it to `threat-model.md`. Use `threat-model.md` to determine in-scope components, trust boundaries, attacker roles, explicit non-goals, and triage classification. Findings that are out of model or by design under `threat-model.md` should be reported as such, not treated as Doris vulnerabilities. Such reviews will not be conducted unless explicitly requested by the user, and no conclusions regarding this matter need to be provided.

## When running in a WORKTREE directory

To ensure smooth test execution without interference between worktrees, the first thing to do upon entering a worktree directory is to check if `.worktree_initialized` exists. If not, execute `hooks/setup_worktree.sh`, setting `$ROOT_WORKSPACE_PATH` to the base directory (typically `${DORIS_REPO}`) beforehand. After successful execution, verify that `.worktree_initialized` has been touched and that `thirdparty/installed` dependencies exist correctly. Also check if submodules have been properly initialized; if not, do so manually.

When working in worktree mode, all operations must be confined to the current worktree directory. Do not enter `${DORIS_REPO}` or use any resources there. Compilation and execution must be done within the current worktree directory. The compiled Doris cluster must use random ports not used by other worktrees (modify BE and FE conf before compilation, using a uniform offset of `${DORIS_PORT_OFFSET_RANGE}` from default ports without conflicting with other worktrees' ports). Run from the `output` directory within the worktree. To run regression tests, modify `regression-test/conf/regression-conf.groovy` and set the port numbers in `jdbcUrl` and other configuration items to your new ports so the corresponding worktree cluster can be used for regression testing.

## Coding Standards

Assert correctness only—never use defensive programming with `if` or similar constructs. Any `if` check for errors must have a clearly known inevitable failure path (not speculation). If no such scenario is found, strictly avoid using `if(valid)` checks. However, you may use the `DORIS_CHECK` macro for precondition assertions (if inside performance-sensitive areas like loops, it can only be `DCHECK`). For example, if logically A=true should always imply B=true, then strictly avoid `if (A && B)` and instead use `if (A) { DORIS_CHECK(B); ... }`. In short, the principle is: upon discovering errors or unexpected situations, report errors or crash—never allow the process to continue.

For `PaddedPODArray` and its peripheral packaging types, such as some certain Column, negative alignment allows the use of -1 as a valid index. No additional special handling is needed when the index may be -1.

When adding code, strictly follow existing similar code in similar contexts, including interface usage, error handling, and locking patterns. When adding any code, first try to reference existing functionality. Second, examine the relevant context paragraphs to fully understand the logic.

After adding code, conduct self-review and refactoring attempts to ensure good abstraction and reuse as much as possible.

### Code Style Enforcement

All code must pass style checks before committing. Use the corresponding skill for detailed step-by-step procedures.

**BE (C++) Formatting**: Run `build-support/clang-format.sh` to auto-fix formatting. This script enforces clang-format v16; do not use other versions. Run `build-support/check-format.sh` to check without modifying files. See the `be-code-style` skill for details.

**BE (C++) Static Analysis**: After building BE (which generates `compile_commands.json`), run `build-support/run-clang-tidy.sh` to check modified C++ files against the `.clang-tidy` config. The script parses `git diff` to filter warnings to changed lines where possible, reducing noise from pre-existing code (diagnostics from included headers may still appear). For Cloud C++ files, pass `--build-dir` pointing to the Cloud compilation database (e.g., `cloud/build_ASAN`). Try to fix all reported warnings; if a warning cannot be reasonably fixed, add a `// NOLINT` comment with justification and report it. See the `clang-tidy-check` skill for details.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/doris](https://github.com/apache/doris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
