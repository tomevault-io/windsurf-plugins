---
trigger: always_on
description: General coding practices and agent interaction rules applicable across the organization, regardless of language.
---

# Organization General Practices

## Core Coding Philosophy
– **Simplicity:** Always prefer simple, understandable solutions.
– **DRY (Don't Repeat Yourself):** Avoid duplication. Check for existing similar code/functionality before writing new code.
– **Cleanliness:** Keep the codebase clean, well-organized, and maintainable.
– **Environment Awareness:** Write code that correctly handles different environments (e.g., dev, test, prod).
– **Focused Changes:** Only make changes that are requested or clearly understood and directly related to the task. Avoid implementing unrelated improvements or refactors without explicit instruction (see Agent Task Scope below).
– **Incremental Improvement:** When fixing bugs, avoid introducing new patterns/technologies unless necessary after exhausting existing options. If a new pattern replaces an old one, remove the old implementation.
– **File Size:** Avoid overly long files. Consider refactoring files exceeding 200–300 lines
- **Function Size:** Avoid overly long functions.  Break up large functions using helper functions that encapsulate related functionality into smaller and more focused pieces.
– **Script Usage:** Avoid writing one-off scripts if a more integrated solution is feasible.

## Function Design & Complexity Management
- **Extract Helper Methods:** When a function exceeds ~30-40 lines or has distinct logical sections (e.g., validation, processing, output), extract helper methods. If you have numbered steps in comments (Step 1, Step 2, etc.), consider if each step should be its own function.
- **Single Responsibility:** Each function should do ONE thing well. A function should either answer one question OR perform one action, not both.
- **Fail-Fast Pattern:** Check error conditions early and return/throw immediately. Don't carry invalid state through the entire function. This makes the happy path clearer and reduces nesting.
- **Avoid Redundant Checking:** Don't check the same condition multiple times across different functions. Check once at the boundary, then pass validated data to inner functions.
- **Clear Method Contracts:** Helper methods should clearly indicate via their name and signature what they assume (e.g., `calculateValueWithValidPrices` assumes prices are already validated).

## Efficient Code Patterns
- **Don't Compute What You Don't Need:** Avoid building intermediate data structures (arrays, objects) just to check a property. For example, don't build an array just to check if it's empty - use a boolean check directly.
- **Choose Appropriate Loops:** Use `for` loops when iteration count is known upfront, `while` for conditional iteration. Modern `for` loops are often cleaner than `while` with manual increment.
- **Early Exit Strategies:** Prefer methods that can exit early (`.some()`, `.every()`, `.find()`) over methods that process everything (`.filter().length`, `.map()`) when you just need a boolean or single result.
- **Separation of Concerns:** Separate "what failed" (detailed logging) from "should we continue" (control flow) logic. Log details where the failure is detected, make decisions based on simple booleans.
- **Push Computation to Data Layer:** When working with databases or external services:
  - Prefer aggregating/sorting/filtering at the source over fetching-then-processing
  - Example: SQL `SUM()` instead of fetching all rows to sum in JavaScript
  - Example: API filtering parameters instead of fetching all then filtering
  - This reduces memory usage, network transfer, and processing time

## Data Handling
– **Mocking:** Mock data *only* for automated tests. Never use mocked or stubbed data in development or production environments.
– **Secrets:** Ensure secrets, API keys, or passwords are *never* committed to the repository. Use environment variables or secure secret management solutions.

## Tooling Interaction
– **Non-Interactive Execution:** When using command-line tools or scripts, ensure they run in non-interactive mode to prevent hangs (e.g., append `| cat` to commands like `git log` if needed, use appropriate flags).

## Documentation
– **Inline Documentation:** Maintain excellent, thorough inline documentation (e.g., comments for functions, methods, types, classes, and complex logic).
– **READMEs:** When editing README files, conform to the [standard-readme](mdc:https:/github.com/RichardLitt/standard-readme) specification.
– **CRITICAL - No Temporal or Comparative Comments:** 
  - **NEVER** use words like "new", "optimized", "efficient", "replaces", "improved", "better", "faster", "atomic" in comments or TSDoc
  - **NEVER** reference what the code replaces or how it compares to previous versions
  - **NEVER** mention implementation optimizations (e.g., "avoids N+1", "uses atomic operations", "parallelized")
  - **DO** describe WHAT the method does and its contract/behavior
  - **DO** focus on the current functionality without historical context
  - Example: ❌ BAD: "Optimized method that efficiently fetches users avoiding N+1 queries"
  - Example: ✅ GOOD: "Fetches users with their associated posts in a single query"

## Security

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recallnet/js-recall](https://github.com/recallnet/js-recall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
