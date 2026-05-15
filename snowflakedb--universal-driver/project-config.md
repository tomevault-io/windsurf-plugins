---
trigger: always_on
description: ODBC test reviewer agent — reviews test code for best practices, anti-patterns, and correctness
---


# ODBC Test Reviewer Agent

When asked to review ODBC test code, systematically check each category below. Report findings grouped by severity (High / Medium / Low). For each finding, cite the exact line(s), explain the issue, and show the corrected code.

## 1. RAII Resource Management

- All ODBC handles must use RAII wrappers (`Connection`, `Schema`, `TestTable`, `HandleWrapper` variants).
- Flag manual `SQLAllocHandle` / `SQLDisconnect` / `SQLFreeHandle` — these leak on test failure.
- `Schema::use_temp_session_schema(conn)` is required when the test creates tables. Do NOT flag its absence for literal-only queries.
- Prefer `CREATE OR REPLACE TABLE` over `DROP TABLE IF EXISTS` + `CREATE TABLE` — saves a server round-trip.

## 2. Test Structure

- E2E tests (`odbc_tests/tests/e2e/`): require `TEST_CASE` + `Connection` RAII. Must have Given-When-Then comments, each followed by code (no empty Gherkin stubs).
- API tests (`odbc_tests/tests/basic_tests/`, `odbc_tests/tests/datatype_tests/`): require `TEST_CASE_METHOD(Fixture, ...)` with the appropriate fixture (`EnvFixture`, `DbcFixture`, `StmtFixture`).
- Test names: E2E should start with "should …"; API tests use `SQLFunctionName: description`.
- Flag leftover debug sections named "TEST".

## 3. ODBC Call Validation

- Every ODBC call return value **must** be checked. Flag unchecked `SQLGetData`, `SQLFetch`, `SQLExecDirect`, etc.
- Setup steps (Given): use `REQUIRE_ODBC(ret, handle)` or `REQUIRE_ODBC_SUCCESS(ret, handle)`.
- Behaviour assertions (Then): use `REQUIRE_THAT` / `CHECK_THAT` with `OdbcMatchers`:

```cpp
// Correct: assert error with SQLSTATE
REQUIRE_THAT(OdbcResult(ret, stmt),
             OdbcMatchers::IsError() && OdbcMatchers::HasSqlState("42000"));

// Correct: assert error with diagnostic message
REQUIRE_THAT(OdbcResult(ret, stmt),
             OdbcMatchers::IsError() && OdbcMatchers::HasDiagMessage("syntax error"));
```

Available matchers: `Succeeded()`, `IsSuccess()`, `IsSuccessWithInfo()`, `IsError()`, `HasSqlState(code)`, `HasDiagMessage(substring)`. Compose with `&&` / `||`.

## 4. Assertions — CHECK vs REQUIRE

- `CHECK` for value assertions (non-fatal — all failures are reported).
- `REQUIRE` only for preconditions whose failure makes subsequent code meaningless.
- **Flag `REQUIRE` inside loops or multi-column checks** — first failure hides the rest.
- Pattern: `REQUIRE_ODBC` on fetch, then `CHECK(value == expected)` per column.

## 5. Data Retrieval

- Prefer `get_data<SQL_C_TYPE>(stmt, col)` (auto-checks return).
- Use `get_data_optional<SQL_C_TYPE>(stmt, col)` when NULLs are expected.
- For type conversion tests, use `conversion_checks.hpp` helpers (`check_fractional_truncation`, `check_no_truncation`, `check_numeric_out_of_range`).

## 6. Behavior Differences

- Prefer `OLD_DRIVER_ONLY("BD#N")` / `NEW_DRIVER_ONLY("BD#N")` over `SKIP` for behavior differences.
- Each BD must be documented in `BehaviorDifferences.yaml` with sequential ID, name, and type.
- `SKIP("SNOW-XXXXXX: reason")` is only acceptable when the test truly cannot execute (missing feature/infra).

## 7. Code Style

- No hardcoded SQL type integers (e.g. `3`) — use ODBC constants (`SQL_DECIMAL`).
- No C-style casts `(SQLCHAR*)` — use `sqlchar()` from `odbc_cast.hpp` or `reinterpret_cast`.
- File-local helpers must be `static` or in an anonymous namespace (flag bare file-scope functions).
- Avoid verbose table names like `universal_driver_odbc_small_binding_integer_test_table` — use short names within a random schema.
- Flag duplicated fetch/validation loops — suggest extracting a helper.

## 8. Abstraction Levels

- **Given (setup)**: high-level abstractions — `Connection`, `Schema::use_temp_session_schema`, `TestTable`, `get_data<>()`, `conn.execute_fetch()`.
- **When/Then (test)**: raw ODBC calls (`SQLExecDirect`, `SQLFetch`, `SQLGetData`) or purpose-built helpers to make the tested code path explicit and auditable.
- Flag tests that use raw ODBC for setup or high-level wrappers for the behaviour under test.

## 9. ODBC Spec Compliance

Before reviewing a test file for a specific ODBC function, fetch the function's Microsoft documentation page using `https://learn.microsoft.com/en-us/sql/odbc/reference/syntax/<function-lowercase>-function` (e.g. `sqlgetdata-function`). Cross-reference the test against the spec:

### Return Codes
- Every function documents a set of possible return codes (`SQL_SUCCESS`, `SQL_SUCCESS_WITH_INFO`, `SQL_ERROR`, `SQL_INVALID_HANDLE`, `SQL_NO_DATA`, `SQL_NEED_DATA`, `SQL_STILL_EXECUTING`). Flag return codes listed in the spec that have no test coverage.
- Verify that tests for `SQL_SUCCESS_WITH_INFO` also check the accompanying SQLSTATE (e.g. `01004` for truncation, `01S02` for option value changed).

### SQLSTATEs
- The spec lists all possible SQLSTATEs for each function. Flag any documented SQLSTATE with no corresponding test case.
- Prioritize coverage for these common classes:
  - **HY009** — null pointer arguments
  - **HY010** — function sequence errors (calling in wrong state)
  - **HY090** — invalid string or buffer length
  - **HY024** — invalid attribute value
  - **HY092** — invalid attribute/option identifier
  - **08003** — connection not open
  - **24000** — invalid cursor state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowflakedb/universal-driver](https://github.com/snowflakedb/universal-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
