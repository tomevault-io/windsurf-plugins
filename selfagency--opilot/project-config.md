---
trigger: always_on
description: Welcome! This document guides you through the recommended workflows, conventions, and tooling when working on the **Opilot** VS Code extension project. Follow these instructions strictly to ensure consistency, quality, and smooth collaboration.
---

# Agent Instructions: Visual Studio Code Extension Integrating Ollama into GitHub Copilot Chat

Welcome! This document guides you through the recommended workflows, conventions, and tooling when working on the **Opilot** VS Code extension project. Follow these instructions strictly to ensure consistency, quality, and smooth collaboration.

---

## Before Starting Work

- **Create or switch to the correct branch before editing code.**
- Branch name format: `[type]/[short-title]` (e.g., `feat/add-ollama-integration`).

---

## Project Overview

You are working on the **Opilot** extension, which integrates Ollama models into GitHub Copilot Chat inside VS Code.

### Key Features to Keep in Mind

- Local and cloud Ollama model usage inside Copilot Chat.
- Custom Ollama sidebar for model management.
- `@ollama` chat participant for dedicated conversations.
- Inline code completions using local models.
- Modelfile creation, editing, and building with syntax support.
- Streaming responses and vision model support.
- Local execution for privacy.
- Configuration via VS Code settings.

---

## Development Environment Setup

- **Prerequisites:**

  - Node.js 20+
  - pnpm (version pinned in `package.json`)
  - VS Code 1.109.0 or higher
  - GitHub Copilot Chat extension installed
  - Ollama installed locally or remote access configured

- **Installing Ollama:**

  - Download from [https://ollama.ai/download](https://ollama.ai/download)
  - Start Ollama app or run `ollama serve`
  - Login to Ollama Cloud if using cloud models (`ollama login`)

- **Extension Installation:**

  - Install from VS Code Marketplace or `.vsix` file

---

## Code Quality Standards

This project uses **Ultracite**, a zero-config preset built on **Biome** that enforces strict code quality standards through automated formatting and linting. The project has migrated from oxlint/oxfmt to Biome via Ultracite, so `check-formatting` and `format` tasks are no longer needed — `ultracite check` and `ultracite fix` cover both linting and formatting in a single command.

### Quick Reference

- **Lint and format code**: `pnpm dlx ultracite fix`
- **Check for issues**: `pnpm dlx ultracite check`
- **Diagnose setup**: `pnpm dlx ultracite doctor`

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selfagency/opilot](https://github.com/selfagency/opilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
