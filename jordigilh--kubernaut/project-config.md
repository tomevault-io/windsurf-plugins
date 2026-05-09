---
trigger: always_on
description: Business code integration patterns and validation requirements for kubernaut
---

# Business Code Integration Standards

## Core Requirement - MANDATORY

**Rule**: ALL business logic MUST be integrated into main application workflows
**Prevention**: Design integration points during APDC Analysis and Plan phases

## Integration Definition

Business code is **properly integrated** when:
1. **Main Application Entry**: Code is instantiated and used in main application entry points
2. **Runtime Execution**: Code executes during normal application workflows (not just tests)
3. **Dependency Chain**: All dependencies are properly wired in main application
4. **Interface Implementation**: Interfaces are implemented by real business code (not just mocks)
5. **Configuration Loading**: Business code configuration is loaded in main application
6. **Error Integration**: Business code errors are handled in main application flow
7. **Monitoring Integration**: Business code metrics are exposed in main application

## Integration Validation Matrix

| Integration Aspect | Validation Method | Required Evidence |
|-------------------|------------------|-------------------|
| **Main Entry Point** | Code tracing from `cmd/*/main.go` | Direct instantiation or factory usage |
| **Runtime Path** | Execution flow analysis | Normal workflow execution path |
| **Dependency Wiring** | Dependency injection validation | All deps available at runtime |
| **Interface Usage** | Implementation verification | Real implementations in main app |
| **Configuration** | Config loading verification | Business config loaded in main |
| **Error Handling** | Error flow analysis | Business errors handled in main |
| **Monitoring** | Metrics integration check | Business metrics exposed |

## 🚫 **IRONCLAD INTEGRATION VERIFICATION PROCESS**

### **MANDATORY INTEGRATION VALIDATION - BLOCKING REQUIREMENTS**

**🚫 BLOCKING REQUIREMENT - BEFORE ANY BUSINESS CODE CREATION**:

<function_calls>
<invoke name="Grep">
<parameter name="pattern">[BusinessComponentType]</parameter>
<parameter name="path">cmd/</parameter>
<parameter name="output_mode">content</parameter>
<parameter name="-n">true</parameter>
</invoke>
</function_calls>

<function_calls>
<invoke name="Task">
<parameter name="subagent_type">general-purpose</parameter>
<parameter name="description">Main application integration analysis</parameter>
<parameter name="prompt">Analyze main applications in cmd/ directory to understand integration patterns. Find how similar business components are instantiated, configured, and wired into the application flow.</parameter>
</invoke>
</function_calls>

```
✅ INTEGRATION VERIFICATION CHECKPOINT:
- [ ] Main application search executed ✅/❌
- [ ] Integration patterns identified and documented ✅/❌
- [ ] Similar component usage patterns discovered ✅/❌
- [ ] Integration plan developed for cmd/ applications ✅/❌
- [ ] Runtime execution path verified ✅/❌

❌ STOP: Cannot create business code until ALL checkboxes are ✅
```

**🚫 MANDATORY INTEGRATION EVIDENCE - POST-IMPLEMENTATION**:

<function_calls>
<invoke name="Grep">
<parameter name="pattern">[ImplementedBusinessComponent]</parameter>
<parameter name="path">cmd/</parameter>
<parameter name="output_mode">files_with_matches</parameter>
</invoke>
</function_calls>

<function_calls>
<invoke name="Grep">
<parameter name="pattern">New[BusinessComponent]|Create[BusinessComponent]</parameter>
<parameter name="path">cmd/</parameter>
<parameter name="output_mode">content</parameter>
</invoke>
</function_calls>

```
✅ INTEGRATION EVIDENCE CHECKPOINT:
- [ ] Component appears in main applications ✅/❌
- [ ] Component instantiation confirmed ✅/❌
- [ ] Runtime execution path verified ✅/❌
- [ ] No orphaned business code ✅/❌

❌ VIOLATION: If ANY checkbox is ❌ → "🚨 ORPHANED BUSINESS CODE VIOLATION: All business code MUST be integrated in main applications - DEVELOPMENT STOPPED"
```

### Step 1: Call Graph Analysis (Tool-Enforced)
```bash
# MANDATORY: Execute these validation commands
grep -r "[BusinessComponentType]" cmd/ --include="*.go"
find pkg/ -name "*.go" -not -name "*_test.go" -exec grep -l "[BusinessComponentType]" {} \;
```

### Step 2: Runtime Path Verification
```go
// Add integration validation to main application startup
func validateBusinessIntegration() error {
    // Verify each business component is properly integrated
    if businessComponent == nil {
        return fmt.Errorf("business component not integrated")
    }

    // Test business component can be invoked
    if err := businessComponent.HealthCheck(); err != nil {
        return fmt.Errorf("business component not functional: %w", err)
    }

    return nil
}
```

### Step 3: Dependency Chain Validation
```go
// Verify all business code dependencies are available
func validateDependencyChain() error {
    requiredDeps := []string{"database", "vectorDB", "llmClient", "k8sClient"}

    for _, dep := range requiredDeps {
        if !isDependencyAvailable(dep) {
            return fmt.Errorf("required dependency %s not available for business code", dep)
        }
    }

    return nil
}
```

## Integration Patterns

### Pattern 1: Direct Main Application Integration
```go
// cmd/kubernaut/main.go
func main() {
    // Create business components
    analyticsEngine := insights.NewAnalyticsEngine(deps...)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
