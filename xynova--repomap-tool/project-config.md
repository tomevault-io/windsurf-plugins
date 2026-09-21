---
trigger: always_on
description: Comprehensive testing and code quality rules with anti-cheating protocols
---


# 🤖 **AI REASONING PROTOCOL**

## **Before Taking Any Testing Action:**
**ALWAYS provide step-by-step reasoning that:**
1. States what testing action you're about to take
2. Explains which rules from this document apply
3. Shows how you've considered those rules in your approach
4. Outlines your specific plan and next steps

---

# 🚫 **FORBIDDEN ACTIONS - ANTI-CHEATING PROTOCOLS**

## **Testing Anti-Cheating Rules**
- **DO NOT** create tests that mock core functionality to achieve coverage
- **DO NOT** write tests that only verify return types without testing behavior
- **DO NOT** create tests with hardcoded responses or fake data
- **DO NOT** write tests that only cover happy paths without error conditions
- **DO NOT** create trivial tests that test obvious functionality
- **DO NOT** use mocks for internal business logic (only for external dependencies)

## **Code Quality Anti-Cheating Rules**
- **DO NOT** refactor by simply moving code around without real separation of concerns
- **DO NOT** create modules with artificial separation (e.g., by line count only)
- **DO NOT** implement placeholder logic that returns hardcoded values
- **DO NOT** create fake implementations that don't perform real business logic
- **DO NOT** use deprecated patterns without updating to modern alternatives
- **DO NOT** create modules that can't be tested independently

## **Integration Testing Anti-Cheating Rules**
- **DO NOT** create integration tests that mock everything
- **DO NOT** write integration tests that only verify CLI return codes
- **DO NOT** create integration tests with fake file system operations
- **DO NOT** write integration tests that skip error recovery scenarios
- **DO NOT** create integration tests that don't validate real system behavior

# ✅ **REQUIRED ACTIONS - QUALITY PROTOCOLS**

## 🧪 **TESTING REQUIREMENTS:**
- **ALWAYS** write comprehensive tests for new functionality
- **NEVER** claim tests pass without actually running them
- **NEVER** provide fake test solutions or incomplete test coverage
- **ALWAYS** verify tests run successfully with `make test-unit`
- **ALWAYS** fix failing tests before claiming success
- **ALWAYS** test edge cases and error conditions
- **ALWAYS** ensure test coverage for new code paths
- **CLI TESTS ARE MANDATORY** - CLI is the primary UX, must be thoroughly tested
- **ALWAYS** write tests that use real data and files when possible
- **ALWAYS** include error condition testing (at least 20% of tests)
- **ALWAYS** verify actual behavior, not just return types
- **ALWAYS** include performance constraints in relevant tests
- **ALWAYS** test with real project structures, not artificial ones

## 📁 **FIXTURE REQUIREMENTS:**
- **PREFER static shared fixtures** over dynamic test data generation
- **USE existing test fixtures** from `tests/fixtures/` directory
- **AVOID state-changing operations** in test fixtures (this software doesn't change state)
- **REUSE fixture data** across multiple test cases for consistency
- **CREATE static test files** that represent real project structures
- **USE shared example data** that can be referenced by multiple tests
- **AVOID generating test data** at runtime when static data suffices
- **PREFER deterministic fixtures** that produce consistent results
- **USE existing test repositories** like `tests/fixtures/test-repo/` for integration tests

## 🔍 TESTING WORKFLOW:
1. **Write tests first** for new functionality
2. **Run tests immediately** after any code changes
3. **Fix all failures** before proceeding
4. **Verify end-to-end** functionality works
5. **Never skip testing** - it's mandatory, not optional

## 🐛 **DEBUG TEST NAMING CONVENTIONS:**
- **ALWAYS** prefix debug/investigation tests with `ignore.test_*` 
- **ALWAYS** use this naming pattern for temporary tests created to debug issues
- **ALWAYS** ensure debug tests don't get captured in git commits
- **EXAMPLES**: `ignore.test_debug_import_issue.py`, `ignore.test_investigate_parser.py`
- **PURPOSE**: Keep formal test suite clean while allowing debugging investigations

## ⚠️ **TESTING RULES:**
- **NO FAKE SOLUTIONS**: Every test must actually pass
- **NO SKIPPED TESTS**: All tests must run and pass
- **NO PARTIAL COVERAGE**: Test all new code paths
- **NO ASSUMPTIONS**: Verify everything works, don't guess
- **NO CLI WITHOUT TESTS**: Every CLI command must have comprehensive tests
- **NO FALSE CLAIMS**: Never claim 100% coverage without verifying actual numbers
- **NO BS STATEMENTS**: Always be precise about what is actually tested vs. claimed
- **NO OVER-MOCKING**: Tests must use real data and validate actual behavior
- **NO PLACEHOLDER IMPLEMENTATIONS**: Business logic must be real, not fake
- **NO SUPERFICIAL REFACTORING**: Modules must have real separation of concerns

## 🚀 FINAL VALIDATION REQUIREMENTS:
- **ALWAYS** run `make ci` before finishing work
- **ALWAYS** fix any CI errors before claiming completion
- **ALWAYS** ensure good real coverage for all updates (>80% for new code)
- **ALWAYS** verify integration tests pass (not just unit tests)
- **ALWAYS** check that new functionality works end-to-end
- **NEVER** finish work with failing CI or poor coverage

## 🖥️ CLI TESTING REQUIREMENTS:
- **CLI IS THE UX**: Every CLI command must be thoroughly tested

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
