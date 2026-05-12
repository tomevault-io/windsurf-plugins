---
trigger: always_on
description: This file defines the strict quality and development standards for this codebase. All rules must be followed at all times.
---

# CLAUDE CODE RULES

This file defines the strict quality and development standards for this codebase. All rules must be followed at all times.

---

## GLOBAL FRONTEND DEVELOPMENT RULES

### 1. TypeScript Strictness

- NEVER use `any`. This is a strict rule.
- Always type variables, function parameters, and return types correctly.
- Use precise Interfaces, Types, or Generics. If a type is truly unknown, use `unknown` and handle it properly.
- **Interfaces vs Types:** Use `interface` for core entities and object shapes. Use `type` ONLY for unions, intersections, and complex compositions.
- **Immutability:** Strictly use `as const` for fixed configuration objects and arrays to enforce literal typing.

### 2. Commenting Rules (Strict Format)

- **Language:** Write all comments exclusively in ENGLISH.
- **Length:** Comments must be extremely concise (1 or 2 lines maximum).
- **Content:** Only write technical comments that explain _why_ a complex logic is implemented, not _what_ the code does.
- **PROHIBITED FORMATS:**
  - NO numbered lists inside code comments.
  - NO bullet points or dashes.
  - NO decorative lines, ASCII art, or separators.
  - Write standard, inline sentences: `// Brief technical explanation here.`

### 3. Preserving Code, Diffing & Renaming

- **No Random Renaming:** NEVER arbitrarily change the names of existing functions, variables, or hooks. If a function is named `fetchData`, keep it `fetchData` unless explicitly asked to rename it.
- **Surgical Changes:** When modifying an existing file, alter ONLY the strictly necessary lines required to complete the task. Do not reformat, re-indent, or modify untouched surrounding code.
- **Preserve Comments:** DO NOT delete, alter, or format existing structural comments.
- Leave existing code structure completely untouched unless specifically instructed to refactor that exact block.

### 4. No Lazy Coding

- NEVER use placeholders like `// ... existing code ...` or `// implement logic here`.
- Always output the complete, functional code block required for the change, without truncating functions or objects.

### 5. Imports & Dependencies

- **Absolute Imports Only:** Use ABSOLUTE paths exclusively (e.g., `@/components/Button`). NEVER use relative paths like `../../utils` or `./components`.
- **No Hallucinations:** Use ONLY the libraries and dependencies already present in the codebase. Do not invent or import external packages unless explicitly requested.

### 6. React Architecture & Patterns

- Use Named Exports exclusively for components, functions, and hooks. Avoid `export default` (unless explicitly required by Next.js routing files like `page.tsx` or `layout.tsx`).
- DO NOT use `React.FC` or `React.FunctionComponent`. Type your component props directly (e.g., `function MyComponent({ prop }: MyProps)`).
- **Naming Convention:** Components MUST be `PascalCase`. Hooks and utility functions MUST be `camelCase`. Constants MUST be `UPPER_SNAKE_CASE`.
- **Context Safety:** Custom hooks consuming React Context MUST check if the context is `undefined` and throw a clear error if used outside their Provider.

### 7. Error Handling

- NEVER swallow errors silently.
- Using `try { ... } catch (e) { console.log(e) }` is strictly prohibited. You must handle errors properly, display them to the user if necessary, or `throw` them up the chain.
- **Error Chaining:** When catching and re-throwing errors, ALWAYS preserve the original trace using the `cause` property: `throw new Error('Action failed', { cause: error });`.

### 8. Naming Conventions & Magic Numbers

- **No Magic Values:** NEVER use hardcoded "magic numbers" or "magic strings" in business logic, computations, or conditions. Extract them into descriptive constants in `src/constants/` or the relevant module's `constants.ts` (e.g., `const MAX_RETRIES = 3;`, `const SECONDS_PER_HOUR = 3600;`).
- **What IS a magic number:** time durations, thresholds, API status codes, protocol values, format strings, locale identifiers, mathematical multipliers — any literal whose meaning is not immediately obvious from context.
- **What is NOT a magic number:** Tailwind CSS class values (`w-8`, `gap-2`, `grid-cols-4`), SVG attributes (`viewBox`, `cx`, `strokeWidth`, `d` paths), Framer Motion transition configs, `padStart(2, '0')`, and trivial `0`/`1` in boolean-like expressions. These are presentational values and stay inline.
- **Booleans:** Boolean variables must always start with a descriptive prefix like `is`, `has`, `should`, or `can` (e.g., `isValid`, `hasError`).

### 9. Styling (Strict Tailwind CSS)

- **Tailwind Only:** Exclusively use Tailwind CSS classes for styling. NEVER write inline CSS (`style={{...}}`).
- **No Arbitrary Values:** STRICT PROHIBITION on arbitrary dimension values (e.g., DO NOT use `w-[50px]`, `h-[20px]`, `gap-[15px]`). You MUST use standard Tailwind sizing/spacing classes (e.g., `w-12`, `h-5`, `gap-4`).
- **Merge Utilities:** Use a merge utility like `cn()` or `clsx` to concatenate dynamic or conditional classes cleanly.

### 10. General Best Practices

- **Early Returns:** Avoid deeply nested `if/else` pyramids. Use guard clauses to exit early.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matteocelani/f1-telemetry](https://github.com/matteocelani/f1-telemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
