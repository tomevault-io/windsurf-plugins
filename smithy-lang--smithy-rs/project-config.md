---
trigger: always_on
description: - **`codegen-core/`** - Shared codegen
---

# Smithy-rs AI Agent Guide

## Package Layout

- **`codegen-core/`** - Shared codegen
- **`codegen-server/`** - Server codegen
- **`codegen-client/`** - Client codegen
- **`rust-runtime/`** - Runtime libraries
- **`codegen-server-test/`** - Server integration tests

Protocol files: `codegen-{core,server}/.../protocols/`

## Protocol Tests

Protocol tests validate that generated code correctly implements Smithy protocols (like restJson1, awsJson1_1, etc.).

### Adding Protocol Tests

Protocol tests are defined in Smithy model files using `@httpRequestTests` and `@httpResponseTests` traits:

```
@http(uri: "/my-operation", method: "GET")
@httpRequestTests([
    {
        id: "MyOperationRequest",
        documentation: "Test description",
        protocol: "aws.protocols#restJson1",
        method: "GET",
        uri: "/my-operation",
        queryParams: ["param1=value1", "param2=value2"],
        params: {
            queryMap: {
                "param1": "value1",
                "param2": "value2"
            }
        },
        appliesTo: "client",
    }
])
operation MyOperation {
   input: MyOperationInput,
}
```

### Key Protocol Test Locations

- **`codegen-core/common-test-models/rest-json-extras.smithy`** - restJson1 protocol tests
- **`codegen-core/common-test-models/constraints.smithy`** - Constraint validation tests with restJson1
- **`codegen-client-test/model/main.smithy`** - awsJson1_1 protocol tests

### httpQueryParams Bug Investigation

When investigating the `@httpQueryParams` bug (where query parameters weren't appearing in requests), the issue was in `RequestBindingGenerator.kt` line 173. The bug occurred when:

1. An operation had ONLY `@httpQueryParams` (no regular `@httpQuery` parameters)
2. The condition `if (dynamicParams.isEmpty() && literalParams.isEmpty() && mapParams.isEmpty())` would skip generating the `uri_query` function

The fix was to ensure `mapParams.isEmpty()` was included in the condition check. The current implementation correctly generates query parameters for `@httpQueryParams` even when no other query parameters exist.

**Testing httpQueryParams**: Create operations with only `@httpQueryParams` to ensure they generate proper query strings in requests.

## rustTemplate Formatting

**CRITICAL**: Because `#` is the formatting character in `rustTemplate`, Rust attributes must be escaped:

❌ Wrong: `#[derive(Debug)]`
✅ Correct: `##[derive(Debug)]`

This applies to ALL Rust attributes: `##[non_exhaustive]`, `##[derive(...)]`, `##[cfg(...)]`, etc.

## preludeScope: Rust Prelude Types

**Always use `preludeScope` for Rust prelude types:**

```kotlin
rustTemplate(
    "let result: #{Result}<#{String}, #{Error}> = #{Ok}(value);",
    *preludeScope,  // Provides Result, String, Ok
    "Error" to myErrorType
)
```

❌ Wrong: `"let result: Result<String, Error> = Ok(value);"`
✅ Correct: Use `*preludeScope` in templates

## RuntimeType and Dependencies

`RuntimeType` objects contain:

- **`path`**: Rust path (e.g., `"::mime::Mime"`)
- **`dependency`**: `CargoDependency` or `InlineDependency`

Using a `RuntimeType` automatically adds its dependency to `Cargo.toml`.

### Creating RuntimeTypes

```kotlin
// Pre-defined dependencies
val Mime = CargoDependency.Mime.toType()
val Bytes = CargoDependency.Bytes.toType().resolve("Bytes")

// Runtime crates
val smithyTypes = RuntimeType.smithyTypes(runtimeConfig)
```

### Always Use Symbols

❌ Wrong: `rust("const MIME: ::mime::Mime = ::mime::APPLICATION_JSON;")`
✅ Correct: `rustTemplate("const MIME: #{Mime}::Mime = #{Mime}::APPLICATION_JSON;", "Mime" to RuntimeType.Mime)`

### String Interpolation in Templates

**For RuntimeTypes and complex objects**: Use `#{name}` syntax. NOTE: you do not need to use `#{name:W}`. This is now
the default. You may see old code with this pattern.
**For simple strings**: Use `$` with `.dq()` for double-quoted strings

```kotlin
// ❌ Wrong - causes "Invalid type provided to RustSymbolFormatter"
rustTemplate("let content_type = \"#{content_type}\";", "content_type" to "application/json")

// ✅ Correct - use $ interpolation for strings
rustTemplate("let content_type = ${contentType.dq()};")
```

## RuntimeType.forInlineFun: Lazy Generation

Code is only generated if used. `forInlineFun` enables lazy generation:

```kotlin
val mimeType = RuntimeType.forInlineFun("APPLICATION_JSON", module) {
    rustTemplate(
        "pub const APPLICATION_JSON: #{Mime}::Mime = #{Mime}::APPLICATION_JSON;",
        "Mime" to RuntimeType.Mime
    )
}
```

⚠️ **Footgun**: Name collisions mean only one implementation gets generated.

## Git Workflow

**Committing with pre-commit hooks:**

This repo uses pre-commit hooks (formatting, linting, `runtime-versioner`). Some hooks modify files (e.g., reformatting). When a hook modifies files, the commit will fail. To fix:

```bash
git add -u
git commit -m "your message"
```

The `git add -u` stages the hook's modifications, and the second commit attempt will pass. Do **not** use `--no-verify` to skip hooks.

## GitHub CLI Integration

**View issues and PRs:**

```bash
gh issue view <number> --repo smithy-lang/smithy-rs
gh pr view <number> --repo smithy-lang/smithy-rs
gh pr diff <number> --repo smithy-lang/smithy-rs
```

**Debug CI failures:**

```bash
# Get PR status and identify failed checks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smithy-lang/smithy-rs](https://github.com/smithy-lang/smithy-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
