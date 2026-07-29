---
trigger: always_on
description: Use when modifying LearningTestTool Python scripts, test families, result comparison logic, or test infrastructure under test/LearningTestTool/. Covers test tool commands, directory conventions, result context management, and comparison patterns.
---


# LearningTestTool Instructions

## Overview

LearningTestTool is the **non-regression test infrastructure** for Khiops. It manages 800+ test cases across 40+ suites for three tools (KMODL, MODL_Coclustering, KNITransfer). See `test/LearningTestTool/README.md` for full usage details.

## Directory Structure

```
test/LearningTestTool/
  ├─ py/                     # All Python implementation
  │   ├─ kht_test.py         # Main command: run tests & compare results
  │   ├─ kht_apply.py        # Apply maintenance instructions on test dirs
  │   ├─ kht_collect_results.py  # Gather results by filter (errors, warnings)
  │   ├─ kht_export.py       # Backup/archive test trees
  │   ├─ kht_env.py          # Display Khiops environment variables
  │   ├─ kht_help.py         # Show command overview
  │   ├─ _kht_constants.py   # All shared constants (paths, tool names, timeouts)
  │   ├─ _kht_utils.py       # Utility functions, directory type validation
  │   ├─ _kht_families.py    # Test family definitions (basic, full, complete)
  │   ├─ _kht_results_management.py  # Result context: platform, computing mode
  │   ├─ _kht_check_results.py       # Cross-platform result comparison engine
  │   ├─ _kht_standard_instructions.py  # Built-in maintenance instructions
  │   └─ _kht_one_shot_instructions.py  # One-off maintenance instructions
  ├─ sh/                     # Linux/macOS shell wrappers → py/
  └─ cmd/                    # Windows batch wrappers → py/
```

## File Naming Convention

- **`kht_*.py`**: User-facing command scripts (entry points)
- **`_kht_*.py`**: Internal modules (prefixed with `_`, not called directly)

## LearningTest Directory Hierarchy

The tool operates on a strict 4-level directory hierarchy:

| Level | Name | Example | Contains |
|-------|------|---------|----------|
| home dir | `LearningTest/` | `test/LearningTest/` | Tool dirs + dataset collections |
| tool dir | `Test<Tool>/` | `TestKhiops/` | Suite dirs |
| suite dir | `<SuiteName>/` | `Standard/` | Test dirs |
| test dir | `<TestName>/` | `IrisLight/` | `test.prm`, `results/`, `results.ref*` |

Each test dir contains:
- `test.prm` — Khiops scenario file (required)
- `test.json` — Optional JSON parameters
- `results/` — Current test output
- `results.ref*` — Reference results (may have context suffixes)
- `comparisonResults.log` — Comparison output

## Three Khiops Tools

| Tool Name | Executable | Test Dir | Parallel |
|-----------|-----------|----------|----------|
| `Khiops` | `MODL` | `TestKhiops/` | Yes |
| `Coclustering` | `MODL_Coclustering` | `TestCoclustering/` | Yes |
| `KNI` | `KNITransfer` | `TestKNI/` | No |

Constants are in `_kht_constants.py`: `TOOL_NAMES`, `TOOL_EXE_NAMES`, `TOOL_DIR_NAMES`, `PARALLEL_TOOL_NAMES`.

## Test Families

Defined in `_kht_families.py`. Each family is a list of test suites per tool:

| Family | Scope | Approximate Duration |
|--------|-------|---------------------|
| `basic` | Minimal smoke test (`Standard` suite only) | ~1 min |
| `full` | All non-regression suites (default) | ~1 hour |
| `full-no-kni` | Same as `full` without KNI | ~1 hour |
| `complete` | Exhaustive testing | ~1 day |
| `all` | All subdirectories (management, not testing) | N/A |

To add a test suite to a family, add it to `FAMILY_TEST_SUITES[family, tool]` in `_kht_families.py`.

## Main Commands

### `kht_test` — Run Tests

```bash
kht_test <source_dir> <binaries> [options]
```

- **`source_dir`**: test/suite/tool/home dir (auto-detected level); can be an individual test dir like `TestKhiops/CrashTests/trainclassifier_KWDatabaseSlicerTask`
- **`binaries`**: path to tool executables, or aliases `r` (release), `d` (debug), `check` (comparison only, no execution)
- Key options: `-f/--family`, `-p/--processes N` (MPI processes), `--min-test-time`, `--max-test-time`, `--test-timeout-limit`

Examples:
```bash
# Run all CrashTests with 4 MPI processes using release binary
kht_test /path/to/LearningTest/TestKhiops/CrashTests r -p 4

# Compare results only (no re-run)
kht_test /path/to/LearningTest/TestKhiops/CrashTests check -f full -p 4

# Run a single test dir with debug binary
kht_test /path/to/LearningTest/TestKhiops/Standard/Iris d -p 1
```

### `kht_apply` — Maintenance Instructions

```bash
kht_apply <instruction> [source_dir]
```

Built-in instructions: `errors`, `makeref`, `list`, `logs`. Custom one-shot instructions defined in `_kht_one_shot_instructions.py`.

### `kht_collect_results` — Gather Results

```bash
kht_collect_results <source_dir> <target_dir> [--all|--errors|--warnings]
```

### `kht_export` — Archive Tests

```bash
kht_export <source_dir> <target_dir> [--all|--scripts|--references|--datasets]
```

## Result Context Management

Reference results can specialize by **computing mode** and **platform** via directory name suffixes:

```
results.ref                          # Default (fallback)
results.ref-parallel                 # Parallel-specific
results.ref-sequential               # Sequential-specific
results.ref-Darwin_Linux             # macOS or Linux

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KhiopsML/khiops](https://github.com/KhiopsML/khiops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
