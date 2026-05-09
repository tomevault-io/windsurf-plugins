---
trigger: always_on
description: Interface and method validation before code generation
---

# Interface and Method Validation Mandate - ENHANCED

## 🔍 **MANDATORY: Interface and Method Validation Before Code Generation**

### Critical Validation Process - ENHANCED
**MANDATORY**: Before generating ANY test code that calls methods or uses types:

1. **Interface Verification**: Use `codebase_search` to verify actual interface definitions
2. **Method Existence Check**: Confirm all called methods exist with EXACT signatures
3. **Type Validation**: Verify all referenced types and struct fields exist
4. **Import Validation**: Ensure all imported packages and types are available
5. **🆕 COMPILATION VERIFICATION**: MANDATORY compilation check after interface usage
6. **🆕 TYPE COMPATIBILITY CHECK**: Verify parameter and return type compatibility

### 🆕 **MANDATORY CODE GENERATION HALT PROTOCOL - ENHANCED**
**BEFORE generating ANY line of code:**

1. **MANDATORY SEARCH**: Run `codebase_search "existing [ComponentType] real implementations"` first, then `codebase_search "existing [ComponentType] mock implementations"`
2. **MANDATORY VERIFICATION**: If real business logic exists, PREFER it over mocks; if using mocks, use existing ones
3. **🆕 BUILD ERROR PREVENTION**: Check for common build error patterns
4. **🆕 IMPORT CONSISTENCY**: Verify all imports exist and are properly used
5. **VIOLATION RESPONSE**: If attempting to create duplicate mocks, IMMEDIATELY STOP and use existing

**ENFORCEMENT TRIGGER WORDS**:
- Creating any type with "Mock" in name → TRIGGER validation
- Using `NewMock*` → TRIGGER existing pattern search
- Implementing interfaces → TRIGGER interface validation

**VIOLATION AUTO-DETECTION - ENHANCED**:
```bash
# If you find yourself typing any of these, STOP:
type Mock* struct          # ❌ VIOLATION: Check existing mocks first
func NewMock*             # ❌ VIOLATION: Use existing patterns
*Mock struct {            # ❌ VIOLATION: Reuse existing mocks
logrus.New()              # ❌ VIOLATION: Use existing mocks.NewMockLogger()
mockLogger                # ❌ VIOLATION: Check variable declaration
import.*logrus.*\n.*not   # ❌ VIOLATION: Unused import detected
```

**🆕 COMMON BUILD ERROR PATTERNS TO PREVENT**:
```bash
# These patterns MUST trigger immediate validation:
mockLogger.*without.*var  # ❌ Undefined variable usage
import.*unused           # ❌ Unused import statements
mocks\..*without.*import  # ❌ Mock usage without import
NewMock.*duplicate       # ❌ Duplicate mock creation
```

---

## 🚨 **MANDATORY VALIDATION SEQUENCE - ENHANCED**

### **Step 1: Interface Discovery and Verification**
```bash
# MANDATORY: Search for existing interfaces before creating/using
codebase_search "existing [InterfaceName] interface definitions"
codebase_search "existing [InterfaceName] implementations"

# Verify interface exists and get exact signature
grep -r "type.*[InterfaceName].*interface" pkg/ --include="*.go"
```

**Example Validation**:
```bash
# Before using WorkflowEngine interface
codebase_search "existing WorkflowEngine interface definitions"
# Result should show: pkg/workflow/engine/interfaces.go

# Verify method signatures
grep -A 10 "type WorkflowEngine interface" pkg/workflow/engine/interfaces.go
```

### **Step 2: Method Signature Validation**
```bash
# MANDATORY: Verify exact method signatures before calling
grep -A 20 "type.*[InterfaceName].*interface" [interface_file.go]

# Check method parameters and return types
grep "[MethodName].*(" [interface_file.go]
```

**Example Method Validation**:
```go
// ✅ CORRECT: Verify method signature first
// From pkg/workflow/engine/interfaces.go:
// CreateWorkflow(ctx context.Context, alert AlertData) (*Workflow, error)

// Then use in test:
workflow, err := workflowEngine.CreateWorkflow(ctx, alertData)
```

### **Step 3: Mock Existence and Reuse Check**
```bash
# MANDATORY: Check for existing mocks before creating new ones
find pkg/testutil/mocks/ -name "*[ComponentName]*" -type f
grep -r "Mock[ComponentName]" pkg/testutil/ --include="*.go"

# If mocks exist, REUSE them
# If no mocks exist, check if real component should be used instead
```

**Mock Reuse Decision Matrix**:
| Component Type | Action |
|---------------|--------|
| **External Services** (AI, K8s, DB) | Use existing mocks from `pkg/testutil/mocks/` |
| **Business Logic** (Engine, Analytics) | Use REAL components |
| **Configuration** | Use real config with test values |
| **Utilities** | Use real utilities |

### **🆕 Step 4: Compilation Verification**
```bash
# MANDATORY: Test compilation after interface usage
go build ./test/[test_package]/ 2>&1 | tee build_check.log

# Check for common errors:
grep "undefined:" build_check.log    # Undefined symbols
grep "cannot use" build_check.log    # Type mismatches
grep "not enough arguments" build_check.log  # Parameter mismatches
```

### **🆕 Step 5: Import Consistency Check**
```bash
# MANDATORY: Verify all imports are used and correct
go mod tidy
goimports -w [test_file.go]

# Check for unused imports
go build [test_file.go] 2>&1 | grep "imported and not used"
```

---

## 🔧 **AUTOMATED VALIDATION TOOLS**

### **Interface Validation Script**
```bash
#!/bin/bash
# scripts/validate-interface-usage.sh

INTERFACE_NAME="$1"
TEST_FILE="$2"

echo "🔍 VALIDATING INTERFACE USAGE: $INTERFACE_NAME in $TEST_FILE"

# Step 1: Find interface definition

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
