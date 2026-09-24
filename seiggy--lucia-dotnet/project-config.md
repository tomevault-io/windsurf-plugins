---
trigger: always_on
description: applyTo: '*.ts, *.tsx'
---

---
applyTo: '*.ts, *.tsx'
---
Role Definition:
 - TypeScript Language Expert
 - Software Architect
 - Code Quality Specialist

General:
  Description: >
    TypeScript code should be written to maximize readability, maintainability, and correctness
    while minimizing complexity and coupling. Leverage TypeScript's strong typing system,
    embrace modern JavaScript features, and follow established best practices.
  Requirements:
    - Write clear, self-documenting code with TSDoc for public APIs.
    - Keep abstractions simple and focused.
    - Minimize dependencies and coupling.
    - Enable and adhere to strict compiler options.
    - Use ESLint and Prettier for consistent code style and quality.

Compiler and Tooling:
  - Enable Strict Mode:
      ```json
      // tsconfig.json
      {
        "compilerOptions": {
          "strict": true,
          // Other strict flags often enabled with strict:
          // "noImplicitAny": true,
          // "strictNullChecks": true,
          // "strictFunctionTypes": true,
          // "strictBindCallApply": true,
          // "strictPropertyInitialization": true,
          // "noImplicitThis": true,
          // "alwaysStrict": true
        }
      }
      ```
  - Use Linters and Formatters:
    - Integrate ESLint with TypeScript support (e.g., `@typescript-eslint/parser`, `@typescript-eslint/eslint-plugin`).
    - Use Prettier for consistent code formatting.

Type Definitions:
  - Prefer `unknown` over `any`:
      ```typescript
      // Good: Use unknown for values with uncertain types
      function processData(data: unknown) {
        if (typeof data === 'string') {
          console.log(data.toUpperCase());
        }
        // ... more type checks as needed
      }

      // Avoid: Using any bypasses type checking and can hide errors
      function processDataAny(data: any) {
        console.log(data.toUpperCase()); // Potential runtime error if data is not a string
      }
      ```
  - Use `interface` for Public APIs and `type` for Others:
      ```typescript
      // Good: Interface for defining the shape of objects, especially for public APIs or when extension is expected
      export interface UserProfile {
        id: string;
        username: string;
        email?: string;
      }

      // Good: Type for utility types, unions, intersections, or when dealing with primitives/literals
      type UserId = string | number;
      type ComponentState = 'loading' | 'success' | 'error';
      type UserWithPermissions = UserProfile & { permissions: string[] };
      ```
  - Leverage Utility Types:
      ```typescript
      interface Todo {
        title: string;
        description: string;
        completed: boolean;
        createdAt: Date;
      }

      // Good: Use Partial to make all properties optional (e.g., for updates)
      type PartialTodo = Partial<Todo>;
      const todoToUpdate: PartialTodo = { description: "new description" };

      // Good: Use Readonly to make all properties readonly
      type ReadonlyTodo = Readonly<Todo>;
      const stableTodo: ReadonlyTodo = { title: "Stable", description: "Cannot change", completed: false, createdAt: new Date()};

      // Good: Use Pick to select specific properties
      type TodoPreview = Pick<Todo, "title" | "completed">;

      // Good: Use Omit to exclude specific properties
      type TodoCreation = Omit<Todo, "completed" | "createdAt">; // Assuming these are set later
      ```

Immutability:
  - Leverage `readonly` for Immutability:
      ```typescript
      // Good: Using readonly for properties in interfaces and types
      interface Point {
        readonly x: number;
        readonly y: number;
      }
      const p: Point = { x: 10, y: 20 };
      p.x = 5; // Error: Cannot assign to 'x' because it is a read-only property.

      type ImmutableConfig = {
        readonly apiKey: string;
        readonly endpoint: string;
      };

      // Good: Using ReadonlyArray<T> or readonly T[] for immutable arrays
      const immutableArray: ReadonlyArray<number> = [1, 2, 3];
      immutableArray.push(4); // Error: Property 'push' does not exist on type 'readonly number[]'.
      immutableArray[0] = 0; // Error: Index signature in type 'readonly number[]' only permits reading.

      const anotherImmutableArray: readonly number[] = [4, 5, 6];
      anotherImmutableArray.pop(); // Error: Property 'pop' does not exist on type 'readonly number[]'.
      ```

Code Organization:
  - Use Modules for Encapsulation:
    - Organize code into modules using ES6 `import`/`export` syntax.
    - Group related functionality within modules to maintain a clear structure.
  - Avoid Default Exports (Prefer Named Exports):
      ```typescript
      // Good: Using named exports promotes clarity and consistency
      // file: stringUtils.ts
      export function capitalize(str: string): string { /* ... */ }
      export function truncate(str: string, length: number): string { /* ... */ }

      // Usage:
      import { capitalize, truncate } from './stringUtils';

      // Avoid: Default exports can lead to inconsistent naming upon import
      // file: mainHelper.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seiggy/lucia-dotnet](https://github.com/seiggy/lucia-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
