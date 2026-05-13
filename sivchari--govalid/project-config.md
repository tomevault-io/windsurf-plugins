---
trigger: always_on
description: This document outlines the efficient development workflow established with our new automated registry system.
---

# Development Workflow for govalid

This document outlines the efficient development workflow established with our new automated registry system.

## 🚀 Implementation Pattern for New Markers

### 1. Feature Branch Setup
```bash
git checkout -b feature/{marker-name}-marker
```

### 2. Automated Validator Creation

#### A. One-Command Scaffold and Generation
```bash
# Create a new validator AND generate all registry files in one command:
make generate-validator MARKER=phoneNumber

# This:
# ✓ Generates scaffold: internal/validator/rules/phonenumber.go
# ✓ Updates internal/markers/markers_generated.go
# ✓ Creates internal/validator/registry/initializers/phonenumber.go
# ✓ Updates internal/validator/registry/initializers/all.go
# ✓ Updates internal/analyzers/govalid/registry_init.go
```

#### B. Implement Validator Logic
Edit the generated file with your validation logic:
```go
func (v *phonenumberValidator) Validate() string {
    // Return Go expression that evaluates to true when validation FAILS
    return fmt.Sprintf("!isValidPhoneNumber(t.%s)", v.FieldName())
}

func (v *phonenumberValidator) Imports() []string {
    return []string{"regexp"} // Add required imports
}
```

### 3. Testing Structure

#### A. Golden Tests (`internal/analyzers/govalid/testdata/src/{markername}/`)
- `{markername}.go` - Test input with marker comments
- `govalid.golden` - Expected generated output

#### B. Unit Tests (`test/unit/{markername}_test.go`)
```go
func Test{MarkerName}Validation(t *testing.T) {
    tests := []struct {
        name        string
        data        test.{MarkerName}
        expectError bool
    }{
        {"valid", test.{MarkerName}{Field: "valid_value"}, false},
        {"limit_minus_one", test.{MarkerName}{Field: "boundary-1"}, false},
        {"exactly_at_limit", test.{MarkerName}{Field: "boundary"}, false},
        {"limit_plus_one", test.{MarkerName}{Field: "boundary+1"}, true},
    }
    // Test both govalid and go-playground/validator
}
```

#### C. Benchmark Tests (`test/benchmark/benchmark_{markername}_test.go`)
```go
func BenchmarkGoValid{MarkerName}(b *testing.B) {
    instance := test.{MarkerName}{Field: "test_value"}
    b.ResetTimer()
    for b.Loop() {
        err := test.Validate{MarkerName}(&instance)
        if err != nil {
            b.Fatal("unexpected error:", err)
        }
    }
    b.StopTimer()
}

func BenchmarkGoPlayground{MarkerName}(b *testing.B) {
    validate := validator.New()
    instance := test.{MarkerName}{Field: "test_value"}
    b.ResetTimer()
    for b.Loop() {
        err := validate.Struct(&instance)
        if err != nil {
            b.Fatal("unexpected error:", err)
        }
    }
    b.StopTimer()
}
```

### 4. Test Execution Order
```bash
# 1. Build and install updated binary
go install ./cmd/govalid/

# 2. Generate test files
cd test && go generate

# 3. Run golden tests
cd .. && go test ./internal/analyzers/govalid/ -v

# 4. Run unit tests
cd test && go test ./unit/ -v

# 5. Run benchmarks
go test ./benchmark/ -bench=Benchmark.*{MarkerName} -benchmem

# 6. Update benchmark README
# Edit test/benchmark/README.md with results

# 7. Run lint checks and fix any issues
cd .. && make golangci-lint

# 8. Re-run benchmarks after any optimization changes
# If code changes were made to fix lint issues or optimize performance:
cd test && go test ./benchmark/ -bench=Benchmark.*{MarkerName} -benchmem

# 9. Update benchmark README again if performance changed
# Edit test/benchmark/README.md with updated results
```

### 5. Documentation Updates
- Update main README.md with marker explanation
- Update benchmark/README.md with performance results
- Document any behavior differences from go-playground/validator

## 🆕 New System Architecture

### Registry-Based Validator Discovery

The new system eliminates manual registration through an automated discovery and registry pattern:

```go
// internal/validator/registry/registry.go
type Registry interface {
    Markers() []string
    Validator(marker string) (ValidatorFactory, error)
    Init() error
}

// internal/validator/registry/initializers/
// Each validator has its own initializer automatically generated
type PhoneNumberInitializer struct{}

func (p PhoneNumberInitializer) Marker() string {
    return markers.GoValidMarkerPhoneNumber
}

func (p PhoneNumberInitializer) Init() registry.ValidatorFactory {
    return rules.ValidatePhoneNumber
}
```

### Automatic Code Generation Flow

```
make generate-validator MARKER=phonenumber
                    ↓
    ┌───────────────────────────────────┐
    │ Creates scaffold in rules/        │
    │ • internal/validator/rules/       │
    │   phonenumber.go                  │
    └───────────────────────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │ Automatically discovers all       │
    │ validators in rules/              │
    └───────────────────────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │ Generates:                        │
    │ • markers_generated.go            │
    │ • initializers/*.go               │
    │ • all.go                          │
    │ • registry_init.go                │
    └───────────────────────────────────┘

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sivchari/govalid](https://github.com/sivchari/govalid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
