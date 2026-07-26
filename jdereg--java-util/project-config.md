---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 CRITICAL RULES - READ FIRST 🚨

**BEFORE doing ANYTHING else, understand these NON-NEGOTIABLE requirements:**

### MANDATORY FULL TEST SUITE VALIDATION

**EVERY change, no matter how small, MUST be followed by running the full test suite:**

```bash
mvn clean test
```

**ALL 10,000+ tests MUST pass before:**
- Moving to the next issue/file/task
- Committing any changes  
- Asking for human approval
- Starting any new work

**If even ONE test fails:**
- Stop immediately
- Fix the failing test(s)
- Run the full test suite again
- Only proceed when ALL tests pass

**This rule applies to ANY code modification and is MORE IMPORTANT than the actual change itself.**

### MANDATORY HUMAN APPROVAL FOR COMMITS

**NEVER commit without explicit "Y" or "Yes" approval from human.**

### MANDATORY HUMAN APPROVAL FOR DEPLOYMENT

**NEVER deploy without explicit human approval. Always ask for permission before starting any deployment process.**

## 🎯 WORK PHILOSOPHY - INCREMENTAL ATOMIC CHANGES 🎯

**Mental Model: Work with a "List of Changes" approach**

### The Change Hierarchy
- **Top-level changes** (e.g., "Fix security issues in DateUtilities")
  - **Sub-changes** (e.g., "Fix ReDoS vulnerability", "Fix thread safety")
    - **Sub-sub-changes** (e.g., "Limit regex repetition", "Add validation tests")

### Workflow for EACH Individual Change
1. **Pick ONE change** from any level (top-level, sub-change, sub-sub-change)
2. **Implement the change**
   - During development: Use single test execution for speed (`mvn test -Dtest=SpecificTest`)
   - Iterate until the specific functionality works
3. **When you think the change is complete:**
   - **MANDATORY**: Run full test suite: `mvn clean test`
   - **ALL 10,000+ tests MUST pass**
   - **If ANY test fails**: Fix immediately, run full tests again
4. **Once ALL tests pass:**
   - Ask for commit approval: "Should I commit this change? (Y/N)"
   - Human approves, commit immediately
   - Move to next change in the list

### Core Principles
- **Start work**: At the start of new work, create a "Todo" list.
- **Chat First**: As a general work guideline, when starting a new Todo list, or a feature idea, always "chat first, get agreement from human, then code."
- **Minimize Work-in-Process**: Keep delta between local files and committed git files as small as possible
- **Always Healthy State**: Committed code is always in perfect health (all tests pass)
- **Atomic Commits**: Each commit represents one complete, tested, working change
- **Human Controls Push**: Human decides when to push commits to remote

**🎯 GOAL: Each change is complete, tested, and committed before starting the next change**

## ADDITIONAL TESTING REQUIREMENTS

**CRITICAL BUILD REQUIREMENT**: The full maven test suite MUST run all 10,000+ tests. If you see only ~10,000 tests, there is an OSGi or JPMS bundle issue that MUST be fixed before continuing any work. Use `mvn -Dbundle.skip=true test` to bypass bundle issues during development, but the underlying bundle configuration must be resolved.

**CRITICAL TESTING REQUIREMENT**: When adding ANY new code (security fixes, new methods, validation logic, etc.), you MUST add corresponding JUnit tests to prove the changes work correctly. This includes:
- Testing the new functionality works as expected
- Testing edge cases and error conditions  
- Testing security boundary conditions
- Testing that the fix actually prevents the vulnerability
- All new tests MUST pass along with the existing 10,000+ tests
## Build Commands

**Maven-based Java project with JDK 8 compatibility**

- **Build**: `mvn compile`
- **Test**: `mvn test`
- **Package**: `mvn package`
- **Install**: `mvn install`
- **Run single test**: `mvn test -Dtest=ClassName`
- **Run tests with pattern**: `mvn test -Dtest="*Pattern*"`
- **Clean**: `mvn clean`
- **Generate docs**: `mvn javadoc:javadoc`

## Architecture Overview

**java-util** is a high-performance Java utilities library focused on memory efficiency, thread-safety, and enhanced collections. The architecture follows these key patterns:

### Core Structure
- **Main package**: `com.cedarsoftware.util` - Core utilities and enhanced collections
- **Convert package**: `com.cedarsoftware.util.convert` - Comprehensive type conversion system
- **Cache package**: `com.cedarsoftware.util.cache` - Caching strategies and implementations

### Key Architectural Patterns

**Memory-Efficient Collections**: CompactMap/CompactSet dynamically adapt storage structure based on size, using arrays for small collections and switching to hash-based storage as they grow.

**Null-Safe Concurrent Collections**: ConcurrentHashMapNullSafe, ConcurrentNavigableMapNullSafe, etc. extend JDK concurrent collections to safely handle null keys/values.

**Dynamic Code Generation**: CompactMap/CompactSet use JDK compiler at runtime to generate optimized subclasses when builder API is used (requires full JDK).

**Converter Architecture**: Modular conversion system with dedicated conversion classes for each target type, supporting thousands of built-in conversions between Java types.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdereg/java-util](https://github.com/jdereg/java-util) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
