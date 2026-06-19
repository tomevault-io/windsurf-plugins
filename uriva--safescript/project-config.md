---
trigger: always_on
description: Write and run safescript programs. Safescript is a sandboxed language with static analysis — before code runs, you can see exactly which hosts it contacts, which secrets it reads/writes, and its resource bounds.
---


# Safescript Language Reference

Safescript is a small, sandboxed language designed for secure automation. Every
program is statically analyzed before execution, so you always know what network
hosts it contacts, what secrets it reads or writes, and its resource bounds.
There are no escape hatches.

## Quick Start

Use `analyze_safescript` to inspect a script before running it. Use
`run_safescript` to execute a function from a script. Community skills may also
expose individual safescript functions as callable tools directly.

## Program Structure

A program is zero or more imports and zero or more function definitions. Imports
and functions can appear in any order at the top level.

```
import helperFn from "https://example.com/lib.ss" perms { hosts: ["api.example.com"] } hash "sha256:abc123..."

myFunction = (name: string, count: number): string => {
  result = stringConcat({ parts: [name, " x", jsonStringify({ value: count }).text] })
  return result.result
}
```

## No Semicolons

Safescript does not use semicolons. Statements are separated by newlines or
whitespace.

## Function Definitions

```
functionName = (param1: Type1, param2: Type2): ReturnType => {
  // body statements
  return expression
}
```

Every function must end with `return expression`. There is no early return —
`return` cannot appear inside `if`/`else` blocks, only as the final item in the
function body. Return type annotation is optional.

## Types

Primitives: `string`, `number`, `boolean`

Arrays: `string[]`, `number[]`, `boolean[]`, or nested: `string[][]`

Objects: `{ fieldName: Type, otherField: Type }`

Nested: `{ items: { name: string, score: number }[], total: number }`

Array of objects: `{ name: string }[]`

## Expressions

### Literals

- Strings: `"hello"` (double quotes only, no single quotes, no template
  literals)
- Numbers: `42`, `3.14` (no scientific notation, no hex/octal/binary)
- Booleans: `true`, `false`
- Arrays: `[1, 2, 3]`, `["a", "b"]`, `[]`
- Objects: `{ name: "alice", age: 30 }`

### String Escape Sequences

`\n` (newline), `\t` (tab), `\\` (backslash), `\"` (double quote)

### Object Shorthand

`{ name }` is equivalent to `{ name: name }`. String keys are also allowed:
`{ "content-type": value }`.

### Operators (by precedence, lowest first)

1. Ternary: `condition ? thenExpr : elseExpr`
2. Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`
3. Additive: `+` (numbers or string concatenation), `-`
4. Multiplicative: `*`, `/`, `%`
5. Unary: `-x`
6. Dot access: `obj.field`, chainable: `a.b.c`

String concatenation with `+` only works when BOTH operands are strings. Use
`stringConcat` for joining multiple parts.

### Field Access

Access object fields with dot notation: `response.status`, `result.body`.
Chainable: `identity.keys.signingPublicKey`. Keywords are allowed as field
names: `obj.hash`, `obj.map`, `obj.return`.

## Statements

Three kinds of statements:

### Assignment

```
x = someExpression
```

### Void Call (calling an op without capturing the result)

```
writeSecret({ name: "my-key", value: newValue })
```

### If/Else

```
if condition {
  x = "yes"
} else {
  x = "no"
}
```

No `else if` — nest instead: `else { if ... { } }`. Variables assigned inside
blocks are visible after the block.

## Calling Built-in Ops

Ops are the only way to perform I/O, crypto, and data transformation. They are
called with named arguments in an object:

```
result = httpRequest({ host: "api.example.com", method: "GET", path: "/data" })
parsed = jsonParse({ text: result.body })
```

Some ops support a shorthand (unary sugar) when they take a single argument:

```
h = sha256("hello")           // same as sha256({ data: "hello" })
t = jsonStringify(myObj)       // same as jsonStringify({ value: myObj })
parts = stringConcat(myArray)  // same as stringConcat({ parts: myArray })
```

You CANNOT call user-defined functions directly. They can only be invoked
through `map`, `filter`, or `reduce`.

## Iteration with map, filter, reduce

No loops exist. Use these instead:

### map(functionName, array)

Applies a 1-parameter function to each element. Runs in parallel.

```
double = (n: number): number => {
  return n * 2
}
main = (nums: number[]): number[] => {
  return map(double, nums)
}
```

### filter(functionName, array)

Keeps elements where a 1-parameter function returns truthy. Runs in parallel.

```
isPositive = (n: number): boolean => {
  return n > 0
}
main = (nums: number[]): number[] => {
  return filter(isPositive, nums)
}
```

### reduce(functionName, initial, array)

Folds an array with a 2-parameter function (accumulator, element). Runs
sequentially.

```
add = (acc: number, n: number): number => {
  return acc + n
}
main = (nums: number[]): number => {
  return reduce(add, 0, nums)
}
```

The first argument is always a function NAME (identifier), not an inline
expression.

## Override (DAG composition with substitution)

`override(target, { name: replacement, ... })` returns a first-class DAG value
that behaves like the user function `target`, but with every reference to `name`
(a builtin op label or another user-fn name) rewritten to `replacement` (a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uriva/safescript](https://github.com/uriva/safescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
