---
trigger: always_on
description: This document provides comprehensive guidelines for AI agents working with the GitLab client-go repository. It covers development practices, testing requirements, code formatting, API alignment, and code generation procedures.
---

# AI Agent Guidelines for GitLab client-go

This document provides comprehensive guidelines for AI agents working with the GitLab client-go repository. It covers development practices, testing requirements, code formatting, API alignment, and code generation procedures.

## Repository Overview

The GitLab client-go is a Go client library for the GitLab API, enabling Go programs to interact with GitLab in a simple and uniform way. The repository follows strict Go best practices and maintains close alignment with GitLab's official API documentation.

## Development Workflow

### Prerequisites

When asked to modify code, read CONTRIBUTING.md and README.md for examples and formatting instructions. Where the 
instructions in CONTRIBUTING.md and README.md conflict with information in AGENTS.md, prefer the instructions in 
CONTRIBUTING.md and README.md over the instructions in AGENTS.md

When asked to perform analysis on the codebase instead of changing code, skipping the read of CONTRIBUTING.md and README.md
is allowed and preferred, since understanding contributing guidelines is not required to perform analysis.

### Required Tools

- **Go** - Use the version specified in go.mod
- **gofumpt** - Code formatter
- **golangci-lint** - Linting tool
- **buf** - Protocol buffer tools for code generation
- **gomock** - Mock generation

### Running Tests

```bash
# Run all tests with race detection
mise exec -- make test

# Run the complete reviewable process (includes tests)
mise exec -- make reviewable
```

### Test Patterns

- Tests are parallelized using `t.Parallel()`
- Mock HTTP handlers are used for API testing
- Test data is stored in `testdata/` directory
- Each service method should have corresponding test coverage
  - **CRITICAL** - When fixing bugs or creating new features, ensure new test scenarios are added to cover the new logic.
- When writing a test, write Gherkin comments in-line with the test to make the tests easier to read. This means adding GIVEN/WHEN/THEN comments in tests.
- All tests use the `testing` package with `testify/assert`

Do not use `reflect.DeepEqual` in tests, use testify instead. 

```go
// This is an example of the incorrect test setup
want := &MyStruct{
		MyField: "Meow Kitty"
}
if !reflect.DeepEqual(want, otherStruct) {
    t.Errorf("MyStruct returned %+v, want %+v", otherStruct, want)
}

// Instead, use this
want := &MyStruct{
		MyField: "Meow Kitty"
}
assert.Equal(t, want, otherStruct)
```

### Test Structure Example

```go
func TestGetUser(t *testing.T) {
    t.Parallel()
    mux, client := setup(t)

    path := "/api/v4/users/1"
    mux.HandleFunc(path, func(w http.ResponseWriter, r *http.Request) {
        testMethod(t, r, http.MethodGet)
        mustWriteHTTPResponse(t, w, "testdata/get_user.json")
    })

    user, _, err := client.Users.GetUser(1, GetUsersOptions{})
    assert.NoError(t, err)
    // ... assertions
}
```

## Code Formatting and Linting

### Formatting

The project uses `gofumpt` for code formatting:

```bash
# Format all Go files
mise exec -- make fmt
```

**Formatting Rules:**
- Line width for comments: < 80 characters
- Line width for code: < 100 characters (where sensible)
- Use `gofumpt` for consistent formatting
- Follow Go best practices

### Linting

```bash
# Run all linters
mise exec -- make lint
```

**Linting Configuration:**
- Uses `golangci-lint` with custom configuration in `.golangci.yml`
- Enabled linters: asciicheck, dogsled, dupword, errorlint, goconst, misspell, 
  nakedret, nolintlint, revive, staticcheck, testifylint, unconvert, 
  usestdlibvars, whitespace
- Excludes generated files and examples directory

## Mock generation

This repository uses gomock to generate testing structs, which are in the `testing/` folder. These need to be kept up-to-date with function signatures to that the Service implementations match the interfaces that have generated mocks.

### Available Generation Commands

```bash
# Generate all code (protobuf, mocks, testing client)
mise exec -- make generate

# Clean generated files
mise exec -- make clean
```

### Generation Scripts

1. **`scripts/generate_testing_client.sh`** - Generates testing client with mocks
2. **`scripts/generate_mock_api.sh`** - Generates mock interfaces for all services
3. **`scripts/generate_service_interface_map.sh`** - Generates service interface mapping

### When to Regenerate

- After adding new service interfaces
- After modifying existing interfaces
- Before committing changes
- When mock generation fails

## Function Comment Formatting

### Required Comment Structure

Every public function, type, and method must have properly formatted comments:

```go
// FunctionName performs a specific action with the given parameters.
//
// GitLab API docs: https://docs.gitlab.com/api/endpoint/
func (s *ServiceName) FunctionName(param Type, opt *Options, options ...RequestOptionFunc) (*ReturnType, *Response, error) {
    // Implementation
}
```

### Comment Guidelines

1. **Function Comments:**
   - Start with function name (no "The" or "This function")
   - Use present tense ("performs", "returns", "creates")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrie30/ghorg](https://github.com/gabrie30/ghorg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
