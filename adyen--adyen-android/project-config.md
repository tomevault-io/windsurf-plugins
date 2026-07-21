---
trigger: always_on
description: This document outlines important patterns, practices, and rules to follow when working with the Adyen Android SDK codebase.
---

# Agent Guidelines for Adyen Android SDK

This document outlines important patterns, practices, and rules to follow when working with the Adyen Android SDK codebase.

## Core Principles

### 1. Never Make Assumptions
**ALWAYS ask questions if you are uncertain about:**
- Code patterns or conventions
- Expected behavior
- Implementation details
- Migration strategies
- Whether a change is needed or correct

**Do not proceed with uncertain changes.** It's better to ask and get clarification than to implement incorrect solutions.

### 2. Always Work with a Plan

**CRITICAL: Create plan FIRST, implement AFTER approval:**
- **NEVER start implementation without a plan**
- First, create a comprehensive implementation plan document (e.g., `*_IMPLEMENTATION_PLAN.md`)
- Present the plan to the user for review and approval
- Only after the plan is approved, begin implementation
- Do not execute any code changes during the planning phase

**Before starting any task:**
- Ensure there is an implementation plan document (e.g., `*_IMPLEMENTATION_PLAN.md`)
- If no plan exists, guide the user to create one before proceeding with implementation
- Break down the work into clear, manageable phases
- Identify dependencies, risks, and testing strategy upfront

**When working on multi-phase tasks with a plan document:**
- Update the plan file as you complete tasks/phases
- Mark checkboxes as completed
- Add notes about any deviations or discoveries
- This helps if work needs to be continued later or by someone else
- **Do not commit plan files to the repository** as they are temporary working documents. They should only be committed manually if specifically needed.

**Commit workflow - CRITICAL:**
- **Complete one phase fully before moving to the next**
- After completing a phase, use the `android-commit` skill (`.agents/skills/android-commit.md`) to commit
- Never accumulate multiple phases in a single commit
- Each commit should represent a logical, complete unit of work
- This ensures work can be reviewed incrementally and rolled back if needed

### 3. Test-Driven Development

**Write tests first, then make them green:**
- Before implementing new functionality, write the tests that define the expected behavior
- Follow existing test patterns in the codebase to understand our testing structure
- Search for similar tests to understand naming conventions and structure

**Test structure example:**
```kotlin
@Test
fun `when holder name is empty then validation fails`() {
    // GIVEN
    val input = ""
    
    // WHEN
    val result = validator.validate(input)
    
    // THEN
    assertFalse(result.isValid)
}
```

**Ensure test coverage before refactoring:**
- Always ensure there is adequate test coverage before making any internal refactors
- If tests don't exist, write them first to capture current behavior
- This ensures refactoring doesn't introduce regressions

**Pause and verify between big steps:**
- After completing significant changes, pause to run tests
- Ensure all tests that cover your code changes pass before proceeding
- Don't accumulate too many changes without verification

## Implementation Guidelines

### Public API Changes

**Default to internal visibility for all new code:**
- Classes, functions, properties should be `internal` unless they need to be public
- If a class/function is used across modules, use `@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)` annotation
- Only make things `public` if they are part of the public API
- This helps maintain a clean public API surface and prevents accidental exposure

**Example:**
```kotlin
// Default - internal visibility
internal class CardViewStateFactory { }

// Used across modules - restrict to library group
@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
class SomeSharedClass { }

// Public API
class CardConfiguration { }
```

**When making public API changes:**

**Update API dump files:**
- When you intentionally change the public API, run `./gradlew apiDump` (or `./gradlew :<module>:apiDump`) to regenerate the `.api` files
- These files document the public API surface and must be included in your commit
- Do **not** run `apiDump` automatically — only run it after confirming the API change is intentional. This ensures accidental API changes are caught by `apiCheck` during verification.

**Carefully review all changes:**
- Review each change carefully to ensure it's necessary and correct. If in doubt, ask questions.

**Never make breaking changes without discussion:**
- Breaking changes can happen through removed OR modified code
- **Do not proceed with breaking changes until discussed and confirmed with the user**
- Breaking changes should only be done in a major release
- Always ask during planning phase whether breaking changes are acceptable for the current work

**Ensure changes are necessary:**
- Even non-breaking changes can make the API more complex
- Try to find a solution without any public API changes
- If unsure whether something should be public or not, make it internal or annotate with `@RestrictTo`
- We can always make things public later, but making things internal causes a breaking change

**Maintain consistency:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Adyen/adyen-android](https://github.com/Adyen/adyen-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
