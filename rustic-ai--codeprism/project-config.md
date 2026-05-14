---
trigger: always_on
description: Enforces comprehensive test coverage requirements and validation standards
---


# Test Coverage Requirements and Validation

## Mandatory Coverage Standards

### ❌ Coverage Failures That Led to Testing Gaps

**The codeprism-mcp-server testing gap (6 tests vs 80+ needed) occurred because:**

1. **No systematic coverage planning** - Ad hoc test writing instead of comprehensive planning
2. **No comparison to reference implementation** - Failed to study comprehensive testing patterns  
3. **No coverage validation** - No enforcement of minimum coverage thresholds
4. **Scope limitation** - Testing individual functions instead of entire system functionality

### ✅ Required Coverage Standards

**Minimum coverage targets for comprehensive testing:**

| **Coverage Type** | **Target** | **Validation Method** | **Reference** |
|-------------------|------------|----------------------|---------------|
| **Overall Test Coverage** | **90%+** | `cargo tarpaulin --min 90` | [codeprism-mcp-server: comprehensive coverage](mdc:crates/codeprism-mcp-server/src/) |
| **Tool-Specific Tests** | **100%** (26/26 tools) | Individual test per tool | [Tool test patterns](mdc:crates/codeprism-mcp-server/src/server.rs) |
| **Error Handling Tests** | **100%** of error paths | Error scenario coverage | [Error test examples](mdc:crates/codeprism-mcp-server/src/integration_test.rs) |
| **Integration Tests** | **80%** of workflows | End-to-end validation | [Integration patterns](mdc:tests/test_mcp_test_harness_end_to_end.rs) |
| **Edge Case Tests** | **75%** of boundaries | Boundary condition testing | [Edge case patterns](mdc:crates/codeprism-mcp-server/src/server.rs) |

## Coverage Validation Requirements

### Pre-Commit Coverage Checks

**MANDATORY: Run before every commit**

```bash
#!/bin/bash
# .git/hooks/pre-commit

set -e

echo "🔍 Validating test coverage requirements..."

# 1. Check total test count (must match reference implementation)
CURRENT_TESTS=$(grep -r "#\[tokio::test\]" crates/codeprism-mcp-server/src/ --include="*.rs" | wc -l)
REFERENCE_TESTS=80  # Based on comprehensive testing analysis
MIN_REQUIRED_TESTS=76

if [ "$CURRENT_TESTS" -lt "$MIN_REQUIRED_TESTS" ]; then
    echo "❌ COVERAGE FAILURE: Only $CURRENT_TESTS tests found, need $MIN_REQUIRED_TESTS+"
    echo "   Reference implementation has $REFERENCE_TESTS tests"
    echo "   Add missing tests before committing"
    exit 1
fi

# 2. Check tool-specific coverage (all 26 tools must have tests)
TOOLS_WITH_TESTS=$(grep -r "test_.*_tool\|test_provide_guidance\|test_optimize_code\|test_analyze_" crates/codeprism-mcp-server/src/ --include="*.rs" | wc -l)
REQUIRED_TOOL_TESTS=26

if [ "$TOOLS_WITH_TESTS" -lt "$REQUIRED_TOOL_TESTS" ]; then
    echo "❌ TOOL COVERAGE FAILURE: Only $TOOLS_WITH_TESTS/26 tools have tests"
    echo "   Every tool needs individual test coverage"
    exit 1
fi

# 3. Check error handling coverage
ERROR_TESTS=$(grep -r "test_.*_error\|test_.*_invalid\|test_.*_missing" crates/codeprism-mcp-server/src/ --include="*.rs" | wc -l)
MIN_ERROR_TESTS=20

if [ "$ERROR_TESTS" -lt "$MIN_ERROR_TESTS" ]; then
    echo "❌ ERROR COVERAGE FAILURE: Only $ERROR_TESTS error tests found, need $MIN_ERROR_TESTS+"
    echo "   Add comprehensive error scenario testing"
    exit 1
fi

# 4. Check integration test coverage
INTEGRATION_TESTS=$(grep -r "test_.*_integration\|test_.*_workflow\|test_full_" crates/codeprism-mcp-server/src/ --include="*.rs" | wc -l)
MIN_INTEGRATION_TESTS=10

if [ "$INTEGRATION_TESTS" -lt "$MIN_INTEGRATION_TESTS" ]; then
    echo "❌ INTEGRATION COVERAGE FAILURE: Only $INTEGRATION_TESTS integration tests found, need $MIN_INTEGRATION_TESTS+"
    echo "   Add end-to-end workflow testing"
    exit 1
fi

# 5. Check code coverage percentage
if command -v cargo-tarpaulin >/dev/null 2>&1; then
    COVERAGE=$(cargo tarpaulin --skip-clean --out xml 2>/dev/null | grep -oP '(?<=line-rate=")[^"]*' | head -1 | awk '{print $1*100}')
    MIN_COVERAGE=90
    
    if (( $(echo "$COVERAGE < $MIN_COVERAGE" | bc -l) )); then
        echo "❌ CODE COVERAGE FAILURE: ${COVERAGE}% coverage, need ${MIN_COVERAGE}%+"
        echo "   Add tests to cover untested code paths"
        exit 1
    fi
    
    echo "✅ Code coverage: ${COVERAGE}%"
fi

echo "✅ All coverage requirements met:"
echo "   - Total tests: $CURRENT_TESTS (≥$MIN_REQUIRED_TESTS required)"
echo "   - Tool tests: $TOOLS_WITH_TESTS/26 tools covered"
echo "   - Error tests: $ERROR_TESTS (≥$MIN_ERROR_TESTS required)"  
echo "   - Integration tests: $INTEGRATION_TESTS (≥$MIN_INTEGRATION_TESTS required)"
```

### Coverage Reporting

**REQUIRED: Generate comprehensive coverage reports**

```bash
# Generate detailed coverage report
cargo tarpaulin \
    --out Html --out Xml \
    --exclude-files "target/*" \
    --exclude-files "tests/*" \
    --fail-under 90 \
    --timeout 300

# Validate coverage against requirements
python3 << 'EOF'
import xml.etree.ElementTree as ET

# Parse coverage XML
tree = ET.parse('tarpaulin-report.xml')
root = tree.getroot()

# Extract coverage metrics
line_coverage = float(root.attrib.get('line-rate', 0)) * 100
branch_coverage = float(root.attrib.get('branch-rate', 0)) * 100

# Validate against requirements
print(f"Line Coverage: {line_coverage:.1f}%")
print(f"Branch Coverage: {branch_coverage:.1f}%")

if line_coverage < 90:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
