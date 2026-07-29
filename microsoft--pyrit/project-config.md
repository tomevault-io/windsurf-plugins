---
trigger: always_on
description: Follow these standards to keep the frontend consistent, accessible, and maintainable.
---


# PyRIT Frontend Style Guide — TypeScript, React & Fluent UI v9

Follow these standards to keep the frontend consistent, accessible, and maintainable.

## TypeScript

### Strict Mode & `@ts-ignore`
- The project enforces `"strict": true` in tsconfig.
- Do NOT use `@ts-ignore`. It silences the compiler error without fixing the underlying issue, making future type changes silently unsafe. If you absolutely must suppress a check, use `@ts-expect-error` with a comment explaining why.

### Naming Conventions

| Style | Used for |
|---|---|
| `UpperCamelCase` | Components, classes, interfaces, type aliases, enums |
| `lowerCamelCase` | Variables, parameters, functions, methods, properties, hooks |
| `CONSTANT_CASE` | Top-level or static `readonly` constants |

- **Descriptive names**: Names must be clear to a new reader. Do not use ambiguous abbreviations. Exception: loop variables in ≤10-line scopes may use short names.
- **Treat acronyms as words**: `loadHttpUrl`, not `loadHTTPURL`.
- **No `I` prefix on interfaces**: Use `UserProps`, not `IUserProps`.
- **No `_` prefix/suffix on identifiers**: Instead use TypeScript's `private` keyword for visibility. Unused parameters should use destructuring skips (`[a, , b]`) or an `_`-prefixed arg name only when required by a callback signature.

### Variables
- Always use `const` or `let`. Never use `var`.
- Default to `const`. Only use `let` when you need reassignment.

### Equality
- Always use `===` and `!==`. Never use `==` or `!=`.
- Exception: `== null` is allowed to check for both `null` and `undefined`.

### Type Declarations
- **Every** function parameter MUST have an explicit type annotation.
- Return types SHOULD be annotated — especially on exported functions and complex returns. They MAY be omitted when the return is trivially obvious.
- Leave out type annotations for trivially inferred initializers (`string`, `number`, `boolean`, `RegExp` literal, or `new` expression).

```tsx
// CORRECT — type is obvious from the initializer
const name = 'PyRIT'
const count = 0
const items = new Map<string, number>()

// CORRECT — non-obvious, annotate
const config: AppConfig = loadConfig()

// INCORRECT — redundant annotation
const name: string = 'PyRIT'
const count: number = 0
```

- Prefer `interface` for component props and object shapes. Use `type` for unions, intersections, and mapped types.
- Avoid `enum` — use `as const` objects or union literal types instead.

```tsx
// CORRECT
interface ChatMessageProps {
  role: 'user' | 'assistant'
  content: string
  timestamp: string
  attachments?: MessageAttachment[]
}

// INCORRECT — enum
enum Role { User, Assistant }
```

### Array Type Syntax
- Use `T[]` for simple types (alphanumeric): `string[]`, `number[]`, `Message[]`.
- Use `Array<T>` for complex types: `Array<string | number>`, `Array<[string, number]>`.

```tsx
// CORRECT
const names: string[] = []
const pairs: Array<[string, number]> = []

// INCORRECT
const names: Array<string> = []       // use string[] for simple types
const pairs: [string, number][] = []  // hard to read, use Array<>
```

### Type Assertions
- Use `as` syntax, never angle-bracket syntax (`<Foo>x`).
- Prefer type annotations (`: Foo`) over type assertions (`as Foo`) on object literals — assertions bypass excess-property checking.
- Avoid non-nullability assertions (`!`). If you must use one, add a comment explaining why it's safe.
- Prefer `unknown` over `any`. Use `any` only when truly necessary, and add a comment justifying it.

```tsx
// CORRECT — annotation catches extra/missing properties
const user: User = { name: 'Alice', email: 'a@b.com' }

// INCORRECT — assertion silently ignores typos
const user = { name: 'Alice', emial: 'a@b.com' } as User

// CORRECT — safe narrowing
if (value instanceof Error) {
  console.error(value.message)
}

// LAST RESORT — with justification
// Response is guaranteed to have a body by the middleware
const body = response.body!
```

### `readonly`
- Mark properties, fields, and parameters that are never reassigned with `readonly`.

```tsx
interface Config {
  readonly apiUrl: string
  readonly timeout: number
}
```

### Centralized Types
- All shared TypeScript types live in `src/types/index.ts`.
- Component-local types (e.g., internal state shapes) may be defined in the component file.
- Do NOT define shared types in component files.
- Do NOT include `| null` or `| undefined` in type aliases. Add nullability at the point of use.

```tsx
// CORRECT
type CoffeeResponse = Latte | Americano
function getCoffee(): CoffeeResponse | undefined { ... }

// INCORRECT — bakes nullability into the alias
type CoffeeResponse = Latte | Americano | undefined
```

- Use optional fields (`?`) rather than `| undefined` when a property can be absent.

### Import Aliases
- Use the `@/` path alias for imports from `src/`. Do not use deeply nested relative paths.
- Use relative imports (`./foo`) for files within the same feature directory.

```tsx
// CORRECT
import { Message } from '@/types'
import { backendMessagesToFrontend } from '@/utils/messageMapper'
import { useChatWindowStyles } from './ChatWindow.styles' // same directory

// INCORRECT
import { Message } from '../../../types'
```

### Import Organization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/PyRIT](https://github.com/microsoft/PyRIT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
