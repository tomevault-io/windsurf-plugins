---
trigger: always_on
description: > **Android Context:** These guidelines apply to Android unit and instrumented tests using test-arranger. Android tests run on a modified JVM (or Android Runtime) with reflection limitations that differ from standard JVM tests. Always register custom arrangers explicitly (see below).
---

# Testing Guidelines for Android (Project Conventions)

> **Android Context:** These guidelines apply to Android unit and instrumented tests using test-arranger. Android tests run on a modified JVM (or Android Runtime) with reflection limitations that differ from standard JVM tests. Always register custom arrangers explicitly (see below).

## ⚠️ ANDROID-SPECIFIC REQUIREMENT: Custom Arranger Registration

**Every custom arranger you create MUST be registered in `arranger.properties`.**

Android's reflection-based auto-discovery of `CustomArranger` subclasses **does not work reliably**. 
If you create a custom arranger without registering it, it will be silently ignored and tests will fail or behave unexpectedly.

### Quick setup:

1. Create or edit `src/androidTest/resources/arranger.properties` (for instrumented tests)
2. Add the `arranger.android.customArrangers` property with a comma-separated list of all custom arranger classes:

```properties
arranger.root=com.example.app
arranger.android.customArrangers=com.example.app.test.ProductArranger,com.example.app.test.UserArranger,com.example.app.test.OrderArranger
```

**DO NOT forget this step when creating a new custom arranger.**

## Test structure

- Always use the **given / when / then** pattern.
- Use the exact section comments:
  - `// given`
  - `// when`
  - `// then`
- Keep the `given` section minimal: set only fields relevant to the scenario.
- Use the test-arranger top-level functions (`some<T>()`, `someObjects<T>(n)`, …) to create test data, avoid mocks and hardcoded values.
- Assertions should reflect **behavior and intent**, not incidental object details.

Example:

``` kotlin
@Test
fun `should create report for product brand`() {
    // given
    val product = some<Product>()

    // when
    val report = sut.createBrandReport(listOf(product))

    // then
    assertThat(report.brand).isEqualTo(product.brand)
}
```

------------------------------------------------------------------------

## Test data generation

**By default, use test-arranger to generate test data.**
Do not hand-build large objects with constructors unless the test truly depends on specific values.
Avoid using mocks, prefer instances filled with random data by the test-arranger.

### Basic usage (Kotlin)

The Kotlin API is a set of top-level functions imported from `com.ocadotechnology.gembus.test`.
The generic ones (`some<X>()`, `someObjects<X>(n)`, `someMatching<X>()`) are `reified`/`inline`, so you pass the type as a type argument.
You do not write `Arranger.some(X.class)` as in Java — you use the wrapping functions:

- `some<X>()` → fully populated random instance
- `some<X>("fieldName")` → instance with given field unset (varargs: `some<X>("a", "b")`)
- `someObjects<X>(n)` → a `Sequence` of `n` instances
- `someMatching<X>({ predicate })` → an instance satisfying the predicate
- `someEmail()`, `someLong()`, `someInt()`, `someText()`, `someString()`, `someBoolean()`
- `someFrom(list)` → random element from a collection

Example:

``` kotlin
// given
val product = some<Product>()
product.brand = "VIP"
```

> Note: `someObjects<X>(n)` returns a `Sequence<X>`. Call `.toList()` when you need a `List`.

------------------------------------------------------------------------

## Adjusting arranged data

Prefer expressing intent by modifying only the fields that matter.

### 1. Mutable adjustment via the DSL (preferred when fields are mutable)

When the properties are `var` (mutable), use the trailing-lambda DSL. Each field assigned is set after random population:

``` kotlin
val product = some<Product> {
    brand = "VIP"
}
```

You can set as many fields as needed inside the block, but every assigned property must be mutable (`var`).

### 2. Kotlin data class `copy()`

If the type is a Kotlin `data class`, prefer its built-in `copy()`:

``` kotlin
val product = some<Product>().copy(brand = "VIP")
```

### 3. Override map (for immutable / non-data types)

When the type is immutable and is not a data class, use the overrides map. Keys are field names, values are suppliers (`() -> Any`):

``` kotlin
val product = some<Product>(
    mapOf(
        "brand" to { "VIP" },
        "price" to { BigDecimal.TEN }
    )
)
```

The same overrides map is also accepted by `someObjects<X>(n, overrides)`.

### Rearranger (copy + selective overrides)

Use the Kotlin **Rearranger** DSL (`com.ocadotechnology.gembus.test.rearrangerkt.Rearranger`) when you already have a valid instance and want to tweak a few fields, and the class is immutable, not a `data class`, or otherwise lacks a convenient `copy()`.
The DSL uses **property references** (`Type::property`), which are refactor-safe and IDE-friendly (renames are tracked):

``` kotlin
val original = some<User>()

// given
val admin = Rearranger.copy(original) {
    User::role set "ADMIN"
    User::active set true
}
```

#### When to prefer Rearranger

- You want to start from a valid domain object
- Only a few fields differ
- You want to keep the rest realistic and consistent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ocadotechnology/test-arranger](https://github.com/ocadotechnology/test-arranger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
