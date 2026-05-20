---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Paserati is a production-quality TypeScript/JavaScript runtime written in Go that compiles TypeScript directly to bytecode for a register-based virtual machine, bypassing JavaScript transpilation entirely.

**Core Goals**:

1. **Correctness**: Full ECMAScript 262 compliance and TypeScript compatibility
2. **Performance**: Optimized execution with inline caching, shape-based objects, and register allocation

The runtime executes TypeScript directly without lowering to JavaScript, using compile-time type information for optimizations while maintaining full JavaScript semantics for runtime behavior.

## Development Commands

### Build and Run

```bash
# Build the main binary
go build -o paserati cmd/paserati/main.go

# Run the REPL
./paserati

# Execute a TypeScript file
./paserati path/to/script.ts

# Run expression directly
./paserati -e "let x = 42; x * 2"

# Show bytecode output
./paserati -bytecode script.ts

# Show inline cache statistics
./paserati -cache-stats script.ts

# Disable type checking (for testing JavaScript semantics)
./paserati --no-typecheck script.js
```

**Note**: Test262 runs without type checking enabled (using `SetIgnoreTypeErrors(true)` in the driver).

### Testing

```bash
# Run all tests
go test ./tests/...

# Run script-based tests (most comprehensive) - THIS IS OUR SMOKE TEST
# This test suite MUST always be green - fix issues or mark as FIXME
go test ./tests -run TestScripts

# Run a specific script test
go test ./tests -run "TestScripts/filename.ts"

# Run with verbose output to see individual test names
go test ./tests -v
```

**Important**: The `TestScripts` suite in `tests/scripts_test.go` is our smoke test and should always pass. When something doesn't work, either fix it immediately or mark it as FIXME - never sweep issues under the rug.

**Smoke Test Expectation Format**: Tests in `tests/scripts/` use special comments to define expected outcomes:

```typescript
// expect: value                    // Compare against LAST STATEMENT value
// expect_runtime_error: message    // Expected runtime error
// expect_compile_error: message    // Expected compile error
```

**Critical**: The `// expect: value` comment compares against the **value of the last statement** in the script, NOT against printed output. For example:

```typescript
// expect: 42
let x = 40;
let y = x + 2;
y;  // <-- This expression's value (42) is compared against "expect"
```

The test runner evaluates the script and compares the result of the final expression to the expected value. `console.log()` output is ignored for test comparison purposes.

**Temporary Test Files**: Use the `scratch/` directory (gitignored) for temporary test files, debug scripts, and experimental code. This keeps the repository clean while you debug issues.

### Test262 Compliance Testing

Paserati uses the official ECMAScript Test262 suite to verify language compliance. The `paserati-test262` binary runs these tests with our runtime.

```bash
# Build the test262 runner
go build -o paserati-test262 ./cmd/paserati-test262

# Run all Test262 tests (takes a while)
./paserati-test262 -path ./test262

# Run specific test suite with filtering
./paserati-test262 -path ./test262 -subpath "language/expressions" -filter -timeout 0.5s

# Show suite breakdown with pass rates
./paserati-test262 -path ./test262 -subpath "language/expressions" -suite -filter -timeout 0.5s

# Run a specific test for debugging
./paserati-test262 -path ./test262 -subpath "language/expressions/addition" -pattern "S11.6.1_A3.1_T1.1.js"
```

**Key Flags**:

- `-suite`: Shows pass rates broken down by test suite and subsuite
- `-filter`: Filters out legacy JavaScript patterns (e.g., `with` statements, old ES5 patterns) not relevant for modern TS runtime
- `-timeout`: Set timeout per test (default 5s, use shorter like 0.5s for faster iteration)
- `-subpath`: Limit tests to a specific subdirectory (e.g., "language/expressions", "built-ins/Array")
- `-pattern`: File pattern for test files (default "\*.js")

**Workflow for Fixing Test262 Failures**:

1. **Identify target area**: Run with `-suite -filter` to see pass rates by category

   ```bash
   ./paserati-test262 -path ./test262 -subpath "language/expressions" -suite -filter -timeout 0.5s
   ```

2. **Analyze failure patterns**: Look for common error messages

   ```bash
   ./paserati-test262 -path ./test262 -subpath "language/expressions/addition" -filter | grep "^FAIL" | head -20
   ```

3. **Debug specific failures**: Run individual tests with verbose output

   ```bash
   # Test a specific file
   go run ./cmd/paserati path/to/failing-test.js

   # Or with the test runner
   ./paserati-test262 -path ./test262 -subpath "path/to" -pattern "specific-test.js"
   ```

4. **Create smoke tests**: Add simplified versions to `tests/scripts/` for regression testing

   ```typescript
   // tests/scripts/feature_test.ts
   // expect: value
   console.log("test output");
   ```

5. **Fix the issue**: Make changes following the architecture (lexer → parser → checker → compiler → VM)

6. **Verify improvement**: Re-run the suite to confirm increased pass rate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nooga/paserati](https://github.com/nooga/paserati) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
