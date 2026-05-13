---
trigger: always_on
description: Zagora produces regular type-safe and error-safe TypeScript functions that encapsulate business logic with robust validation, error handling, and context management -- no special clients or routers, no async overhead or network complexity. Perfect for building type-safe libraries, SDKs, APIs, and internal tooling.
---

# Zagora Library

Zagora produces regular type-safe and error-safe TypeScript functions that encapsulate business logic with robust validation, error handling, and context management -- no special clients or routers, no async overhead or network complexity. Perfect for building type-safe libraries, SDKs, APIs, and internal tooling.

## Quick Example

```ts
import { z } from 'zod';
import { zagora } from 'zagora';

const getUser = zagora()
  .input(z.tuple([z.string(), z.number().default(18), z.string().optional()]))
  .handler((_, name, age, country) => {
    // name: string
    // age: number <-- because there is a default value in schema!
    // country: string | undefined <-- because it's marked as optional in schema!
    return `${name} is ${age}, from ${country || 'unknown'}`
  })
  .callable();

getUser('John', 30);
// => John is 30

// @ts-expect-error -- reported at compile-time AND runtime, invalid second argument
getUser('John', 'foo');

// @ts-expect-error -- reported at compile-time AND runtime, missing required argument
getUser();

// NOTE: fine, because second and third arguments are optional (default or optional)
getUser('Barry') // => Barry is 18, from unknown

getUser('Barry', 25) // => Barry is 25, from unknown
getUser('Barry', 33, 'USA') // => Barry is 33, from USA

const result = getUser('Alice');
if (result.ok) {
  console.log(result.data); // "Alice is 18, from unknown"
} else {
  console.error(result.error.kind);

  console.error(result.error);
  // ^ { kind: 'UNKNOWN_ERROR', message, cause }
  // or
  // ^ { kind: 'VALIDATION_ERROR', message, issues: Schema.Issue[] }
}
```

### With primitive inputs

```ts
// primitive input
const helloUppercased = za
  .input(z.string())
  .handler((_, str) => str.toUpperCase())
  .callable();

const res = helloUppercased('Hello world');

if (res.ok) {
  console.log(res);
  // ^ { ok: true, data: 'HELLO WORLD', error: undefined }
}

// array input
const uppercase = zagora({ autoCallable: true, disableOptions: true })
  .input(z.array(z.string()))
  .handler((arrayOfStrings) => {
    // NOTE: `x` is typed as string too!
    return arrayOfStrings.map((x) => x.toUpperCase());
  })

const upRes = uppercase(['foo', 'bar', 'qux']);
if (upRes.ok) {
  console.log(upRes);
  // ^ { ok: true, data: ['FOO', 'BAR', 'QUX' ] }
}
```

You'll also have access to all the types, utils, and error helpers for type-narrowing through package exports.

```ts
import { 
  isValidationError,
  isInternalError,
  isDefinedError,
  isZagoraError,
} from 'zagora/errors';

import * as ZagoraTypes from 'zagora/types';
import * as zagoraUtils from 'zagora/utils';
```


## Features

### Unmatched Robustness

Zagora achieves 100% test coverage, ensuring every aspect of the library is rigorously tested for reliability and correctness. Complementing this, it includes **dedicated type tests** that utilize `expectType` to verify TypeScript types at compile time. Together, these provide robust guarantees that both the compile-time and runtime systems match, delivering confidence of another level.

### Minimal and Standards-Based

Zagora is lightweight with zero dependencies and bloat, built entirely on [StandardSchema](https://standardschema.dev) for universal validation. This means you can use Zod, Valibot, ArkType, or any compliant validator. No lock-in, just the tools you already know and love.

### Never-Throwing Execution

Every function returns a predictable `{ ok, data, error }` result -- exceptions are eliminated completely. Your process never crashes from unhandled errors, similar to Effect.ts or neverthrow. This gives you total control and deterministic error handling across your entire codebase.

### Typed Errors System

Define error schemas upfront and get strongly-typed error helpers inside your handlers. Each error kind is validated at runtime and fully typed at compile-time. You'll never see `try/catch` blocks or guess error shapes again.

### Full Type Inference

Complete TypeScript inference across inputs, outputs, errors, context, defaults, and optionals. Even JavaScript consumers get full autocomplete and IntelliSense support. The type system has been battle-tested with dedicated type-level tests.

### Multiple Arguments Support

Define multiple function arguments using schema tuples with per-argument validation and defaults. Call your functions naturally like `procedure('Alice', 25)` instead of `procedure({ name: 'Alice', age: 25 })`. This creates a familiar API that feels like native TypeScript functions.

### Granular Diagnostics

Zagora supports compile-time reporting for each argument through TypeScript in IDEs and CLIs, catching potential errors before runtime. This diagnostic capability operates at every level, from schema validation to handler invocationm, to context, to environment variables. Developers receive immediate, precise feedback on argument mismatches, improving code reliability and productivity.

### Sync & Async Awareness at every level


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tunnckoCore/zagora](https://github.com/tunnckoCore/zagora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
