---
trigger: always_on
description: ODBC specific test generation
---


# ODBC Test Generation Rules

## Validation Workflow (MANDATORY)

1. **Validate against OLD driver first** using `odbc_tests/run_reference.sh -R <test_suite_name>`
2. **Run format validator** with `tests/tests_format_validator/run_validator.sh`
3. **Run precommit** to ensure linter compliance

## ODBC Reference Documentation

When writing or reviewing ODBC tests, fetch the relevant Microsoft ODBC API reference pages for context. Use these URLs as needed:
- Function reference index: https://learn.microsoft.com/en-us/sql/odbc/reference/syntax/odbc-api-reference
- Individual function (replace `sqlfunctionname` with lowercase function name, e.g. `sqlgetdata`, `sqlexecdirect`, `sqlfetch`): `https://learn.microsoft.com/en-us/sql/odbc/reference/syntax/sqlfunctionname-function`
- Data types: https://learn.microsoft.com/en-us/sql/odbc/reference/appendixes/sql-data-types
- Return codes: https://learn.microsoft.com/en-us/sql/odbc/reference/develop-app/return-codes-odbc
- Diagnostics / SQLGetDiagRec: https://learn.microsoft.com/en-us/sql/odbc/reference/syntax/sqlgetdiagrec-function
- SQLSTATE reference: https://learn.microsoft.com/en-us/sql/odbc/reference/appendixes/appendix-a-odbc-error-codes

Before writing or modifying a test for a specific ODBC function, use the WebFetch tool to retrieve that function's documentation page. For example, when working on a `SQLGetData` test, fetch `https://learn.microsoft.com/en-us/sql/odbc/reference/syntax/sqlgetdata-function`. Use the fetched content to verify correct parameter types, buffer sizes, expected return codes, SQLSTATE values, and documented edge cases.

## C++ Test Structure

### Framework & Includes
- Use **Catch2** testing framework
- Standard includes:
  ```cpp
  #include <catch2/catch_test_macros.hpp>
  #include "Connection.hpp"
  #include "HandleWrapper.hpp"
  #include "macros.hpp"
  #include "test_setup.hpp"
  ```

### Test Case Naming & Structure
- Use `TEST_CASE` + `Connection` RAII. Names start with `"should ..."`.
- Use `TEST_CASE_METHOD(Fixture, ...)` with the appropriate fixture (`EnvFixture`, `DbcFixture`, `StmtFixture`).
- Tags should match category: `[datatype][string]`, `[put_get]`, `[auth]`, `[query]`

### Given-When-Then Comments
Always structure tests with Gherkin comments. Each comment must be followed by relevant code (no empty stubs).

**E2E example:**
```cpp
TEST_CASE("should select data from file uploaded to stage", "[put_get]") {
  // Given File is uploaded to stage
  Connection conn;
  // setup code using high-level abstractions...

  // When File data is queried using Select command
  SQLRETURN ret = SQLExecDirect(stmt.getHandle(), sqlchar("SELECT ..."), SQL_NTS);

  // Then File data should be correctly returned
  CHECK(result == expected);
}
```

**API test example:**
```cpp
TEST_CASE_METHOD(StmtFixture, "SQLGetData: should return correct integer value", "[sqlgetdata]") {
  // Given A query returning an integer is executed
  SQLRETURN ret = SQLExecDirect(stmt, sqlchar("SELECT 42"), SQL_NTS);
  REQUIRE_ODBC(ret, stmt);
  ret = SQLFetch(stmt);
  REQUIRE_ODBC(ret, stmt);

  // When Data is retrieved via SQLGetData
  SQLINTEGER value = 0;
  SQLLEN indicator = 0;
  ret = SQLGetData(stmt, 1, SQL_C_LONG, &value, sizeof(value), &indicator);

  // Then The call should succeed and return the correct value
  REQUIRE_THAT(OdbcResult(ret, stmt), OdbcMatchers::Succeeded());
  CHECK(value == 42);
}
```

## Core Patterns

### Connection Management
```cpp
Connection conn;  // Uses default connection string
// OR
Connection conn(custom_connection_string);

auto stmt = conn.createStatement();
auto stmt = conn.execute("SQL QUERY");
auto stmt = conn.execute_fetch("SQL QUERY");  // execute + fetch first row
```

### ODBC Return Value Checking
Every ODBC call return value **must** be checked. Use different macros depending on context:
- **Setup (Given)**: use `REQUIRE_ODBC(ret, handle)` or `REQUIRE_ODBC_SUCCESS(ret, handle)` — failure is fatal, subsequent code is meaningless.
- **Behaviour assertions (Then)**: use `REQUIRE_THAT` / `CHECK_THAT` with `OdbcMatchers` (see below).

```cpp
// Setup — fatal on failure
SQLRETURN ret = SQLExecDirect(stmt.getHandle(), sqlchar("SELECT 1"), SQL_NTS);
REQUIRE_ODBC(ret, stmt);

// Behaviour assertion — the return code itself is what we're testing
ret = SQLExecDirect(stmt.getHandle(), sqlchar("INVALID SQL"), SQL_NTS);
REQUIRE_THAT(OdbcResult(ret, stmt),
             OdbcMatchers::IsError() && OdbcMatchers::HasSqlState("42000"));
```

### Schema & Table Management

Call `Schema::use_temp_session_schema(conn)` (or the `SQLHDBC` overload) before creating any tables. In CI, all test processes share a single schema (`ODBC_TEST_SCHEMA` env var); in IDE/direct runs, each process gets its own random schema with automatic cleanup.

**Table creation strategy (ordered by preference):**

1. **`CREATE TEMPORARY TABLE`** (default) — session-scoped, auto-dropped on disconnect, session-isolated so names can be simple. Use for all normal data tests.
   ```cpp
   Schema::use_temp_session_schema(conn);
   conn.execute("CREATE TEMPORARY TABLE my_table (id INT, value VARCHAR(100))");
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowflakedb/universal-driver](https://github.com/snowflakedb/universal-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
