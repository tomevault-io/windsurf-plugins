---
trigger: always_on
description: - `zig/` - Zig compiler submodule (DO NOT MODIFY - all changes must be non-AI)
---

# CLR Project

## Project Structure

- `zig/` - Zig compiler submodule (DO NOT MODIFY - all changes must be non-AI)

## Build Commands

- `zig/b` - Builds the custom Zig compiler (uses `zig build --zig-lib-dir lib`)
- `zig/z` - Runs the custom Zig compiler from `zig-out/bin/zig`
- `zig build -Doptimize=ReleaseFast` - Builds libclr.so (the AIR plugin) **IMPORTANT: Always use ReleaseFast unless you need to do advanced debugging and you have recompiled Zig to be in a different mode**
- `zig build test` - Runs unit tests for libclr
- `./run_integration.sh` - Runs BATS integration tests
- `./run_one.sh <test_file>` - Run a single test case (generates `.air.zig` file in project root)
- `./dump_air.sh <source_file> <function_name> [num_lines]` - Dump AIR for a specific function
- `./clear.sh` - Clean up generated `.air.zig` files and other build artifacts

### Debugging AIR

To view the raw AIR for a function:
```sh
./dump_air.sh test/cases/undefined/basic/assigned_before_use.zig assigned_before_use.main 40
```

This shows the instruction indices, tags, and nesting structure. Block bodies may have indices that are **higher** than post-block instructions (e.g., block at %10 may contain %16-%23, while %11-%15 come after the block).

**IMPORTANT**: Do NOT use `-femit-air` or `--verbose-air` flags directly. The main Zig compiler is built in ReleaseFast mode which strips AIR emission support. Always use `./dump_air.sh` which uses a debug-mode Zig build.

## Testing

### Unit Tests

There are TWO sets of unit tests in different locations:

**1. Codegen/DLL tests (`src/`)** - Run with:
```sh
zig build test
```
These test the code generation and DLL infrastructure.

**2. Runtime library tests (`lib/`)** - Run with:
```sh
zig test lib/lib.zig
```
These test the runtime analysis logic (tag handlers, memory_safety, undefined tracking).

**IMPORTANT**: `zig build test` only runs `src/` tests. Always run BOTH when modifying analysis logic.

### Integration Tests (BATS)

Integration tests use [BATS](https://github.com/bats-core/bats-core) (Bash Automated Testing System).

```sh
# Install BATS (if needed)
sudo apt install bats  # Ubuntu/Debian
brew install bats-core # macOS

# Run all integration tests
./run_integration.sh

# Run a single test file
bats test/integration/allocator.bats

# Run tests matching a pattern
bats test/integration/allocator.bats -f "double-free"
```

**Performance Note**: Full integration tests take ~7 minutes to run. Only run `./run_integration.sh` before commits. During development, use targeted testing:
- `./run_one.sh <test_file>` - Test a single case quickly
- `bats test/integration/<file>.bats -f "pattern"` - Run specific tests by pattern
- `bats test/integration/<file>.bats` - Run one test file

**Avoid Redundant Test Runs**: When verifying a feature works:
1. Use `./run_one.sh <test_file>` ONCE to check the output
2. If it works, you're done - don't re-run with BATS or other methods
3. Don't run the same test multiple ways (run_one.sh, then bats, then run_one.sh again)
4. Trust the first result unless you changed code between runs

**Integration Test Efficiency**: The full test suite is expensive (~7 min). Follow these rules:
1. Run `./run_integration.sh` only ONCE per feature, right before committing
2. If the output shows all tests as "ok", they passed - don't re-run to "verify"
3. **NEVER pipe `./run_integration.sh` to `| tail`, `| head`, `| grep`, etc.** - this wastes the full test run. Instead, use tee to capture and display: `./run_integration.sh 2>&1 | tee /tmp/integration_results.txt; echo "Exit code: $?"`
4. If you need to check a specific test, use `./run_one.sh` or `bats -f "pattern"` instead of the full suite
5. NEVER run the full integration suite multiple times in a row for the same set of changes

**Note**: Integration tests are expected to fail during development (the CLR runtime is incomplete). However, if the tests fail due to **compilation errors in the emitted .air.zig analyzer**, that indicates a real problem in codegen that needs to be fixed.

**Important**: Always run BATS from the project root directory. The test helper uses relative paths from its location to find the compiler, libclr.so, and test cases.

**CRITICAL**: NEVER materially modify, comment out, or skip integration tests to make the test suite pass without permission. Failing tests are intentional - they serve as reminders of work to be done. If a test fails, fix the codegen or runtime - not the test. Minor updates like fixing line numbers after test file changes are fine. Integration tests should simply call `compile_and_run` and check the result.

**CRITICAL**: NEVER delete, disable, skip, or remove integration tests without explicit permission. If a test is failing:
1. Fix the underlying bug in codegen or runtime
2. If the test cannot be fixed yet, ask before skipping it
3. Skipped tests MUST have a comment explaining why and reference to tracking issue/doc

**CRITICAL**: Do NOT dismiss or report integration test failures as "pre-existing" without verification. When tests fail:
1. Investigate whether your changes caused the failure
2. If uncertain, check git history or ask the user
3. Never assume a failure existed before your changes - investigate properly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ityonemo/clr](https://github.com/ityonemo/clr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
