---
trigger: always_on
description: **Note:** This library implements the OData v4.01 specification.
---

# AI Agent Instructions for go-odata

## Library Description

**Note:** This library implements the OData v4.01 specification.

`go-odata` is a Go library for building services that expose OData APIs with automatic handling of OData protocol logic. It allows you to define Go structs representing entities, making it easy to build OData-compliant APIs.

### Architecture

The library is structured with:
- **Core service** (`odata.go`, `server.go`): Main OData service and HTTP server
- **Internal handlers** (`internal/handlers/`): Request handlers for entities, metadata, and service documents
- **Metadata processing** (`internal/metadata/`): Entity metadata extraction and analysis
- **Query processing** (`internal/query/`): OData query option parsing and execution
- **Response formatting** (`internal/response/`): OData-compliant response generation
- **Development server** (`cmd/devserver/`): Example implementation with sample data

### Testing

The project includes comprehensive tests:
- Unit tests for handlers, metadata, query processing, and responses (located in `internal/*/`)
- Integration tests for the main OData service (located in `test/`)
- OData v4 compliance tests (located in `compliance/v4/`)
- All tests use GORM with SQLite in-memory database

#### Test Organization

- **Integration tests**: All integration tests for the main OData service are located in the `test/` directory
  - These tests use the `odata_test` package and import the `odata` package
  - They test the public API of the service from an external perspective
- **Unit tests**: Internal package tests remain in their respective `internal/` subdirectories
  - These tests are in the same package as the code they test
- **White-box tests**: The root-level `odata_test.go` contains white-box tests that need access to unexported fields
- **Compliance tests**: OData v4 specification compliance tests in `compliance/v4/`
  - Shell scripts that test against a running development server
  - Validate strict adherence to the OData v4 specification
  - Must be kept in sync with specification requirements

When adding new tests:
- Place integration tests in the `test/` directory
- Use package `odata_test` and import `odata "github.com/nlstn/go-odata"`
- Place unit tests for internal packages in the same directory as the code

#### OData v4 Compliance Testing

**CRITICAL: Compliance tests MUST strictly adhere to the OData v4 specification.**

The `compliance-suite/` directory contains a Go-based test suite that validates the library's compliance with the official OData v4 specification. Tests are organized by OData version:

- **`tests/v4_0/`** - OData 4.0 specification compliance tests
- **`tests/v4_01/`** - OData 4.01-specific compliance tests
- **`framework/`** - Test framework with HTTP client and assertions
- **`main.go`** - Test runner with server management and reporting

##### Compliance Test Requirements

1. **Strict Specification Adherence**: Tests must validate EXACT compliance with the OData v4 spec
   - If the spec requires HTTP status 400, the test must fail if 500 is returned
   - If the spec requires specific headers, those exact headers must be present
   - Error response formats must match the specification exactly
   - No lenient behavior or "close enough" validations

2. **Test Structure**: Each compliance test:
   - Tests one specific section of the OData v4 specification
   - Is named according to the spec section (e.g., `query_filter.go`)
   - Is placed in `tests/v4_0/` for OData 4.0 features, or `tests/v4_01/` for 4.01-specific features
   - For `tests/v4_01/`: MUST only test behavior that is new in 4.01 or explicitly different from 4.0
   - For `tests/v4_01/`: MUST verify version-gated behavior by asserting the 4.01 behavior works when 4.01 is negotiated and does NOT apply when 4.0 is negotiated
   - Includes spec reference URLs in the TestSuite definition
   - Can run independently or as part of the full suite
   - Returns appropriate exit codes for CI/CD integration
   - Cleans up any test data it creates (non-destructive testing)

3. **When Modifying Compliance Tests**:
   - **NEVER make tests more lenient** to accommodate current implementation
   - If a test fails, the implementation must be fixed, not the test
   - Tests should reveal gaps between implementation and specification
   - Document any intentional deviations from the spec with clear justification
   - Update tests only when the OData specification itself changes

4. **Running Compliance Tests**:
   
   **IMPORTANT: Always run compliance tests using the Go-based test suite.**
   
   The test suite at `compliance-suite/` ensures:
   - Proper test environment setup and cleanup
   - Consistent execution across all test versions
   - Comprehensive reporting and error tracking
   - Automatic server management
   
   ```bash
   # Run all compliance tests (4.0 + 4.01) - PREFERRED METHOD
   cd compliance-suite
   go run .
   
   # Run only OData 4.0 tests
   go run . -version 4.0
   
   # Run only OData 4.01 tests
   go run . -version 4.01
   
   # Run specific tests by pattern
   go run . -pattern filter
   ```

5. **Test Coverage**: The Go-based test suite covers:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NLstn/go-odata](https://github.com/NLstn/go-odata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
