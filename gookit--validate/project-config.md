---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# gookit/validate - Go Data Validation Library

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

gookit/validate is a generic Go data validation and filtering library that supports validating Maps, Structs, and HTTP Request data with over 70 built-in validators and filters.

## Working Effectively

### Bootstrap and Setup
- Ensure Go 1.18+ is installed (supports Go 1.19-1.24 per CI)
- Clone repository: `git clone https://github.com/gookit/validate.git`
- Download dependencies: `go mod download` -- takes ~2 seconds
- Verify compilation: `go build ./...` -- takes ~20 seconds

### Build and Test
- Build all packages: `go build ./...` -- takes ~20 seconds, NEVER CANCEL, set timeout to 60+ seconds
- Run all tests: `go test ./...` -- takes ~5 seconds, NEVER CANCEL, set timeout to 30+ seconds  
- Run tests with coverage (CI command): `go test -coverprofile="profile.cov" ./..."` -- takes ~2 seconds
- Run benchmarks: `go test -bench=. -benchmem .` -- takes ~3 seconds
- Validate code: `go vet ./...` -- takes <1 second
- Format code: `go fmt ./...` -- takes <1 second

### Linting
- The project uses golangci-lint but has environmental issues in some setups
- If golangci-lint fails, use: `go vet ./...` as a reliable alternative
- Always run `go vet ./...` and `go fmt ./...` before committing
- CI uses golangci-lint v1.53 with specific linter configuration in `.golangci.yml`

## Validation

### Manual Testing Scenarios
Always test your changes by running validation scenarios:

1. **Basic Map Validation**:
```go
m := map[string]any{
    "name":  "john",
    "age":   25,
    "email": "john@example.com",
}
v := validate.Map(m)
v.StringRule("name", "required|min_len:2")
v.StringRule("age", "required|int|min:1|max:99") 
v.StringRule("email", "required|email")
// Should return: validation passes
```

2. **Struct Validation**:
```go
type User struct {
    Name  string `validate:"required|min_len:2" label:"User Name"`
    Email string `validate:"email" label:"Email Address"`
    Age   int    `validate:"required|int|min:1|max:99" label:"Age"`
}
user := &User{Name: "Alice", Email: "alice@example.com", Age: 30}
v := validate.Struct(user)
// Should return: validation passes
```

3. **Validation with Errors**:
```go
m := map[string]any{"name": "x", "age": 150, "email": "invalid"}
v := validate.Map(m)
v.StringRule("name", "required|min_len:2")
v.StringRule("age", "required|int|min:1|max:99")
v.StringRule("email", "required|email")
// Should return: validation fails with specific error messages
```

4. **Test HTTP Examples**:
- Run examples in `_examples/httpdemo/`: `cd _examples/httpdemo && go run main.go`
- Test HTTP validation endpoints if examples include servers

### Validation Steps for Changes
1. **Always run the full test suite** to ensure no regressions
2. **Test with both valid and invalid data** to verify error handling
3. **Check internationalization** by testing with different locales in `locales/`
4. **Verify struct tag parsing** works correctly with custom validators
5. **Test performance** with benchmarks if modifying core validation logic

## Common Tasks

### Repository Structure
```
.
├── README.md                    # Main documentation
├── go.mod                      # Go module definition
├── .golangci.yml              # Linter configuration
├── .github/workflows/go.yml   # CI/CD pipeline
├── _examples/                 # Usage examples
│   ├── httpdemo/             # HTTP validation examples
│   └── httpdemo2/            # Additional HTTP examples
├── locales/                   # Internationalization
│   ├── zhcn/                 # Chinese Simplified
│   ├── zhtw/                 # Chinese Traditional  
│   └── ruru/                 # Russian
├── docs/                      # Documentation
│   ├── validators.md         # Built-in validators reference
│   └── diff-with-go-validator.md
├── testdata/                  # Test data files
└── *.go                      # Core library files
```

### Key Source Files
- `validate.go` - Main validation entry points and public API
- `validation.go` - Core validation logic and Validation struct  
- `validators.go` - Built-in validator implementations (70+ validators)
- `data_source.go` - Data source abstractions (Map, Struct, HTTP Request)
- `filtering.go` - Data filtering and conversion logic
- `messages.go` - Error message handling and internationalization
- `rule.go` - Validation rule parsing and management
- `validating.go` - Core validation execution logic
- `util.go` - Utility functions for validation operations
- `value.go` - Value type conversions and handling

### Test Files Coverage
The project has 17 test files with 96.4% code coverage:
- `*_test.go` - Unit tests for corresponding source files
- `issues_test.go` - Regression tests for GitHub issues
- `benchmark_test.go` - Performance benchmarks

### Examples and Usage
- Check `_examples/` directory for practical usage patterns
- HTTP validation examples in `_examples/httpdemo/`
- README.md contains comprehensive usage examples
- Use examples as templates for new validation scenarios


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gookit/validate](https://github.com/gookit/validate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
