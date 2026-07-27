---
trigger: always_on
description: PowerShell Pester testing best practices based on Pester v5 conventions
---


# PowerShell Pester v5 Testing Guidelines

This guide provides PowerShell-specific instructions for creating automated tests using PowerShell Pester v5 module. Follow PowerShell cmdlet development guidelines in [powershell.instructions.md](./powershell.instructions.md) for general PowerShell scripting best practices.

## File Naming and Structure

- **File Convention:** Use `*.Tests.ps1` naming pattern
- **Placement:** Place test files next to tested code or in dedicated test directories
- **Import Pattern:** Use `BeforeAll { . $PSScriptRoot/FunctionName.ps1 }` to import tested functions. Use `$PSScriptRoot` or `$PSCommandPath` to locate scripts — do NOT use `$MyInvocation.MyCommand.Path` inside `BeforeAll` (it does not work in Pester v5).
- **No Direct Code:** Put ALL code inside `It`, `BeforeAll`, `BeforeEach`, `AfterAll`, or `AfterEach`. Code placed directly in a `Describe` or `Context` body outside these blocks runs during **Discovery**, not during test execution — its state is NOT available when tests run. Use `BeforeDiscovery` for code that must explicitly run during Discovery.

## Test Structure Hierarchy

```powershell
BeforeAll { # Import tested functions }
Describe 'FunctionName' {
    Context 'When condition' {
        BeforeAll { # Setup for context }
        It 'Should behavior' { # Individual test }
        AfterAll { # Cleanup for context }
    }
}
```

## Discovery and Run

Pester v5 runs test files in two phases: **Discovery** (collecting tests) and **Run** (executing tests).

- `Describe` and `Context` scriptblocks are invoked during **Discovery** to collect tests — all other scriptblocks (`It`, `BeforeAll`, etc.) are saved and invoked during **Run**
- Code placed directly in a `Describe` or `Context` body (outside sub-blocks) runs during Discovery — its results are NOT available during Run
- Variables set in `BeforeAll` are NOT available in `-TestCases`/`-ForEach` or `-Skip` conditions — these are evaluated during Discovery before `BeforeAll` runs
- Use `BeforeDiscovery { }` for code that must run during Discovery (e.g., building test case data from external sources)
- Use `-ForEach` on `Describe`, `Context`, or `It` to pass Discovery-time data into Run-phase blocks
- `$TestDrive` is only available during Run — it cannot be used in `-ForEach` data

```powershell
# WRONG — $items is from BeforeAll (Run phase), not available in -ForEach (Discovery phase)
Describe 'Example' {
    BeforeAll {
        $script:items = @('a', 'b')
    }
    It 'test <_>' -ForEach $script:items {  # $script:items is $null here
        $_ | Should -Not -BeNullOrEmpty
    }
}

# CORRECT — use BeforeDiscovery to populate data used during Discovery
BeforeDiscovery {
    $items = @('a', 'b')
}
Describe 'Example' {
    It 'test <_>' -ForEach $items {
        $_ | Should -Not -BeNullOrEmpty
    }
}
```

## Core Keywords

- **`Describe`**: Top-level grouping, typically named after function being tested
- **`Context`**: Sub-grouping within Describe for specific scenarios
- **`It`**: Individual test cases, use descriptive names
- **`Should`**: Assertion keyword for test validation
- **`BeforeAll/AfterAll`**: Setup/teardown once per block
- **`BeforeEach/AfterEach`**: Setup/teardown before/after each test

## Setup and Teardown

- **`BeforeAll`**: Runs once during the **Run** phase at the start of the containing block; shared with all child blocks and tests. Use for expensive operations (importing the tested function, one-time API calls).
- **`BeforeEach`**: Runs before every `It` in the current or any child block. Use for per-test prerequisites (e.g., creating a fresh file).
- **`AfterEach`**: Runs after every `It` in the current or any child block, inside a `finally` block — guaranteed to run even if the test or its setup fails. Placement within the block does not affect ordering; it always runs last. Write teardown defensively (e.g., check `Test-Path` before removing a file that may not have been created).
- **`AfterAll`**: Runs once after the containing `Describe`/`Context` block, guaranteed even on failure. Use for shared cleanup.
- **Multiple setups/teardowns**: Multiple `BeforeAll`/`BeforeEach` run in definition order; `AfterAll`/`AfterEach` run in the opposite order. There can be only ONE setup and ONE teardown of each kind per block.
- **Skipped when no tests run**: Setups and teardowns are skipped when filtering excludes all tests in the block tree.

### Variable Scoping

- Variables defined in `BeforeAll` are available (read-only) to all child blocks and tests, but CANNOT be written back — each test runs in its own scope to stay isolated. Assigning to such a variable inside `It` creates a test-local copy only.
- `BeforeEach`, `It`, and `AfterEach` share the SAME scope — variables defined in `BeforeEach` or `It` are available in `AfterEach` (e.g., to clean up a file path created in the test).

## Assertions (Should)

- **Basic Comparisons**: `-Be`, `-BeExactly`, `-Not -Be`
- **Collections**: `-Contain`, `-BeIn`, `-HaveCount`
- **Numeric**: `-BeGreaterThan`, `-BeLessThan`, `-BeGreaterOrEqual`
- **Strings**: `-Match`, `-Like`, `-BeNullOrEmpty`
- **Types**: `-BeOfType`, `-BeTrue`, `-BeFalse`
- **Files**: `-Exist`, `-FileContentMatch`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlagueHO/CosmosDB](https://github.com/PlagueHO/CosmosDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
