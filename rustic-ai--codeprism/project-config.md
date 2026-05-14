---
trigger: always_on
description: **Purpose:** Enforce complete implementation standards to prevent stub code, placeholders, and incomplete work from being committed. Based on lessons learned from MCP Test Harness audit where 60%+ of "complete" work was actually placeholder implementations.
---

# Complete Implementation Standards - Zero Placeholder Policy

**Purpose:** Enforce complete implementation standards to prevent stub code, placeholders, and incomplete work from being committed. Based on lessons learned from MCP Test Harness audit where 60%+ of "complete" work was actually placeholder implementations.

**When to use:** ALWAYS - Every commit, every implementation, every code review.

## Absolute Prohibitions Before Commit

**Rule: ZERO tolerance for placeholder implementations in committed code.**
Why: Placeholder code creates false sense of completion, misleads project status, and compounds technical debt. Recent audit revealed 50+ TODO comments and extensive stub implementations in supposedly "complete" features.

### **Prohibited Code Patterns (Must Fix Before Commit)**

```rust
// ❌ NEVER COMMIT - Placeholder implementations
// TODO: Implement actual functionality
// FIXME: This needs proper implementation
// Placeholder implementation
// Stub implementation
// For now, provide stub implementation
fn placeholder_function() { todo!() }
fn function() { unimplemented!() }

// ❌ NEVER COMMIT - Mock responses in production code
async fn get_data() -> Result<Data> {
    // TODO: Replace with actual API call
    Ok(Data::mock()) // ❌ Mock data in production path
}

// ❌ NEVER COMMIT - Commented out TODO sections
// TODO: Add error handling
// TODO: Implement validation
// TODO: Add performance monitoring

// ❌ NEVER COMMIT - Placeholder test cases
#[test]
fn test_feature() {
    // TODO: Write actual test
    assert!(true); // ❌ Meaningless assertion
}

// ❌ NEVER COMMIT - Hardcoded placeholder values
let memory_mb = 0; // Placeholder - would get actual memory usage
let result = "placeholder_response"; // TODO: Get real response
```

### **Acceptable Temporary Patterns (With Strict Rules)**

```rust
// ✅ ACCEPTABLE - Development-only features with clear annotations
#[cfg(feature = "development")]
fn development_mock() -> Data {
    // This is intentionally a mock for development/testing only
    Data::mock()
}

// ✅ ACCEPTABLE - Explicit future work with GitHub issues
// NOTE: Feature XYZ not yet implemented - tracked in Issue #123
// This function returns default behavior until Issue #123 is completed

// ✅ ACCEPTABLE - Dead code that will be removed
#[allow(dead_code)] // Will be used in Issue #456 implementation
struct FutureFeature { ... }

// ✅ ACCEPTABLE - Intentional unimplemented with clear context
fn experimental_feature() -> Result<()> {
    // This feature is intentionally not implemented until design review
    // completes. See design doc: docs/experimental-feature.md
    Err("Feature not yet available".into())
}
```

## Pre-Commit Verification Checklist

**Rule: Complete this checklist before EVERY commit. No exceptions.**

### **Code Completeness Verification**
```bash
# 1. Search for prohibited patterns (MUST return zero results)
grep -r "TODO" src/ --include="*.rs"
grep -r "FIXME" src/ --include="*.rs" 
grep -r "placeholder" src/ --include="*.rs"
grep -r "stub" src/ --include="*.rs"
grep -r "unimplemented!" src/ --include="*.rs"
grep -r "todo!()" src/ --include="*.rs"

# 2. Search for placeholder test patterns
grep -r "assert!(true)" tests/ --include="*.rs"
grep -r "// TODO:" tests/ --include="*.rs"
```

### **Implementation Verification Checklist**
- [ ] **Zero TODO/FIXME comments** in committed code
- [ ] **Zero placeholder implementations** (functions that return mock/dummy data)
- [ ] **Zero stub functions** (functions with empty bodies or `unimplemented!()`)
- [ ] **All functions have real implementations** that perform their intended purpose
- [ ] **All tests actually test functionality** (no `assert!(true)` placeholders)
- [ ] **All error paths are implemented** (not just success paths)
- [ ] **All configuration options are functional** (not just parsed but ignored)
- [ ] **All performance requirements are met** (not estimated or placeholder metrics)

### **Quality Verification Checklist**
- [ ] **All public APIs have comprehensive rustdoc** with working examples
- [ ] **All functions handle errors appropriately** (not just `.unwrap()` or `.expect()`)
- [ ] **All performance-critical paths are optimized** (not just basic implementations)
- [ ] **All security requirements are implemented** (not just documented)
- [ ] **All integration points are functional** (not mocked in production code)

## Implementation Standards by Code Type

### **Function Implementation Standards**

```rust
// ❌ INCOMPLETE - Function exists but doesn't work
pub fn analyze_performance(code: &str) -> PerformanceMetrics {
    // TODO: Implement actual analysis
    PerformanceMetrics::default()
}

// ✅ COMPLETE - Function fully implements its contract
pub fn analyze_performance(code: &str) -> Result<PerformanceMetrics> {
    let ast = parse_code(code)?;
    let complexity = calculate_complexity(&ast)?;
    let memory_usage = estimate_memory_usage(&ast)?;
    let execution_time = estimate_execution_time(&ast)?;
    
    Ok(PerformanceMetrics {
        complexity,
        memory_usage,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
