---
trigger: always_on
description: These guidelines establish a comprehensive framework for building robust, maintainable TypeScript applications with NestJS. They are designed to promote consistency, readability, and long-term sustainability across complex enterprise applications.
---

# Enhanced TypeScript & NestJS Development Guidelines

These guidelines establish a comprehensive framework for building robust, maintainable TypeScript applications with NestJS. They are designed to promote consistency, readability, and long-term sustainability across complex enterprise applications.

## TypeScript Core Principles

### Fundamental Standards

- Use precise, meaningful English for all code and documentation.
- Employ strong typing throughout the codebase:
  - Explicitly declare types for all variables, parameters, and return values.
  - Avoid `any` type unless absolutely necessary; prefer `unknown` when type is truly undetermined.
  - Create domain-specific types and interfaces to enhance code semantics.
  - Leverage TypeScript's discriminated unions and generics for complex type scenarios.
- Document public APIs thoroughly with JSDoc:
  - Include detailed descriptions, parameter explanations, return value documentation, and examples.Pr
  - Document expected exceptions and edge cases.
- Maintain clean code structure:
  - Eliminate unnecessary blank lines within functions.
  - Limit each file to a single primary export.
  - Keep files concise and focused on a single responsibility.
- Utilize TypeScript's latest features when they improve code quality:
  - Optional chaining (`?.`)
  - Nullish coalescing (`??`)
  - Non-null assertion operator (`!`) only when absolutely necessary
  - Template literal types for complex string manipulation

### Naming Conventions

- Follow consistent casing patterns:
  - `PascalCase` for classes, interfaces, types, enums, and decorators
  - `camelCase` for variables, functions, methods, and properties
  - `kebab-case` for file and directory names
  - `UPPER_SNAKE_CASE` for constants and environment variables
- Create semantic and self-descriptive names:
  - Use domain-specific terminology consistently
  - Begin function names with clear action verbs
  - Prefix boolean variables with verbs: `isEnabled`, `hasPermission`, `canModify`
  - Use complete, correctly spelled words over abbreviations
  - Acceptable abbreviations include:
    - Standard technical terms (API, URL, JWT, UUID)
    - Loop counters (`i`, `j`, `k`)
    - Error handling (`err`, `error`)
    - Context objects (`ctx`, `context`)
    - HTTP paradigms (`req`, `res`, `next`)
- Define domain-specific lexicon for consistent terminology across the application

### Function Design

- Create highly cohesive functions with single responsibility:
  - Maximum 20 logical operations per function
  - Clear input/output contracts
  - Consistent error handling strategy
- Follow semantic naming patterns:
  - Action verbs + object for most functions: `calculateTotal`, `validateInput`
  - Predicates for boolean returns: `isValid`, `hasAccess`, `canProceed`
  - Event handlers: `onSubmit`, `handleChange`
  - State mutations: `updateState`, `saveChanges`, `persistData`
- Optimize for readability and maintainability:
  - Implement early returns for preconditions
  - Extract complex logic to well-named helper functions
  - Limit nesting depth to 2-3 levels
  - Utilize functional programming patterns where appropriate
- Enforce parameter discipline:
  - Use destructured objects for functions with >2 parameters (Receive-object, Return-object pattern)
  - Provide meaningful default values
  - Define clear interface types for complex parameters
  - Implement parameter validation at function boundaries
- Maintain single abstraction level within each function

### Data Management

- Employ rich domain models over primitive obsession:
  - Encapsulate related data in purpose-built types
  - Utilize value objects for concepts with intrinsic rules (Email, Currency, PhoneNumber)
  - Implement domain entities with identity and behavior
- Enforce data integrity:
  - Implement validation at domain boundaries
  - Use class-validator for DTO validation
  - Create factory methods for complex object construction
  - Consider runtime type checking for external data
- Prioritize immutability:
  - Use `readonly` for properties that should not change
  - Apply `as const` for immutable literals and arrays
  - Implement proper copy mechanisms for state updates
  - Consider Immer or similar libraries for complex state management
- Design clear data flow:
  - Distinguish between commands and queries
  - Establish consistent data transformation patterns
  - Define clear boundaries between layers

### Class Architecture

- Adhere to SOLID principles:
  - **S**ingle Responsibility: Each class should have only one reason to change
  - **O**pen/Closed: Open for extension, closed for modification
  - **L**iskov Substitution: Subtypes must be substitutable for their base types
  - **I**nterface Segregation: Many client-specific interfaces are better than one general-purpose interface
  - **D**ependency Inversion: Depend on abstractions, not concretions
- Implement compositional design:
  - Favor composition over inheritance
  - Use mixins or higher-order functions when appropriate
  - Apply the strategy pattern for variant behaviors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codeslw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
