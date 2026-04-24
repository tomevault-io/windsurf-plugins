---
trigger: always_on
description: A runtime AI agent that monitors BEAM application health and generates actionable analyses.
---

A runtime AI agent that monitors BEAM application health and generates actionable analyses.

## Copywriting

Always refer to this for guidance on copywriting: https://gist.githubusercontent.com/bradleygolden/3426fe9db8cce5652dbda749eafc57b1/raw/2346ff8da345cf5ae6e9401f63edc534f89189be/elixir-library-documentation-manifesto

## Rules

- Zero production impact: all calls must be read-only with no side effects
- No sensitive data: never analyze or expose PII/PHI
- No backwards compatibility: write clean, current code
- Never use @spec
- Never use Process.sleep in tests. Instead rely on deterministic logic
- Never use non-critical comments
- Avoid tautological or disjunctive assertions. Each test should assert exactly one expected outcome. If you're uncertain which outcome to expect, that indicates the test setup needs to be more specific, not that the assertion should be looser.
- NEVER use case statements or any kind of conditional logic in test assertions where the conditional logic is meant to handle different outcomes.
- Never write test-only functions in production code. Prefer adapters/injection for testability (e.g., inject a time function rather than adding `force_transition/1`). Use `:sys.replace_state/2`, direct message sends, or pre-seeded data in tests.
- **Avoid overly prescriptive documentation** - LLMs can infer from guidance and put together insights. Don't hard-code deterministic steps or enumerate every field. Document intent and patterns, not implementation details.
- **Code smell: Hard-coded deterministic steps** - Only use determinism where inference is 100% not needed. Let LLMs synthesize information rather than following step-by-step recipes.

## Conventions

This project uses the puck dependency for its AI agent interface. Please refer to the deps documentation for more information as needed.

- AI agents are just loops that pattern match on structured output in the form of structs. This is capable via Puck

## Elixir/OTP Principles

Write idiomatic Elixir. Trust the BEAM.

- **Let it crash** - Don't over-defend with try/rescue. Let supervisors handle failures. Rescue only at boundaries where you must translate errors.
- **Supervisors for fault tolerance** - Processes should be supervised. Design supervisor trees that isolate failure domains.
- **Processes for isolation** - Use processes to isolate state and failure, not just for concurrency.
- **Pattern matching over conditionals** - Prefer function clause matching over if/cond/case when dispatching on shape.
- **Pipelines over nesting** - Use |> for data transformations. Avoid deeply nested function calls.
- **GenServer for state** - When you need state, reach for GenServer. Keep state minimal and focused.
- **Task for fire-and-forget** - Use Task.Supervisor for work that doesn't need a reply.
- **Immutability is the default** - Don't fight it. Transform data, don't mutate it.
- **Fail fast, fail loud** - Raise on invalid inputs at boundaries. Don't silently return nil or defaults.

# BAML Language Reference

BAML is a domain-specific language for building type-safe LLM prompts as functions. This project uses `baml_elixir` to call BAML functions from Elixir.

## Testing

Run tests defined in `.baml` files:

```bash
baml-cli test                          # Run all tests
baml-cli test -i "MyFunction:TestName" # Run specific test
```

## Types

### Primitive Types
```baml
bool      // true/false
int       // integers
float     // decimal numbers
string    // text
null      // null value
```

### Composite Types
```baml
string[]           // array of strings
int?               // optional int
string | int       // union type
map<string, int>   // key-value map
"a" | "b" | "c"    // literal union
```

### Multimodal Types
```baml
image    // for vision models
audio    // for audio models
video    // for video models
pdf      // for document models
```

### Type Aliases
```baml
type Primitive = int | string | bool | float
type Graph = map<string, string[]>

// Recursive types are supported through containers
type JsonValue = int | string | bool | float | JsonObject | JsonArray
type JsonObject = map<string, JsonValue>
type JsonArray = JsonValue[]
```

## Classes

Classes define structured data. Properties have NO colon.

```baml
class MyObject {
  // Required string
  name string

  // Optional field (use ?)
  nickname string?

  // Field with description (goes AFTER the type)
  age int @description("Age in years")

  // Field with alias (renames for LLM, keeps original in code)
  email string @alias("email_address")

  // Arrays (cannot be optional)
  tags string[]

  // Nested objects
  address Address

  // Enum field
  status Status

  // Union type
  result "success" | "error"

  // Literal types
  version 1 | 2 | 3

  // Map type
  metadata map<string, string>

  // Multimodal
  photo image
}

// Recursive classes are supported
class Node {
  value int
  children Node[]
}
```

### Field Attributes
- `@alias("name")` - Rename field for LLM (keeps original name in code)
- `@description("...")` - Add context for the LLM

### Class Attributes
- `@@dynamic` - Allow adding fields at runtime

## Enums

Enums are for classification tasks with a fixed set of values.

```baml
enum Category {
  PENDING

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beamlens/beamlens](https://github.com/beamlens/beamlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
