---
trigger: always_on
description: When building generic functions, you may need to use any inside the function
---


# Any inside generic functions

When building generic functions, you may need to use any inside the function
body.

This is because TypeScript often cannot match your runtime logic to the logic
done inside your types.

One example:

```ts
const youSayGoodbyeISayHello = <TInput extends "hello" | "goodbye">(
  input: TInput
): TInput extends "hello" ? "goodbye" : "hello" => {
  if (input === "goodbye") {
    return "hello"; // Error!
  } else {
    return "goodbye"; // Error!
  }
};
```

On the type level (and the runtime), this function returns `goodbye` when the
input is `hello`.

There is no way to make this work concisely in TypeScript.

So using `any` is the most concise solution:

```ts
const youSayGoodbyeISayHello = <TInput extends "hello" | "goodbye">(
  input: TInput
): TInput extends "hello" ? "goodbye" : "hello" => {
  if (input === "goodbye") {
    return "hello" as any;
  } else {
    return "goodbye" as any;
  }
};
```

Outside of generic functions, use `any` extremely sparingly.

# Default exports

Unless explicitly required by the framework, do not use default exports.

```ts
// BAD
export default function myFunction() {
  return <div>Hello</div>;
}
```

```ts
// GOOD
export function myFunction() {
  return <div>Hello</div>;
}
```

Default exports create confusion from the importing file.

```ts
// BAD
import myFunction from "./myFunction";
```

```ts
// GOOD
import { myFunction } from "./myFunction";
```

There are certain situations where a framework may require a default export. For
instance, Next.js requires a default export for pages.

```tsx
// This is fine, if required by the framework
export default function MyPage() {
  return <div>Hello</div>;
}
```

# Discriminated unions

Proactively use discriminated unions to model data that can be in one of a few
different shapes.

For example, when sending events between environments:

```ts
type UserCreatedEvent = {
  type: "user.created";
  data: { id: string; email: string };
};

type UserDeletedEvent = {
  type: "user.deleted";
  data: { id: string };
};

type Event = UserCreatedEvent | UserDeletedEvent;
```

Use switch statements to handle the results of discriminated unions:

```ts
const handleEvent = (event: Event) => {
  switch (event.type) {
    case "user.created":
      console.log(event.data.email);
      break;
    case "user.deleted":
      console.log(event.data.id);
      break;
  }
};
```

Use discriminated unions to prevent the 'bag of optionals' problem.

For example, when describing a fetching state:

```ts
// BAD - allows impossible states
type FetchingState<TData> = {
  status: "idle" | "loading" | "success" | "error";
  data?: TData;
  error?: Error;
};

// GOOD - prevents impossible states
type FetchingState<TData> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: TData }
  | { status: "error"; error: Error };
```

# Enums

Do not introduce new enums into the codebase. Retain existing enums.

If you require enum-like behaviour, use an `as const` object:

```ts
const backendToFrontendEnum = {
  xs: "EXTRA_SMALL",
  sm: "SMALL",
  md: "MEDIUM",
} as const;

type LowerCaseEnum = keyof typeof backendToFrontendEnum; // "xs" | "sm" | "md"

type UpperCaseEnum = (typeof backendToFrontendEnum)[LowerCaseEnum]; // "EXTRA_SMALL" | "SMALL" | "MEDIUM"
```

Remember that numeric enums behave differently to string enums. Numeric enums
produce a reverse mapping:

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

const direction = Direction.Up; // 0
const directionName = Direction[0]; // "Up"
```

This means that the enum `Direction` above will have eight keys instead of four.

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

Object.keys(Direction).length; // 8
```

# Import type

Use import type whenever you are importing a type.

Prefer top-level `import type` over inline `import { type ... }`.

```ts
// BAD
import { type User } from "./user";
```

```ts
// GOOD
import type { User } from "./user";
```

The reason for this is that in certain environments, the first version's import
will not be erased. So you'll be left with:

```ts
// Before transpilation
import { type User } from "./user";

// After transpilation
import "./user";
```

# Installing packages

When installing libraries, do not rely on your own training data.

Your training data has a cut-off date. You're probably not aware of all of the
latest developments in the JavaScript and TypeScript world.

This means that instead of picking a version manually (via updating the
`package.json` file), you should use a script to install the latest version of a
library.

```bash
bun add -D @typescript-eslint/eslint-plugin
```

This will ensure you're always using the latest version.

Prefer to install packages, not in the root, but in the mono repo packages

# Interface extends

ALWAYS prefer interfaces when modelling inheritance.

The `&` operator has terrible performance in TypeScript. Only use it where
`interface extends` is not possible.

```ts
// BAD

type A = {
  a: string;
};

type B = {
  b: string;
};

type C = A & B;
```

```ts
// GOOD

interface A {
  a: string;
}

interface B {
  b: string;
}

interface C extends A, B {
  // Additional properties can be added here
}
```

# Jsdoc


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [settlemint/sdk](https://github.com/settlemint/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
