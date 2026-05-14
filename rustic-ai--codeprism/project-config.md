---
trigger: always_on
description: Enforces proper TDD workflow planning to prevent scope limitation and incomplete test coverage
---


# TDD Workflow Planning Rules

## Critical TDD Planning Mistakes to Avoid

### ❌ Scope Limitation Failures

**NEVER limit TDD scope to individual functions - always consider the entire system.**

```rust
// ❌ BAD: Micro-TDD that misses the big picture
// Writing test for just one tool function without considering:
// - Integration with other tools
// - Error handling across the system  
// - Performance under load
// - Edge cases in the broader context

#[tokio::test]
async fn test_single_tool_basic_case() {
    // Narrow focus that misses system-wide requirements
}
```

```rust
// ✅ GOOD: Macro-TDD that considers entire ecosystem
// Plan comprehensive test coverage first:

// 1. Individual tool functionality (26 tools)
#[tokio::test] async fn test_all_tools_individually() { }

// 2. Tool integration and workflow
#[tokio::test] async fn test_tool_interaction_scenarios() { }

// 3. Error handling across system
#[tokio::test] async fn test_system_error_recovery() { }

// 4. Performance and scalability  
#[tokio::test] async fn test_concurrent_tool_execution() { }

// 5. Edge cases and boundary conditions
#[tokio::test] async fn test_edge_cases_comprehensive() { }
```

### ❌ Planning Sequence Failures

**NEVER start implementation without comprehensive test planning.**

**Wrong Sequence (What I did):**
```
1. Implement tool functionality ❌
2. Write minimal test to pass ❌  
3. Move to next feature ❌
4. Discover massive testing gaps later ❌
```

**Correct Sequence:**
```
1. ✅ Study existing test patterns (codeprism-mcp-server)
2. ✅ Analyze comprehensive requirements  
3. ✅ Plan ALL test scenarios upfront
4. ✅ Write ALL failing tests first
5. ✅ Implement functionality to pass tests
6. ✅ Refactor while maintaining coverage
```

## Mandatory Pre-Implementation Analysis

### 1. Study Existing Patterns FIRST

**Before implementing ANY feature:**

```bash
# REQUIRED: Analyze existing comprehensive tests
grep -r "#\[tokio::test\]" crates/codeprism-mcp-server/src/ --include="*.rs" | wc -l
# Expected: 80+ tests to study

# REQUIRED: Identify test categories  
grep -r "async fn test_.*_tool" crates/codeprism-mcp-server/src/
# Study individual tool test patterns

# REQUIRED: Map error handling patterns
grep -r "test_.*_error\|test_.*_invalid" crates/codeprism-mcp-server/src/
# Understand error test coverage

# REQUIRED: Study edge case patterns  
grep -r "test_.*_edge\|test_.*_boundary" crates/codeprism-mcp-server/src/
# Learn edge case testing approaches
```

### 2. Comprehensive Requirement Analysis

**Create test requirement matrix BEFORE coding:**

| **Component** | **Success Tests** | **Error Tests** | **Edge Tests** | **Integration Tests** | **Performance Tests** |
|---------------|-------------------|-----------------|----------------|----------------------|----------------------|
| Tool 1: provide_guidance | ✅ Required | ✅ Required | ✅ Required | ✅ Required | ✅ Required |
| Tool 2: optimize_code | ✅ Required | ✅ Required | ✅ Required | ✅ Required | ✅ Required |
| Tool 3: analyze_dependencies | ✅ Required | ✅ Required | ✅ Required | ✅ Required | ✅ Required |
| ... (All 26 tools) | ✅ Required | ✅ Required | ✅ Required | ✅ Required | ✅ Required |
| **Total Required** | **26 tests** | **26 tests** | **26 tests** | **26 tests** | **26 tests** |

### 3. Test Planning Documentation

**Document comprehensive test plan BEFORE implementation:**

```rust
// REQUIRED: Create test plan document before coding
/*
TEST PLAN: codeprism-mcp-server Tool Implementation

SCOPE: All 26 tools with comprehensive coverage
REFERENCE: codeprism-mcp-server test patterns

COVERAGE REQUIREMENTS:
1. Individual Tool Tests (26 tests)
   - Each tool: success, error, edge cases
   - Parameter validation
   - Output format verification

2. Integration Tests (10+ tests)  
   - Tool interaction workflows
   - Server lifecycle management
   - Resource management

3. Error Handling Tests (20+ tests)
   - Invalid parameters for each tool
   - System failures and recovery
   - Timeout and resource exhaustion

4. Edge Case Tests (15+ tests)
   - Boundary conditions
   - Empty/large inputs
   - Concurrent access

5. Performance Tests (5+ tests)
   - Response time requirements
   - Memory usage validation
   - Concurrent execution stress

TOTAL TARGET: 76+ tests (comprehensive coverage)
*/
```

## TDD Implementation Workflow

### Phase 1: Comprehensive Test Creation (RED)

**Write ALL failing tests before ANY implementation:**

```rust
// 1. Individual tool tests (ALL 26 tools)
#[tokio::test] async fn test_provide_guidance_success() { todo!() }
#[tokio::test] async fn test_provide_guidance_error() { todo!() }  
#[tokio::test] async fn test_optimize_code_success() { todo!() }
#[tokio::test] async fn test_optimize_code_error() { todo!() }
// ... Continue for all 26 tools

// 2. Integration tests
#[tokio::test] async fn test_full_mcp_workflow() { todo!() }
#[tokio::test] async fn test_concurrent_tool_execution() { todo!() }

// 3. Error handling tests
#[tokio::test] async fn test_invalid_tool_parameters() { todo!() }
#[tokio::test] async fn test_system_error_recovery() { todo!() }

// 4. Edge case tests
#[tokio::test] async fn test_boundary_conditions() { todo!() }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
