---
trigger: always_on
description: This rule applies to all Go test files in the project.
---

# Testing Standards for gzcli

This rule applies to all Go test files in the project.

## Testing Philosophy

### Test Pyramid

Follow the testing pyramid approach:
- **Many** unit tests (fast, isolated)
- **Some** integration tests (slower, test interactions)
- **Few** E2E tests (slowest, test complete workflows)

### Coverage Goals

- **Critical packages** (watcher, challenge, API): >85%
- **Command handlers:** >70%
- **Utilities:** >80%
- **Overall project:** >80%

## Test Naming

### Test Function Names

Use descriptive names that explain what is being tested:

**Format:** `Test<Component>_<Method>_<Scenario>_<ExpectedResult>`

```go
// Good examples
func TestWatcher_HandleFileChange_RedeploysChallenge(t *testing.T)
func TestGZAPI_Login_WithInvalidCredentials_ReturnsError(t *testing.T)
func TestConfig_Load_WithMissingFile_ReturnsError(t *testing.T)

// Bad examples
func TestWatcher(t *testing.T)  // Too vague
func Test1(t *testing.T)        // No description
func TestLoginError(t *testing.T)  // Missing component context
```

### Subtest Names

Use clear subtest names:

```go
t.Run("ValidInput", func(t *testing.T) { ... })
t.Run("EmptyString", func(t *testing.T) { ... })
t.Run("InvalidFormat", func(t *testing.T) { ... })
```

## Table-Driven Tests

### When to Use

Use table-driven tests for:
- Multiple input scenarios
- Testing edge cases
- Parameterized tests
- Functions with clear inputs and outputs

### Structure

```go
func TestParseChallenge(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    *Challenge
        wantErr bool
    }{
        {
            name: "valid challenge",
            input: `name: Test
category: Web`,
            want: &Challenge{
                Name:     "Test",
                Category: "Web",
            },
            wantErr: false,
        },
        {
            name:    "invalid YAML",
            input:   "invalid: [",
            want:    nil,
            wantErr: true,
        },
        {
            name:    "empty input",
            input:   "",
            want:    nil,
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseChallenge(tt.input)

            if (err != nil) != tt.wantErr {
                t.Errorf("ParseChallenge() error = %v, wantErr %v", err, tt.wantErr)
                return
            }

            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("ParseChallenge() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

## Test Organization

### File Structure

```
package/
├── handler.go
├── handler_test.go                  # Unit tests
├── handler_integration_test.go      # Integration tests
├── testdata/                        # Test fixtures
│   ├── valid_input.json
│   └── invalid_input.json
└── testutil/                        # Test helpers
    └── mocks.go
```

### Test vs Production Code

- Tests should be next to the code they test
- Use `_test.go` suffix
- Tests can be in the same package or `_test` package

```go
// Same package - can test unexported functions
package gzapi

func TestInternalFunction(t *testing.T) { ... }

// External package - only test exported API
package gzapi_test

func TestGZAPI_Login(t *testing.T) { ... }
```

## Test Fixtures

### Using testdata/

Store test fixtures in `testdata/` directory:

```go
func TestLoadConfig(t *testing.T) {
    data, err := os.ReadFile("testdata/valid_config.yaml")
    if err != nil {
        t.Fatalf("Failed to load fixture: %v", err)
    }

    cfg, err := ParseConfig(data)
    if err != nil {
        t.Errorf("ParseConfig() failed: %v", err)
    }

    if cfg.URL != "https://example.com" {
        t.Errorf("URL = %v, want https://example.com", cfg.URL)
    }
}
```

### Temporary Directories

Use `t.TempDir()` for temporary directories:

```go
func TestFileOperation(t *testing.T) {
    tmpDir := t.TempDir()  // Automatically cleaned up

    filePath := filepath.Join(tmpDir, "test.txt")
    err := os.WriteFile(filePath, []byte("test"), 0644)
    if err != nil {
        t.Fatal(err)
    }

    // Test with file
}
```

## Mocking and Stubs

### Interface Mocking

Define interfaces for dependencies:

```go
// Interface for HTTP client
type HTTPClient interface {
    Do(req *http.Request) (*http.Response, error)
}

// Mock implementation
type MockHTTPClient struct {
    DoFunc func(req *http.Request) (*http.Response, error)
}

func (m *MockHTTPClient) Do(req *http.Request) (*http.Response, error) {
    if m.DoFunc != nil {
        return m.DoFunc(req)
    }
    return nil, errors.New("DoFunc not set")
}

// Use in tests
func TestAPICall(t *testing.T) {
    mockClient := &MockHTTPClient{
        DoFunc: func(req *http.Request) (*http.Response, error) {
            return &http.Response{
                StatusCode: 200,
                Body:       io.NopCloser(strings.NewReader(`{"status":"ok"}`)),
            }, nil
        },
    }

    api := &API{client: mockClient}
    // Test with mock
}
```

### Using testutil Package

See [testutil README](mdc:internal/gzcli/testutil/README.md) for available test utilities.

## Assertions

### Error Checking

Always check both presence and absence of errors:

```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dimasma0305) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
