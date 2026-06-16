---
trigger: always_on
description: Comprehensive testing standards and patterns for Compozy Go development - enforces mandatory t.Run patterns, testify usage, and mock standards
---

# Testing Standards for Compozy Go Development

<critical>
**MANDATORY REQUIREMENTS:**
- **ALWAYS** check dependent files APIs before write tests to avoid write wrong code
- **ALWAYS** verify against PRD and tech specs - NEVER make assumptions
- **NEVER** use workarounds, especially in tests - implement proper solutions
- **MUST** follow all established project standards:
    - Architecture patterns: `.cursor/rules/architecture.mdc`
    - Go coding standards: `.cursor/rules/go-coding-standards.mdc`
    - Testing requirements: `.cursor/rules/testing-standards.mdc`
    - API standards: `.cursor/rules/api-standards.mdc`
    - Security & quality: `.cursor/rules/quality-security.mdc`
- **MUST** run `make lint` and `make test` before completing ANY subtask
- **MUST** follow `.cursor/rules/task-review.mdc` workflow for parent tasks
**Enforcement:** Violating these standards results in immediate task rejection.
</critical>

## Core Testing Requirements

<requirements type="mandatory">
**MANDATORY testing patterns for all Go code:**
- Use `t.Run("Should describe expected behavior")` pattern for all tests
- Use `stretchr/testify` for assertions and mocks
- Follow table-driven test patterns when appropriate
- Achieve >85% coverage for business logic packages
</requirements>

## Testing Requirements

<requirements type="mandatory">
- **ALL tests MUST use `t.Run("Should...")` pattern** - no direct test implementation without t.Run wrapper
- Test function names: `func TestModuleName_MethodName(t *testing.T)`
- Each test case within t.Run with descriptive "Should..." names
- MUST use `stretchr/testify` for assertions and mocks
- **STANDARDIZE ON TESTIFY MOCK:** Replace existing custom mocks with `testify/mock` implementations
</requirements>

## Anti-Patterns to Avoid

<anti_patterns type="prohibited_patterns">
**NEVER USE TESTIFY SUITE PATTERNS:**
- ❌ **PROHIBITED:** `suite.Suite` embedding or any suite-based test structures
- ❌ **PROHIBITED:** Suite methods like `s.Equal()`, `s.NoError()`, `s.True()`, `s.False()`, `s.T()`
- ❌ **PROHIBITED:** `testsuite.WorkflowTestSuite` or similar suite embeddings
- ❌ **PROHIBITED:** Suite lifecycle methods like `SetupTest()`, `TearDownTest()`, `AfterTest()`

**USE DIRECT ASSERTIONS INSTEAD:**
- ✅ **REQUIRED:** `assert.Equal(t, expected, actual)`
- ✅ **REQUIRED:** `require.NoError(t, err)`
- ✅ **REQUIRED:** `assert.True(t, condition)`
- ✅ **REQUIRED:** Individual test functions with `*testing.T` parameter
</anti_patterns>

<anti_patterns type="bad_examples">
```go
// ❌ NEVER DO THIS - Suite pattern is prohibited
type MyTestSuite struct {
    suite.Suite
    // other fields
}

func (s *MyTestSuite) TestSomething() {
    s.Equal("expected", "actual")  // ❌ WRONG
    s.NoError(err)                 // ❌ WRONG
    s.T().Run("test", func(t *testing.T) { ... }) // ❌ WRONG
}

// ✅ DO THIS INSTEAD - Direct test functions
func TestSomething_Method(t *testing.T) {
    t.Run("Should behave correctly", func(t *testing.T) {
        assert.Equal(t, "expected", "actual")  // ✅ CORRECT
        require.NoError(t, err)                // ✅ CORRECT
    })
}
```
</anti_patterns>

## Table-Driven Tests

<guidelines type="table_tests">
- AVOID table-driven tests for 2-3 cases
- ONLY use when 5+ similar variations exist
- Each table test case must still use "Should..." naming
</guidelines>

## Test Organization

<organization_rules>
- Place `*_test.go` files alongside implementation files
- Each test MUST be independent and repeatable
- Mock external dependencies **only when necessary** using `testify/mock`
- Use project test helpers: `utils.SetupTest()`, `utils.SetupFixture()`
- Test both success and error paths
- Ensure test coverage for all exported functions
</organization_rules>

## Mock Standards

<when_to_mock>
**WHEN TO USE MOCKS:**
- External services (HTTP clients, databases, file systems)
- Dependencies that are slow, unreliable, or have side effects
- Complex interfaces that would make tests brittle or slow
- **NOT REQUIRED** for simple functions, pure logic, or internal utilities
</when_to_mock>

<pattern type="mock_implementation">
```go
// Define mock interface
type MockService struct {
    mock.Mock
}

func (m *MockService) DoSomething(ctx context.Context, param string) error {
    args := m.Called(ctx, param)
    return args.Error(0)
}

// Usage in tests
func TestComponent_Method(t *testing.T) {
    t.Run("Should use mocked service", func(t *testing.T) {
        mockService := new(MockService)
        mockService.On("DoSomething", mock.Anything, "test").Return(nil)

        component := NewComponent(mockService)
        err := component.Method("test")

        assert.NoError(t, err)
        mockService.AssertExpectations(t)
    })
}
```
</pattern>

<refactoring_priorities>
- Replace custom mocks with testify/mock implementations
- Migrate interface-based mocks to use `mock.Mock` embedding
- Standardize mock setup and assertion patterns across the codebase
</refactoring_priorities>

<example type="test_structure">
```go
func TestService_Method(t *testing.T) {
    t.Run("Should succeed with valid input", func(t *testing.T) {
        // arrange, act, assert
    })


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/gograph](https://github.com/compozy/gograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
