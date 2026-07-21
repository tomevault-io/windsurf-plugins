---
trigger: always_on
description: - NEVER run the vite dev server or anything that listens on any port yourself! Ask the user to do it!
---

# Agent Rules for GoScript

## IMPORTANT

- NEVER run the vite dev server or anything that listens on any port yourself! Ask the user to do it!
- Read `docs/explainer.md` for an overview of the project if you need it
- The `design/` documents (including `design/DESIGN.md` and `design/SPEC_DIFFERENCES.md`) are OUTDATED and are NOT authoritative. They describe an earlier state of the compiler and runtime and have drifted from the actual behavior. Treat the compiler source, the `gs/` runtime overrides, and the passing compliance tests under `tests/tests/` as the source of truth. When you touch an area a design doc describes, update that doc to match reality (or note the divergence) rather than implementing to the stale text. They need a full review and update.

### Project-Specific Rules

- When creating git commits when explicitly asked do not add co-author attribution lines. Always use git commit -s to sign-off.
- NEVER use emdash `--` anywhere.
- When creating PRs, match the target repo's existing PR description style. For repos with minimal/plain text PR descriptions, do NOT use markdown headers (## Summary, ## Test plan, etc.) or bullet lists. Use plain prose matching the commit message style.

- DO NOT maintain backwards compatibility - this is an experimental project
- Remove any "for backwards compatibility" comments and fallback logic
- NEVER hardcode things: examples include function names, builtins, etc.
- Actively improve touched code and design docs when the opportunity is clear: if you find a stale or contradictory design note in the area you are changing, update the doc to match the source/tests instead of preserving the contradiction.
- Never use `as unknown as ...`. It is a red flag for a type-system escape hatch hiding a bad runtime contract; return the actual runtime type or fix the owner type signature so the checker catches mismatches.
- Go standard library sources are located at "go env GOROOT" (shell command)
- Leverage adding more tests (e.g., `compiler/analysis_test.go`) instead of debug logging for diagnosing issues. If the new test case is temporary, add a `tmp_test.go` file to keep things separated.
- AVOID type arguments unless necessary (prefer type inference)
- When making Git commits referencing issues use the short form: Fixes #128 (for example)
- When making Git commits use the existing commit message pattern and Linux-kernel style commit message bodies.
- When you would normally add a new compliance test check if a very-similar compliance test already exists and if so extend that one instead. For example testing another function in the same package.

## Project Overview

GoScript is an experimental Go to TypeScript transpiler that enables developers to convert high-level Go code into maintainable TypeScript. It translates Go constructs—such as structs, functions, and pointer semantics—into idiomatic TypeScript code while preserving Go's value semantics and type safety. It is designed to bridge the gap between the robust type system of Go and the flexible ecosystem of TypeScript.

**This is an experimental project** - we do not maintain backwards compatibility and prioritize simplicity and correctness over legacy support. You may sometimes encounter a problem that requires a complete re-design or re-think or re-architecting of an aspect of goscript, which is perfectly okay, in this case write a design to `tests/WIP.md` and think it through extensively before performing your refactor. It's perfectly OK to delete large swaths of code as needed. Focus on correctness.

If you want to overwrite WIP.md you must `rm` it first.

The GoScript runtime, located in `gs/builtin/builtin.ts`, provides necessary helper functions and is imported in generated code using the `@goscript/builtin` alias.

**Output Style**: Generated TypeScript should not use semicolons and should always focus on code clarity and correctness.

**Philosophy**: Follow Rick Rubin's concept of being both an engineer and a reducer (not always a producer) by focusing on the shortest, most straightforward solution that is correct.

## Compliance Testing Workflow

When working on compliance tests:

1. **Test Location**: Compliance tests are located at `./tests/tests/{testname}/testname.go` with a package main and using `println()` only for output, trying to not import anything.

2. **Running Tests**:

   **For a specific test:**
   ```bash
   go test -timeout 60s -run ^TestCompliance/if_statement$ ./compiler
   ```

   **For a full local suite (optional; useful for deliberate breadth checks):**
   ```bash
   # Run once, capture to file, check result
   mkdir -p .tmp && go test -timeout 10m ./compiler 2>&1 > .tmp/test_output.txt; echo "Exit code: $?"

   # If exit code is non-zero, find all failing tests:
   grep -E "^--- FAIL:" .tmp/test_output.txt

   # Then run specific failing tests with -v for details:
   go test -v -timeout 60s -run ^TestCompliance/failing_test_name$ ./compiler
   ```

   **IMPORTANT:** Do NOT pipe test output directly to grep/tail during the test run. The test framework may produce verbose output that looks like errors but isn't. Always check the exit code first, then analyze the output file if needed. The `.tmp/` directory is gitignored.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aperturerobotics/goscript](https://github.com/aperturerobotics/goscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
