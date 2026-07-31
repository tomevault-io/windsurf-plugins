---
trigger: always_on
description: 1. [Overview](#1-overview)
---

# Project: quadprog

## Table of Contents
1. [Overview](#1-overview)
2. [Mathematical Foundation & Implementation](#2-mathematical-foundation--implementation)
3. [Project Standards & Priority](#3-project-standards--priority)
4. [Development Guidelines](#4-development-guidelines)
    - [JavaScript Style Guide (Enforced by ESLint)](#javascript-style-guide-enforced-by-eslint)
    - [Type Safety & JSDoc](#type-safety--jsdoc)
    - [Testing & Quality Assurance](#testing--quality-assurance)
    - [Error Handling](#error-handling)
    - [Mandatory Pre-Commit Workflow (STRICT ENFORCEMENT)](#mandatory-pre-commit-workflow-strict-enforcement)
    - [Commit Messages](#commit-messages)
    - [Documentation Maintenance](#documentation-maintenance)
    - [Versioning & Git Tags](#versioning--git-tags)
5. [Project Layout](#5-project-layout)

## 1. Overview
A JavaScript library for solving quadratic programming problems using the Goldfarb-Idnani dual method.

## 2. Mathematical Foundation & Implementation
This library is a direct port of the dual active-set algorithm described by Goldfarb and Idnani (1983).

### A. Core Algorithm
The implementation strictly adheres to the dual method for solving strictly convex quadratic programs:
- **Starting Point:** It utilizes the unconstrained minimum of the objective function as the initial point.
- **Active-Set Type:** It iteratively adds violated constraints to the active set until dual feasibility (and thus optimality) is achieved.
- **Numerical Stability:** It employs Cholesky factorizations (`dpofa`, `dposl`, `dpori`) and stable updating procedures instead of explicit matrix inversions, as recommended in Section 4 of the paper.
- **Performance Representation:** Internal caches may use active-constraint markers to reduce JavaScript runtime overhead, but these optimizations must preserve the original active-set selection rules, 1-based logical indexing, and observable solver results.

### B. Compatibility & Conventions
- **R Package Alignment:** Variable names and API structures (e.g., `unconstrained_solution`) are preserved to maintain alignment with the canonical R package `quadprog` and the original Fortran implementation.
- **Indexing:** The core logic (`lib/qpgen2.js`) maintains Fortran-style 1-based indexing by padding arrays. This ensures exact correspondence with the mathematical steps outlined in the original paper.
- **Control Flow:** The use of labeled functions (`fnGoto50`, etc.) in the core solver is intentional to preserve the validated logic flow of the original algorithm.

## 3. Project Standards & Priority
These directives define the project-specific standards and override any conflicting global defaults.

### A. Priority Directive
In case of any conflict between this file and global agent instructions, project rules override global defaults **except** non-negotiable environment and formatting constraints (OS, shell, EOL, encoding) defined in the global agent configuration.

## 4. Development Guidelines

### JavaScript Style Guide (Enforced by ESLint)
The project follows a strict coding standard that mirrors the high-quality defaults used by the **ESLint core project**. These rules are automatically enforced via `npm run lint`.
- **Indentation:** 4 spaces (no tabs).
- **Quotes:** Double quotes (`"`) for all strings, unless avoiding escapes.
- **Spacing:** Mandatory spaces around operators (`a + b`) and after keywords (`if (`, `for (`).
- **Blank Lines:**
    - A mandatory blank line MUST follow any variable declaration (`const`, `let`) if the next statement is not another declaration.
    - Consecutive declarations in the same block do not require blank lines between them.
- **Block Braces:** Mandatory curly braces `{}` for all control statements (`if`, `for`, `while`), even for single-line blocks.
- **Strict Logic & Consistency:**
    - **Equality:** Always use `===` and `!==`.
    - **Imports:** Duplicate imports are forbidden (`no-duplicate-imports`).
    - **Scoping:** Variable shadowing (`no-shadow`) and usage before definition (`no-use-before-define`) are restricted.
    - **Clean Code:** Forbidden redundant logic like useless returns, concatenated constants, or empty blocks.
- **Constants:** Use `const` by default; use `let` only when reassignment is necessary. `var` is forbidden.

### Type Safety & JSDoc
The project uses a hybrid JavaScript/TypeScript approach to ensure type safety and developer productivity without a full compilation step.
- **Global Types:** Core domain entities are defined in a global `types.d.ts` file. These types are ambiently available across the entire project.
- **Mandatory Annotations:**
    - All function parameters and return values MUST be documented using JSDoc (`@param`, `@returns`) using the global types.
    - Class properties and complex variables (like empty arrays destined for specific objects) MUST have `@type` annotations.
    - **Avoid `any`:** The use of `/** @type {any} */` or any cast to `any` is strictly FORBIDDEN unless absolutely impossible to avoid (e.g., third-party library limitations).
    - **TypeScript Directives:** Always use `@ts-expect-error` instead of `@ts-ignore` to suppress inevitable type errors. This ensures that if the underlying issue is resolved, the directive will trigger a new error, forcing its removal.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [albertosantini/quadprog](https://github.com/albertosantini/quadprog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
