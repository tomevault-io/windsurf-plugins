---
trigger: always_on
description: This document defines code review guidelines for the Magda project, a JUCE/C++ audio DAW framework. Reviews should focus on critical bugs, performance improvements, and code readability to ensure high-quality, maintainable code.
---

# GitHub Copilot Review Instructions for Magda

This document defines code review guidelines for the Magda project, a JUCE/C++ audio DAW framework. Reviews should focus on critical bugs, performance improvements, and code readability to ensure high-quality, maintainable code.

## Review Focus Areas

### 1. Critical Bugs (Highest Priority)
- **Memory Safety**: Check for memory leaks, use-after-free, buffer overflows, and dangling pointers
- **Thread Safety**: Verify proper synchronization in audio processing code, especially in real-time contexts
- **Audio Buffer Issues**: Look for buffer over/underruns, incorrect sample rate handling, and improper buffer management
- **Resource Management**: Ensure proper RAII patterns, especially with JUCE objects and audio resources
- **Null Pointer Dereferences**: Check for proper null checks before dereferencing pointers
- **Integer Overflow/Underflow**: Verify arithmetic operations, especially in sample calculations and buffer indexing
- **Plugin Lifecycle Issues**: Ensure proper plugin initialization, cleanup, and state management
- **JUCE Message Thread Violations**: Verify UI operations happen on the message thread, audio code runs on audio thread

### 2. Performance Improvements (High Priority)
- **Real-time Audio Safety**: Flag allocations, locks, or blocking operations in audio processing callbacks
- **Unnecessary Copies**: Identify opportunities to use move semantics, const references, or std::string_view
- **Algorithm Efficiency**: Look for O(n²) or worse algorithms that could be optimized
- **Cache Locality**: Suggest improvements to data structure layout for better cache performance
- **SIMD Opportunities**: Identify vectorizable loops in DSP code
- **Excessive Memory Allocations**: Flag unnecessary heap allocations, especially in hot paths
- **Lock Contention**: Identify potential bottlenecks from lock contention in multi-threaded code
- **Redundant Operations**: Point out redundant calculations or repeated work

### 3. Code Readability & Clarity (Medium Priority)
- **Naming Conventions**: Ensure consistency with project style (camelCase for functions/variables, CamelCase for classes)
- **Function Complexity**: Flag functions exceeding 80 lines or with cognitive complexity > 15
- **Magic Numbers**: Suggest named constants for literal values, except obvious cases (0, 1, 2)
- **Clear Intent**: Ensure code is self-documenting; suggest refactoring when intent is unclear
- **JUCE Idioms**: Verify proper use of JUCE patterns (e.g., MessageManager, ValueTree, listeners)
- **Modern C++20**: Encourage use of C++20 features where appropriate (concepts, ranges, coroutines if applicable)
- **Error Handling**: Verify appropriate error handling strategies
- **Documentation**: Check that complex algorithms or non-obvious code have explanatory comments
- **TODO Comments**: Read and respect TODO comments in the code being reviewed. If a TODO indicates planned work or known limitations, consider this context when reviewing related changes. Don't request changes that conflict with or duplicate TODO items.

## What NOT to Focus On

- **Style Issues Already Covered by clang-format**: Don't comment on formatting that will be auto-fixed
- **Minor Optimizations**: Avoid nitpicking micro-optimizations that don't impact performance measurably
- **Personal Preferences**: Don't suggest changes based solely on personal style preferences
- **Trivial Issues**: Skip commenting on very minor issues that don't affect functionality or maintainability
- **Pre-existing Issues**: Focus on the code being changed, not unrelated existing code
- **Nitpicking**: Do not nitpick. Focus on substantive issues that genuinely affect correctness, performance, or maintainability. If a comment wouldn't block a merge, it's probably not worth making.

## Resolving Previous Review Comments

- **Mark resolved issues as resolved**: When a subsequent PR push addresses feedback from a previous review iteration, those comments MUST be marked as resolved. Do not leave stale unresolved comments hanging.
- **Don't re-raise resolved issues**: If a concern from a previous review has been addressed in the current diff, do not re-raise it or raise a variation of it.
- **Acknowledge progress**: When re-reviewing after changes, focus only on what's new or still unresolved. Don't repeat feedback that has already been acted upon.

## Refactoring Guidelines (for Copilot Coding Agent)

When assigned refactoring issues, follow these rules strictly:

### Do NOT
- **Extract constants from obvious values** — `0.0`, `0.5`, `1.0`, `2.0`, `60.0`, `PI`, pixel padding values (4, 8, 12, 20), font sizes, etc. are not magic numbers. They are clearer inline than as `FULL_RANGE`, `HALF_CYCLE`, `SECONDS_PER_MINUTE`, or `kOuterPadding`.
- **Split functions under 50 NLOC** — Small functions don't need decomposition regardless of CCN.
- **Create PRs that only rename magic numbers or extract constants** — This is busywork, not refactoring.
- **Split one-liner expressions into separate functions** — `return 1.0f - phase;` does not need a `generateReverseSawWave()` wrapper.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Conceptual-Machines/magda-core](https://github.com/Conceptual-Machines/magda-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
