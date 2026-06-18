---
trigger: always_on
description: Clean up feature flags from codebases. Removes flag checks, makes conditional code unconditional (graduate = keep enabled code, drop = remove enabled code), handles cascade cleanup via static analysis, cleans tests/config/docs. Use when the user wants to clean up, remove, graduate, drop, retire, or sunset a feature flag.
---


# Feature Flag Cleanup Skill

> *"A proper cleanup leaves no trace of the old regime, Milord."*

Trigger phrases: "clean up feature flag", "remove feature flag", "graduate feature flag", "drop feature flag", "flag cleanup", "retire flag", "sunset flag"

---

## Input Requirements

Before starting, the skill MUST collect these inputs from the user:

| Input | Required | Description |
|-------|----------|-------------|
| **Flag name** | ✅ | The string identifier (e.g., `flag_mod_xpm_travel`) |
| **Action** | ✅ | `graduate` (keep enabled code) or `drop` (remove enabled code) |
| **Repository path** | ✅ | Path to the repository to clean |

### Action Definitions

- **Graduate** → The feature is permanently enabled. Remove the flag check, KEEP the code inside the `true`/enabled branch. Delete the `false`/disabled branch.
- **Drop** → The feature is being removed. Remove the flag check, DELETE the code inside the `true`/enabled branch. Keep the `false`/disabled/fallback branch.

---

## Execution Steps

### Phase 1: Discovery

> Discovery is reliable because flags are ALWAYS defined as string constants in a dedicated constants file per module. No confirmation checkpoint needed — proceed directly to transformation.

1. **Find the flag constant definition**
   - `grep` for the flag name string literal (e.g., `"flag_mod_xpm_travel"`) across the repository
   - This will locate the constant file/class (e.g., `FlagConstants.dart`, `FeatureFlags.kt`, `FeatureFlags.swift`)
   - Note the constant variable name (e.g., `kFlagModXpmTravel`, `FLAG_MOD_XPM_TRAVEL`)

2. **Trace all references via the constant name**
   - `grep` for the constant variable name across the entire repo (including `android/` and `ios/` directories for Flutter projects)
   - This gives the complete and exhaustive list of usage sites

3. **Categorize usages** (for transformation routing)
   - **Code conditionals**: if/else, ternary, switch, widget conditional rendering → Phase 2
   - **Config/registry entries**: flag registration, default value definitions → Phase 4
   - **Test references**: test mocks, test setup, flag-specific test cases → Phase 5
   - **Documentation**: comments, README references, changelog entries → Phase 6

4. **Proceed immediately** — no user confirmation needed. Results shown in PR.

### Phase 2: Code Transformation

For each code conditional usage, apply the correct transformation based on the action:

- **Graduate** → keep the `true`/enabled branch, remove the `false`/disabled branch
- **Drop** → keep the `false`/disabled branch, remove the `true`/enabled branch

> **📖 See [cleanup-patterns.md](./cleanup-patterns.md)** for detailed transformation examples covering all 10 patterns: simple if/else, no-else blocks, collection-if, entire file/class, nested conditions, early returns, ternary expressions, variable assignments, Kotlin when expressions, and SwiftUI conditional views. Also includes boolean simplification rules and edge cases.

**Key principles:**
- Apply boolean simplification when flag is part of a compound expression
- Trace indirect usages (flag stored in variable → find all variable usages)
- If unsure about a transformation → skip it, leave a TODO comment
- Only resolve the TARGET flag — leave other flags/conditions untouched

### Phase 3: Cascade Cleanup

> Strategy: Leverage static analysis tools (not manual reasoning) to find cascade issues. Iterate until the analyzer reports no new warnings.

**Approach: Analyzer-Driven Iterative Cleanup**

```
loop:
  1. Run static analysis (`dart analyze`, Kotlin compiler, Swift build)
  2. Parse warnings/errors related to files already modified:
     - Unused imports
     - Unused variables/fields
     - Unused parameters
     - Unreachable code
     - Empty bodies/blocks
  3. If new warnings found → fix them
  4. If warnings create MORE orphaned code → repeat from step 1
  5. If no new warnings → exit loop (clean state achieved)
```

**What to fix in each iteration:**
1. **Dead imports**: Remove `import` lines that are no longer referenced
2. **Unused variables/fields**: Remove declarations that lost their only consumer
3. **Empty blocks**: Remove empty methods, empty classes, empty if-bodies
4. **Orphaned files** (drop action): If removing a class leaves a file with nothing in it → delete file
5. **Unused parameters**: Remove from private methods. For public APIs → flag in PR as "⚠️ breaking change — needs manual review"

**Stopping conditions:**
- Analyzer reports zero new warnings on touched files → done
- Only remaining warnings are pre-existing (existed before cleanup) → done
- Iteration count exceeds 10 → stop, report remaining issues in PR description

**Critical: Ignore pre-existing warnings.** Before starting cleanup, capture the baseline analyzer output. Only act on NEW warnings that appeared after the flag removal. Never fix unrelated pre-existing issues — that's scope creep and pollutes the PR diff.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iqbal-mekari/feature-flag-cleanup](https://github.com/iqbal-mekari/feature-flag-cleanup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
