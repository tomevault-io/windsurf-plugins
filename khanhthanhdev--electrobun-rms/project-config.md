---
trigger: always_on
description: This project uses **Ultracite**, enforcing strict code quality standards through automated formatting and linting. Additionally, this guide covers **OAT UI components**, **React performance best practices**, and **folder structure conventions** for consistent, high-quality code.
---


# Ultracite Code Standards & LLM Development Guide

This project uses **Ultracite**, enforcing strict code quality standards through automated formatting and linting. Additionally, this guide covers **OAT UI components**, **React performance best practices**, and **folder structure conventions** for consistent, high-quality code.

---

## Quick Reference

- **Format code**: `bun x ultracite fix`
- **Check for issues**: `bun x ultracite check`
- **Diagnose setup**: `bun x ultracite doctor`

Biome (the underlying engine) provides robust linting and formatting. Most issues are automatically fixable.

---

## Part 1: Ultracite Code Standards

### Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

#### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

#### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

#### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

#### React & JSX

- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use the `key` prop for elements in iterables (prefer unique IDs over array indices)
- Nest children between opening and closing tags instead of passing as props
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility:
  - Provide meaningful alt text for images
  - Use proper heading hierarchy
  - Add labels for form inputs
  - Include keyboard event handlers alongside mouse events
  - Use semantic elements (`<button>`, `<nav>`, etc.) instead of divs with roles

#### Error Handling & Debugging

- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

#### Code Organization

- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

#### Security

- Add `rel="noopener"` when using `target="_blank"` on links
- Avoid `dangerouslySetInnerHTML` unless absolutely necessary
- Don't use `eval()` or assign directly to `document.cookie`
- Validate and sanitize user input

#### Performance

- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports
- Avoid barrel files (index files that re-export everything)
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

---

## Part 2: React Performance Best Practices

### Priority 1: Eliminating Waterfalls (CRITICAL)

**Goal**: Prevent sequential async operations that could run in parallel.

- **async-parallel**: Use `Promise.all()` for independent async operations
  ```typescript
  // ✗ Bad: Sequential
  const user = await fetchUser();
  const posts = await fetchPosts();
  
  // ✓ Good: Parallel
  const [user, posts] = await Promise.all([
    fetchUser(),
    fetchPosts(),
  ]);
  ```

- **async-defer-await**: Move `await` into branches where actually used
  ```typescript
  // ✗ Bad: Awaiting upfront
  const data = await fetchData();
  if (condition) {
    useData(data);
  }
  
  // ✓ Good: Defer await to usage
  const dataPromise = fetchData();
  if (condition) {
    const data = await dataPromise;
    useData(data);
  }
  ```

- **async-suspense-boundaries**: Use Suspense to stream content progressively
  ```typescript
  // ✓ Good: Lazy load with Suspense
  const LazyComponent = lazy(() => import('./Heavy'));
  export default function App() {
    return (
      <Suspense fallback={<Loading />}>
        <LazyComponent />
      </Suspense>
    );
  }
  ```

### Priority 2: Bundle Size Optimization (CRITICAL)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khanhthanhdev/electrobun-rms](https://github.com/khanhthanhdev/electrobun-rms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
