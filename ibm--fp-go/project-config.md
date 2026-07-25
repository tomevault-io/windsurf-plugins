---
trigger: always_on
description: This document provides guidelines for AI agents working on the fp-go/v2 project.
---

# Agent Guidelines for fp-go/v2

This document provides guidelines for AI agents working on the fp-go/v2 project.

## Table of Contents

- [Documentation Standards](#documentation-standards)
  - [Go Doc Comments](#go-doc-comments)
  - [File Headers](#file-headers)
- [Testing Standards](#testing-standards)
  - [Test Structure](#test-structure)
  - [Test Coverage](#test-coverage)
  - [Example Test Pattern](#example-test-pattern)
- [Code Style](#code-style)
  - [Functional Patterns](#functional-patterns)
  - [Error Handling](#error-handling)
- [Checklist for New Code](#checklist-for-new-code)

## Documentation Standards

### Go Doc Comments

1. **Use Standard Go Doc Format**
   - Do NOT use markdown-style links like `[text](url)`
   - Do NOT use markdown-style headers like `# Section` or `## Subsection`
   - Use simple type references: `ReaderResult`, `Validate[I, A]`, `validation.Success`
   - Go's documentation system will automatically create links
   - Use plain text with blank lines to separate sections

2. **Structure**
   ```go
   // FunctionName does something useful.
   //
   // Longer description explaining the purpose and behavior.
   //
   // Type Parameters:
   //   - T: Description of type parameter
   //
   // Parameters:
   //   - param: Description of parameter
   //
   // Returns:
   //   - ReturnType: Description of return value
   //
   // See Also:
   //   - RelatedFunction: Brief description
   func FunctionName[T any](param T) ReturnType {
   ```

3. **Code Examples**
   - Do NOT include code examples in function documentation strings
   - Instead, create `ExampleXXX` functions following Go's example function conventions
   - Example functions should be placed in `*_test.go` files
   - Use the naming pattern `Example<FunctionName>` or `Example<FunctionName>_<scenario>`
   - Example functions must include `// Output:` comments to be recognized by `go test`
   - Example function code should follow these guidelines:
     - Use idiomatic Go patterns
     - Prefer `result.Eitherize1(strconv.Atoi)` over manual error handling
     - Prefer `strconv.Itoa` over `func(x int) string { return fmt.Sprintf("%d", x) }`
     - Prefer direct function references such as `strconv.Itoa` over redundant wrappers like `func(x int) string { return strconv.Itoa(x) }`
     - Prefer reusable combinators over ad-hoc closures when an equivalent helper exists
     - Prefer `F.Flow2(option.Predicate(N.MoreThan(0)), option.Map(N.Mul(2)))` over `func(x int) Option[int] { if x > 0 { return Some(x * 2) }; return None[int]() }` in non-idiomatic option examples
     - Prefer `S.Format[T]("...")` over `func(x T) string { return fmt.Sprintf("...", x) }`, matching the type parameter to the formatted value
     - Show realistic, runnable examples
   - Example:
     ```go
     // ExampleFunctionName demonstrates basic usage.
     func ExampleFunctionName() {
         result := FunctionName(42)
         fmt.Println(result)
         // Output: 42
     }
     ```

4. **Pipe and Flow in Example Functions**

   In the non-idiomatic `option` (and similar) packages, operators have the shape
   `func(T) U`, so `F.Pipe1` works and is preferred over direct nesting:

   ```go
   // Preferred
   result := F.Pipe1(Some(42), Map(strconv.Itoa))

   // Avoid
   result := Map(strconv.Itoa)(Some(42))
   ```

   In the **idiomatic** option package, operators have the shape `func(A, bool) (B, bool)`
   (an `Operator[A, B]`). These take two arguments, so `F.Pipe1` cannot be used.
   Instead, build the pipeline with `F.FlowN` and apply it to the initial tuple via
   Go's multi-return spreading:

   ```go
   // Preferred — Flow composes Operators; the tuple (S, bool) is spread into the call
   result, ok := F.Flow3(
       Bind(setX, func(s State) (int, bool) { return Some(10) }),
       Bind(setY, func(s State) (int, bool) { return Some(20) }),
       Map(computeSum),
   )(Do(State{}))

   // Avoid — F.Pipe3 requires a single first argument; Do returns (S, bool) — two values
   result := F.Pipe3(Do(State{}), Bind(...), Bind(...), Map(...))
   ```

5. **Default / fallback values in Example Functions**

   Use `lazy.Of(value)` instead of an inline zero-returning closure wherever a
   `func() T` is needed for a default:

   ```go
   // Preferred
   GetOrElse(lazy.Of(0))(opt)

   // Avoid
   GetOrElse(func() int { return 0 })(opt)
   ```

6. **Array Initialization in Example Functions**

   In example functions, prefer `A.From` over array literal syntax to avoid
   duplicating type parameters:

   ```go
   // Preferred — type is inferred from elements
   options := A.From(
       Some(1),
       None[int](),
       Some(3),
   )

   // Avoid — duplicates the type parameter
   options := []Option[int]{
       Some(1),
       None[int](),
       Some(3),
   }
   ```

   This applies to all monadic types: `Option[T]`, `Either[E, A]`, `Result[A]`, `Validation[E, A]`.

   **Note**: In test code within packages that would create circular dependencies with
   the `array` package (e.g., `option` package tests), continue using array literal syntax.
   Only example functions should use `A.From`.

### File Headers

Always include the Apache 2.0 license header:

```go
// Copyright (c) 2023 - 2025 IBM Corp.
// All rights reserved.
//

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBM/fp-go](https://github.com/IBM/fp-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
