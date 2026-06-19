---
trigger: always_on
description: >
---


# Okyline Schema Language v1.7.0

Okyline is a declarative language for describing and validating JSON structures using inline constraints on field names. Schemas are valid JSON documents with real example values.

## ⚠️ Before any schema generation

**MANDATORY**: Read the reference files BEFORE producing an Okyline schema:

1. `references/syntax-reference.md` — complete syntax of constraints
2. `references/internal-references.md` — `$defs` and references (`&Name`)
3. `references/conditional-directives.md` — if conditional logic
4. `references/expression-language.md` — if $compute is necessary
5. `references/virtual-fields.md` — if $field (virtual fields) is necessary
6. `references/external-imports.md` — ONLY if `$deps`/`$import` are explicitly requested or already present in a loaded schema

Never generate a schema based solely on this SKILL.md file.
The examples here are a summary, not an exhaustive reference.

## References: when to use

Use `$defs` + reference (`&Name`) when:
1. **Recursion** (mandatory — only way to express recursive structures)
2. **Repeated structures** identical across multiple usages (e.g. `Period`, `Address`)
3. **Template pattern** — base structure specialized per usage via `$override` or `$amend` (e.g. `Coding`)
4. **Explicit user request**

Default → inline in `$oky`. Don't over-abstract — factorize only when it reduces real duplication or expresses a meaningful shared type.


## Core Syntax

```
"fieldName | constraints | label": exampleValue
```

- **fieldName**: JSON field name
- **constraints**: Validation rules (space-separated)
- **label**: Optional human-readable description
- **exampleValue**: Determines the inferred type

If a label is present without constraints, use `| |`:

  ❌ `"acheteur|Client"` → "Client" parsed as constraint!
  ✅ `"acheteur| |Client"` → "Client" is the label

## Minimal Schema Structure

```json
{
  "$oky": {
    "name|@ {2,50}|User name": "Alice",
    "email|@ ~$Email~": "alice@example.com",
    "age|(18..120)": 30
  }
}
```

## Essential Constraints

| Symbol | Meaning | Example |
|--------|---------|---------|
| `@` | Required field | `"name\|@": "Alice"` |
| `?` | Nullable (can be null) | `"middle\|?": "John"` |
| `{min,max}` | String length | `"code\|{5,10}": "ABC123"` |
| `(min..max)` | Numeric range | `"age\|(18..65)": 30` |
| `('a','b')` | Enum values | `"status\|('ACTIVE','INACTIVE')": "ACTIVE"` |
| `~pattern~` | Regex or format | `"email\|~$Email~": "a@b.com"` |
| `[min,max]` | Array size | `"tags\|[1,5]": ["eco"]` |
| `[*]` | Array (no size constraint) | `"items\|[*]": [1, 2]` |
| `->` | Element validation constraint | `"tags\|[*] -> {2,10}": ["eco"]` |
| `!` | Unique elements | `"codes\|[*]!": ["A","B"]` |
| `#` | Key field (for object uniqueness) | `"id\|#": 123` |
| `&Name` | Reference to definition (value-based, type indicator) | `"address": "&Address"` |


### Open-ended numeric constraints

When only one bound is needed, use comparison operators:

| Syntax | Meaning | Example |
|--------|---------|---------|
| `(>0)` | Strictly positive | `"quantity\|(>0)": 5` |
| `(>=0)` | Positive or zero | `"price\|(>=0)": 29.99` |
| `(<100)` | Strictly < 100 | `"percentage\|(<100)": 50` |
| `(<=100)` | ≤ 100 | `"score\|(<=100)": 85` |

❌ Never invent syntax with missing bound: `(0..)`, `(..100)`
✅ Use comparisons: `(>=0)`, `(<=100)`

❌ Never use huge placeholder bounds: `(0..99999999)`, `(-99999999..100)`
✅ Use comparisons: `(>=0)`, `(<=100)`


⚠️ **INVALID SYNTAX** — Okyline does NOT support open ranges:
- ❌ `(0..)`, `(..100)`, `(1..*)`, `(0..*)`
- These will cause validation errors

### Unbounded collection size

Use `*` to indicate "no limit":

| Syntax | Meaning |
|--------|---------|
| `[*]` | Array with no size constraint |
| `[1,*]` | At least 1 element, no maximum |
| `[~pattern~:*]` | Map with validated keys, unlimited entries |

❌ Never omit a bound: `[1,]`, `[~pattern~:]`
✅ Use `*` explicitly: `[1,*]`, `[~pattern~:*]`


## Built-in Formats

Use with `~$FormatName~`: `$Date`, `$DateTime`, `$Time`, `$Email`, `$Uri`, `$Uuid`, `$Ipv4`, `$Ipv6`, `$Hostname`

## Type Inference Rules

- Type is inferred from example value (no explicit declarations)
- `42` → Integer, `3.14` → Number, `"text"` → String, `true` → Boolean
- Arrays must have at least one element for type inference
- **`null` cannot be used as example** — no type can be inferred. Use `?` for nullable fields with a real example value:

❌ `"middleName|?": null`
✅ `"middleName|?{1,50}": "Marie"`

❌ `"discount|?(0..100)": null`
✅ `"discount|?(0..100)": 15`

### Decimal Values Ending in .00

JSON serializers drop trailing zeros: `78.00` becomes `78`. 
To preserve decimal precision in examples, wrap in quotes:

❌ `"amount": 78.00`   → Serialized as 78, type inferred as Integer
✅ `"amount": "78.00"` → Preserved, type inferred as Number

This only affects decimals with zero fractional parts (.00, .0).
Regular decimals (45.5) and integers (10) are unaffected.


## ⚠️ `?` vs Absence — Critical Distinction

`?` and "optional field" are NOT the same thing in Okyline:

| | Meaning | When to use |
|--|---------|-------------|
| No `@` | Field may be **absent** from the document | FHIR optional fields, nullable in most APIs |
| `?` | Field may be explicitly **`null`** | Only when `null` is a meaningful intentional value |

```json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Okyline/Okyline-skill](https://github.com/Okyline/Okyline-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
