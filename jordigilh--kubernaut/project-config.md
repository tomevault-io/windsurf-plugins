---
trigger: always_on
description: Critical testing anti-patterns to avoid - detection and remediation
---


# Testing Anti-Patterns - Critical Reference

## 🚨 **CRITICAL ANTI-PATTERNS - IMMEDIATE REJECTION**

### **ANTI-PATTERN 1: NULL-TESTING (Most Critical)**

**Definition**: Testing for basic existence rather than business outcomes

**Violations**:
```go
// ❌ NULL-TESTING: Tests nothing meaningful
Expect(result).ToNot(BeNil())
Expect(result).ToNot(BeEmpty())
Expect(len(items)).To(BeNumerically(">", 0))
Expect(err).To(BeNil())  // Weak error testing
```

**Correct Approach**:
```go
// ✅ BUSINESS OUTCOME TESTING
Expect(workflow.Template.SafetyValidation).To(ContainElement("resource-limits"))
Expect(analysis.ConfidenceScore).To(BeNumerically(">=", 0.8))
Expect(recommendation.Actions).To(HaveLen(3))
```

---

### **ANTI-PATTERN 2: STATIC DATA TESTING**

**Definition**: Testing hardcoded values instead of business logic

**Violations**:
```go
// ❌ STATIC DATA TESTING
input := "test-alert-name"
Expect(result.Name).To(Equal("test-alert-name"))
```

**Correct Approach**:
```go
// ✅ BUSINESS LOGIC TESTING
input := testutil.GenerateAlertName()
Expect(result.Name).To(MatchRegexp("^alert-[0-9]+$"))
```

---

### **ANTI-PATTERN 3: LIBRARY TESTING**

**Definition**: Testing third-party libraries instead of business logic

**Violations**:
```go
// ❌ LIBRARY TESTING
logger := logrus.New()
Expect(logger).ToNot(BeNil())
```

**Correct Approach**:
```go
// ✅ BUSINESS LOGIC TESTING
logger := business.NewStructuredLogger(config)
logger.LogWorkflowEvent("workflow-123", "completed")
Expect(logger.GetLastEvent().WorkflowID).To(Equal("workflow-123"))
```

**Rule**: If it's not in `pkg/`, don't test it

---

### **ANTI-PATTERN 4: IMPLEMENTATION TESTING**

**Definition**: Testing how code works instead of what business outcome it produces

**Violations**:
```go
// ❌ IMPLEMENTATION TESTING
Expect(engine.parseWorkflowSteps(input)).To(HaveLen(3))
Expect(workflow.internalCache).To(HaveKey("step-1"))
```

**Correct Approach**:
```go
// ✅ BUSINESS OUTCOME TESTING
workflow, err := engine.CreateWorkflow(input)
Expect(err).ToNot(HaveOccurred())
Expect(workflow.Status).To(Equal(business.StatusReady))
Expect(workflow.CanExecute()).To(BeTrue())
```

---

### **ANTI-PATTERN 5: MOCK OVERUSE**

**Definition**: Mocking internal business logic instead of external dependencies

**Violations**:
```go
// ❌ MOCK OVERUSE: All business logic mocked = testing nothing
mockValidator := mocks.NewMockWorkflowValidator()
mockAnalyzer := mocks.NewMockPerformanceAnalyzer()
mockOptimizer := mocks.NewMockResourceOptimizer()
engine := NewEngine(mockValidator, mockAnalyzer, mockOptimizer)
```

**Correct Approach**:
```go
// ✅ REAL BUSINESS LOGIC: Mock only external dependencies
mockLLMClient := mocks.NewMockLLMClient()
mockK8sClient := fake.NewClientBuilder().Build()

// Use REAL business components
validator := business.NewWorkflowValidator()
analyzer := business.NewPerformanceAnalyzer()
engine := NewEngine(validator, analyzer, mockLLMClient, mockK8sClient)
```

---

### **ANTI-PATTERN 6: ENVIRONMENT VARIABLE CONFIGURATION (Python)**

**Definition**: Using environment variables for configuration instead of YAML files (ADR-030)

**Scope**: Python production code AND test code

**Violations**:
```python
# ❌ Configuration via env vars
url = os.getenv("DATA_STORAGE_URL")
timeout = int(os.getenv("TIMEOUT", "30"))
```

**Correct Approach**:
```python
# ✅ YAML configuration
import yaml
with open(config_file, 'r') as f:
    config = yaml.safe_load(f)
url = config['data_storage']['url']
```

**ONLY Exception**: `CONFIG_FILE` env var to specify YAML file location

**Detection**: `grep -r "os.getenv" holmesgpt-api/ --include="*.py" | grep -v "CONFIG_FILE"`

---

## 🔧 **AUTOMATED DETECTION**

### **Run Anti-Pattern Detection**

```bash
# Comprehensive anti-pattern check
make lint-test-patterns

# Or directly via script
./scripts/validation/check-test-anti-patterns.sh

# Verbose mode (shows file locations)
./scripts/validation/check-test-anti-patterns.sh --verbose
```

### **Manual Detection Commands**

```bash
# Detect NULL-TESTING violations
grep -r "ToNot(BeNil())\|ToNot(BeEmpty())" test/ --include="*_test.go"

# Detect STATIC DATA violations
grep -r "Equal(\".*\")" test/ --include="*_test.go" | grep -v "BR-"

# Detect LIBRARY TESTING violations
grep -r "logrus\.New()\|context\.WithValue" test/ --include="*_test.go"

# Check for missing BR references
find test/ -name "*_test.go" -exec grep -L "BR-" {} \;
```

---

## 🚨 **ENFORCEMENT**

### **Pre-Commit Hook**

```bash
#!/bin/bash
# .git/hooks/pre-commit

./scripts/validation/check-test-anti-patterns.sh

if [ $? -ne 0 ]; then
    echo "❌ Test anti-patterns detected. Commit blocked."
    exit 1
fi
```

**Setup**: Run `./scripts/setup-githooks.sh` to install

---

### **CI/CD Integration**

**Pipeline**: Tests automatically checked in pull requests
**Blocking**: PR cannot merge with anti-pattern violations
**Reporting**: Detailed violation report in PR comments

---

## 📊 **REMEDIATION WORKFLOW**

### **Step 1: Identify Violations**

```bash
# Run detection with file locations
./scripts/validation/check-test-anti-patterns.sh --verbose
```

### **Step 2: Fix Pattern**

```go
// BEFORE (NULL-TESTING)
It("should return a result", func() {
    result := engine.Process(input)
    Expect(result).ToNot(BeNil())
})


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jordigilh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
