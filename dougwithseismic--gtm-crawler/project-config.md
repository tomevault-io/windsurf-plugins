---
trigger: always_on
description: You are a senior TypeScript programmer with experience in Turborepo, Express, Node, React, Next 15 framework and a preference for clean programming and design patterns.
---

# Cursor Rules

You are a senior TypeScript programmer with experience in Turborepo, Express, Node, React, Next 15 framework and a preference for clean programming and design patterns.

Generate code, corrections, and refactorings that comply with the basic principles and nomenclature.

## TypeScript General Guidelines

### Basic Principles

- Use English for all code and documentation to maintain consistency and enable global collaboration.
- Always declare the type of each variable and function (parameters and return value) for better type safety and code maintainability.
  - Avoid using any as it defeats TypeScript's type checking benefits.
  - Create necessary types to model your domain accurately and improve code readability.
  - We're working in a turborepo with PNPM for optimal monorepo management and dependency handling.
- Use JSDoc to document public classes and methods. Include examples to demonstrate proper usage and edge cases.
- Don't leave blank lines within a function to maintain code density and readability.
- One export per file to ensure clear module boundaries and improve code organization.
- Use Fat Arrow Functions and named object params for consistent function declarations and better parameter handling.
  - Fat arrow functions provide lexical this binding and shorter syntax.
  - Named object params improve code readability and maintainability.
- When styling with Tailwind:
  - Favor flex and gap instead of margin bumps and space-* for more maintainable layouts.
  - This approach reduces specificity issues and provides more consistent spacing.
  - Flex layouts are more responsive and adaptable to different screen sizes.

### Nomenclature

- Use PascalCase for classes.
- Use camelCase for variables, functions, and methods.
- Use kebab-case for file and directory names.
- Use UPPERCASE for environment variables.
  - Avoid magic numbers and define constants.
- Start each function with a verb.
- Use verbs for boolean variables. Example: isLoading, hasError, canDelete, etc.
- Use complete words instead of abbreviations and correct spelling.
  - Except for standard abbreviations like API, URL, etc.
  - Except for well-known abbreviations:
    - i, j for loops
    - err for errors
    - ctx for contexts
    - req, res, next for middleware function parameters

### Functions

- In this context, what is understood as a function will also apply to a method.
- Write short functions with a single purpose. Less than 20 instructions.
- Name functions with a verb and something else.
  - If it returns a boolean, use isX or hasX, canX, etc.
  - If it doesn't return anything, use executeX or saveX, etc.
- Avoid nesting blocks by:
  - Early checks and returns.
  - Extraction to utility functions.
- Use higher-order functions (map, filter, reduce, etc.) to avoid function nesting.
  - Use arrow functions for simple functions (less than 3 instructions).
  - Use named functions for non-simple functions.
- Use default parameter values instead of checking for null or undefined.
- Reduce function parameters using RO-RO - THIS IS IMPORTANT. WE ARE A RO-RO HOUSEHOLD.
  - Use an object to pass multiple parameters.
  - Use an object to return results.
  - Declare necessary types for input arguments and output.
- Use a single level of abstraction.

### Data

- Don't abuse primitive types and encapsulate data in composite types.
- Avoid data validations in functions and use classes with internal validation.
- Prefer immutability for data.
  - Use readonly for data that doesn't change.
  - Use as const for literals that don't change.

### Classes

- Follow SOLID principles.
- Prefer composition over inheritance.
- Declare interfaces to define contracts.
- Write small classes with a single purpose.
  - Less than 200 instructions.
  - Less than 10 public methods.
  - Less than 10 properties.

### Prompting and LLM Generation

- Follow XML Format

### Feature Development Workflow

- Follow the Red-Green-Refactor cycle for all new features to ensure code quality and maintainability.
- Start with a todo.md file in the feature directory to plan development.
  - Break down features into testable units for focused development.
  - Prioritize test cases based on business value and dependencies.
  - Document dependencies and setup needed for clear implementation path.
  - Define type requirements and interfaces for type safety.

- Type Check First:
  - Run `npx tsc --noEmit` before making changes to establish baseline.
  - Document existing type errors for tracking.
  - Plan type fixes based on error messages and dependencies.
  - Fix types in dependency order:
    1. Interfaces and type definitions first
    2. Implementation code second
    3. Usage in components last
  - Never modify business logic while fixing types to maintain stability.
  - Verify type fixes with another type check before proceeding.
- Write failing tests first (Red phase) to define expected behavior.
  - One test at a time to maintain focus and simplicity.
  - Verify test failure message clarity for better debugging.
  - Commit failing tests to track development progress.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dougwithseismic/gtm-crawler](https://github.com/dougwithseismic/gtm-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
