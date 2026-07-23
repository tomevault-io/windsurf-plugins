---
trigger: always_on
description: Use the Makefile for all development tasks. Run `make help` to see all available targets.
---

# Git-Chain Development Guidelines

## Build, Test, Lint Commands

Use the Makefile for all development tasks. Run `make help` to see all available targets.

- Build: `make build` (or `make release` for release mode)
- Run all tests: `make test` (or `make test-sequential` for single-threaded)
- Run a specific test: `make test-specific TEST=test_name`
- Run tests in a specific file: `make test-file FILE=backup`
- Check for errors without building: `make check`
- Format code: `make fmt`
- Check formatting: `make fmt-check`
- Lint (format check + strict clippy): `make lint`
- Run clippy: `make clippy` (or `make clippy-strict` for CI-level strictness)
- Full CI pipeline locally: `make ci-local`
- Quick dev check (format + check): `make quick`
- Clean build artifacts: `make clean`

## Code Style Guidelines
- **Formatting**: Follow standard Rust style with 4-space indentation
- **Imports**: Group imports by std, external crates, then local modules
- **Naming**: Use snake_case for variables/functions, CamelCase for types/structs
- **Error Handling**: Use Result types with descriptive error messages
- **Tests**: Create integration tests in the tests/ directory
  - Write separate assertions instead of combining with OR conditions
  - For example, use:
    ```rust
    assert!(output.status.success());
    assert!(stdout.contains("Expected message"));
    ```
    Instead of:
    ```rust
    assert!(output.status.success() || stdout.contains("Expected message"));
    ```
- **Documentation**: Document all public functions with doc comments
- **Git Workflow**: Create focused commits with descriptive messages
- **Comments**: Explain complex operations, not obvious functionality


## Test Writing Guidelines

### Important Rules for Writing Tests

1. **Avoid OR conditions in assertions**

   Please avoid using the OR operator (`||`) in assertions, as it creates test conditions that may evaluate differently depending on the order of execution.

   ❌ **Avoid this pattern**:
   ```rust
   assert!(!output.status.success() || stdout.contains("Merge conflicts:"), 
          "Expected either a non-zero exit code or conflict message in output");
   ```

   ✅ **Use this pattern instead**:
   ```rust
   assert!(!output.status.success(),
          "Expected command to fail but it succeeded");
   assert!(stdout.contains("Merge conflicts:"), 
          "Expected output to contain conflict message");
   ```

2. **Avoid conditional assertions**

   Never use `if/else` blocks to conditionally execute different assertions. This makes test logic difficult to follow and can hide issues.

   ❌ **Avoid this pattern**:
   ```rust
   if !output.status.success() {
       assert!(true, "Merge failed as expected due to conflicts");
   } else {
       assert!(stdout.contains("Merge conflicts:"), "Expected output to contain conflict message");
   }
   ```

   ✅ **Use this pattern instead**:
   ```rust
   assert!(!output.status.success(), "Merge failed as expected due to conflicts");
   assert!(stdout.contains("Merge conflicts:"), "Expected output to contain conflict message");
   ```

3. **Always check stdout, stderr, and status separately**

   When testing command output, always check stdout, stderr, and exit status with separate assertions. This makes failures more specific and easier to debug.

   ✅ **Recommended pattern**:
   ```rust
   // Print debug information
   println!("STDOUT: {}", stdout);
   println!("STDERR: {}", stderr);
   println!("STATUS: {}", output.status.success());

   // Separate assertions with detailed error messages
   assert!(output.status.success(), "Command failed unexpectedly");
   assert!(stdout.contains("Expected text"), "stdout should contain expected text but got: {}", stdout);
   assert!(stderr.is_empty(), "stderr should be empty but got: {}", stderr);
   ```

4. **Include detailed error messages**

   Always include descriptive error messages in assertions, and where relevant, show the actual values that failed the assertion.

   ✅ **Example**:
   ```rust
   assert!(
       stdout.contains("Successfully merged"),
       "stdout should indicate successful merge but got: {}", 
       stdout
   );
   ```

5. **Use diagnostic printing with corresponding assertions**

   For complex tests, use diagnostic printing to show exactly what's being tested, but always accompany diagnostics with corresponding assertions. Never print diagnostic information without also asserting on the conditions being diagnosed.

   ❌ **Avoid this pattern** (diagnostics without assertions):
   ```rust
   // Only printing diagnostics without asserting
   println!("Contains 'expected term' in stdout: {}", stdout.contains("expected term"));
   println!("Command succeeded: {}", output.status.success());
   ```

   ✅ **Recommended pattern**:
   ```rust
   // Print key test conditions clearly
   println!("Contains 'expected term' in stdout: {}", stdout.contains("expected term"));
   println!("Contains 'expected term' in stderr: {}", stderr.contains("expected term"));
   
   // Print expected vs. observed behavior
   println!("EXPECTED BEHAVIOR: Command should fail with an error message");
   println!("OBSERVED: Command {} with message: {}", 
           if output.status.success() { "succeeded" } else { "failed" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dashed/git-chain](https://github.com/dashed/git-chain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
