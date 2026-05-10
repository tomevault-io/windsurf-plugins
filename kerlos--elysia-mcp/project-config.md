---
trigger: always_on
description: TITLE: Overwrite Transformations Retaining Schema Type in Zod 4
---

TITLE: Overwrite Transformations Retaining Schema Type in Zod 4
DESCRIPTION: Introduces the new `.overwrite()` method in Zod v4, designed for transformations that do not change the inferred type. This method returns an instance of the original schema class, allowing continued method chaining and retaining introspectability.
SOURCE: https://zod.dev/v4/v4

LANGUAGE: TypeScript
CODE:
```
z.number().overwrite(val => val ** 2).max(100);
// => ZodNumber
```

----------------------------------------

TITLE: Compose Discriminated Unions in Zod 4
DESCRIPTION: Demonstrates the new ability in Zod v4 to use one discriminated union schema (`MyErrors`) as a member within another discriminated union (`MyResult`), enabling powerful schema composition patterns.
SOURCE: https://zod.dev/v4/v4

LANGUAGE: TypeScript
CODE:
```
const BaseError = z.object({ status: z.literal("failed"), message: z.string() });
const MyErrors = z.discriminatedUnion("code", [
  BaseError.extend({ code: z.literal(400) }),
  BaseError.extend({ code: z.literal(401) }),
  BaseError.extend({ code: z.literal(500) })
]);

const MyResult = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  MyErrors
]);
```

----------------------------------------

TITLE: Defining Synchronous Zod-Validated Functions in Zod v4
DESCRIPTION: Demonstrates the new API for defining Zod-validated functions using `z.function()` in Zod v4. It shows how to define input and output schemas upfront and implement the function logic synchronously using `.implement()`.
SOURCE: https://zod.dev/v4/v4/changelog

LANGUAGE: TypeScript
CODE:
```
const myFunction = z.function({
  input: [z.object({
    name: z.string(),
    age: z.number().int(),
  })],
  output: z.string(),
});

myFunction.implement((input) => {
  return `Hello ${input.name}, you are ${input.age} years old.`;
});
```

----------------------------------------

TITLE: Use Top-Level String Format Functions - Zod v4 - JavaScript
DESCRIPTION: Lists the various string format validation functions (like email, uuid, url, etc.) that are now available directly as top-level methods on the `z` module in Zod v4. This change makes them more concise to use and improves tree-shaking.
SOURCE: https://zod.dev/v4/v4

LANGUAGE: JavaScript
CODE:
```
z.email();\nz.uuidv4();\nz.uuidv7();\nz.uuidv8();\nz.ipv4();\nz.ipv6();\nz.cidrv4();\nz.cidrv6();\nz.url();\nz.e164();\nz.base64();\nz.base64url();\nz.jwt();\nz.ascii();\nz.utf8();\nz.lowercase();\nz.iso.date();\nz.iso.datetime();\nz.iso.duration();\nz.iso.time();
```

----------------------------------------

TITLE: Zod 4: Using .check() in zod/v4-mini
DESCRIPTION: Illustrates the use of the new `.check()` method available in `zod/v4-mini`, which allows composing multiple validations and transforms (referred to as 'checks') on a schema.
SOURCE: https://zod.dev/v4/v4/changelog

LANGUAGE: TypeScript
CODE:
```
import { z } from "zod/v4-mini";

z.string().check(
  z.minLength(10),
  z.maxLength(100),
  z.toLowerCase(),
  z.trim(),
);
```

----------------------------------------

TITLE: Customize Required and Invalid Type Errors with Zod 4
DESCRIPTION: Shows how Zod v4 replaces the separate `required_error` and `invalid_type_error` parameters with a single `error` function that receives an issue object, allowing conditional error messages based on the issue type (e.g., `invalid_type` or `required`).
SOURCE: https://zod.dev/v4/v4

LANGUAGE: TypeScript
CODE:
```
// Zod 3
- z.string({
-   required_error: "This field is required"
-   invalid_type_error: "Not a string",
- });
```

LANGUAGE: TypeScript
CODE:
```
// Zod 4
+ z.string({ error: (issue) => issue.input === undefined ?
+  "This field is required" :
+  "Not a string"
+ });
```

----------------------------------------

TITLE: Customize Errors with Function Syntax in Zod 4
DESCRIPTION: Illustrates how Zod v4 replaces the `errorMap` function with the unified `error` function for more complex error customization, allowing access to the issue details to return specific messages based on validation failures like `too_small`.
SOURCE: https://zod.dev/v4/v4

LANGUAGE: TypeScript
CODE:
```
// Zod 3
- z.string({
-   errorMap: (issue, ctx) => {
-     if (issue.code === "too_small") {
-       return { message: `Value must be >${issue.minimum}` };
-     }
-     return { message: ctx.defaultError };
-   },
- });
```

LANGUAGE: TypeScript
CODE:
```
// Zod 4
+ z.string({
+   error: (issue) => {
+     if (issue.code === "too_small") {
+       return `Value must be >${issue.minimum}`
+     }
+   },
+ });
```

----------------------------------------

TITLE: Adding Issues Directly to ZodError (JavaScript)
DESCRIPTION: Shows the recommended way to add new issues to a `ZodError` instance in Zod v4 by directly pushing to the `issues` array, replacing the deprecated `.addIssue()` methods.
SOURCE: https://zod.dev/v4/v4/changelog

LANGUAGE: JavaScript
CODE:
```
myError.issues.push({
  // new issue
});
```

----------------------------------------

TITLE: Configure Zod Error Message Locale - Zod v4 - JavaScript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kerlos/elysia-mcp](https://github.com/kerlos/elysia-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
