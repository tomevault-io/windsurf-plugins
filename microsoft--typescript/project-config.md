---
trigger: always_on
description: **You MUST read and follow these instructions before taking any coding action in this repository.**
---

# 🚨 CRITICAL: This Repository Is in Maintenance Mode 🚨

**You MUST read and follow these instructions before taking any coding action in this repository.**

This repository (`microsoft/TypeScript`) is the JavaScript-based TypeScript compiler and is **effectively closed for general development**. TypeScript 6.0 is the **last JavaScript-based release**. Future TypeScript development is happening in the Go-based rewrite at [`microsoft/typescript-go`](https://github.com/microsoft/typescript-go).

See [issue #62963](https://github.com/microsoft/TypeScript/issues/62963) and the [TypeScript 7.0 progress blog post](https://devblogs.microsoft.com/typescript/progress-on-typescript-7-december-2025/) for full details.

## ❌ DO NOT create coding PRs for this repository

**Unless the user explicitly acknowledges the above and confirms their change falls into one of these narrow accepted categories:**

- Crashes introduced in 5.9 or 6.0 that **also repro in 7.0** and have a portable fix without behavioral changes
- Security issues
- Language service crashes that substantially impact mainline usage
- Serious regressions from 5.9 that impact a large proportion of users
- Non-disruptive `lib.d.ts` changes

## Required behavior for coding agents

If asked to make code changes or create a PR in this repository, you **MUST**:

1. Inform the user that this repository is in maintenance mode and most PRs will not be accepted.
2. Ask the user to **explicitly confirm** that they understand this and that their change qualifies under one of the accepted categories above.
3. **Refuse to proceed** until that acknowledgement is given.
4. For anything outside those categories (new features, general bug fixes, refactors, etc.), direct the user to [`microsoft/typescript-go`](https://github.com/microsoft/typescript-go) instead.

---

# Guide for Copilot

This document provides a concise guide for writing TypeScript fourslash tests and compiler tests, along with build instructions.

## Build Instructions Summary

### Setup
1. Install Node.js (current or LTS)
2. Clone the repository: `git clone --depth=1 https://github.com/microsoft/TypeScript`
3. Install dependencies: `npm ci`

### Common Build Tasks
```bash
npx hereby local             # Build the compiler into built/local
npx hereby clean             # Delete the built compiler  
npx hereby tests             # Build the test infrastructure
npx hereby runtests          # Run all tests
npx hereby runtests-parallel # Run tests in parallel 🚨 MANDATORY BEFORE FINISHING!
npx hereby runtests --runner=fourslash # Run only fourslash tests
npx hereby runtests --runner=compiler # Run only compiler tests
npx hereby runtests --tests=<testPath> # Run specific test
npx hereby baseline-accept   # Accept new test baselines
npx hereby lint              # Run eslint 🚨 MANDATORY BEFORE FINISHING!
npx hereby format            # Run code formatting 🚨 MANDATORY BEFORE FINISHING!
```

## Fourslash Test Syntax Guide

Fourslash tests are interactive TypeScript language service tests. They validate IDE features like completions, quick info, navigation, and refactoring.

### Basic Structure
```typescript
/// <reference path='fourslash.ts'/>

////code goes here with /*markers*/

// Test assertions go here
```

### Key Syntax Elements

#### 1. Source Code Definition
Use `////` to define source code lines:
```typescript
////function foo(x: number) {
////    return x + 1;
////}
////let result = foo(/*marker*/42);
```

#### 2. Markers for Positioning
Use `/**/` for anonymous markers or `/*name*/` for named markers:
```typescript
////let x = /*1*/someValue;
////let y = /*cursor*/anotherValue;
```

#### 3. Multi-file Tests
Use `// @Filename:` to define multiple files:
```typescript
// @Filename: /a.ts
////export const value = 42;

// @Filename: /b.ts  
////import { value } from './a';
////console.log(/*marker*/value);
```

#### 4. Ranges
Use `[|text|]` to define text ranges:
```typescript
////function test() {
////    [|return 42;|]
////}
```

### Common API Patterns

#### Navigation & Positioning
```typescript
goTo.marker("markerName");         // Navigate to marker
goTo.marker();                     // Navigate to anonymous marker /**/
```

#### Verification (Prefer these over baselines)
```typescript
verify.currentLineContentIs("expected content");
verify.completions({ includes: "itemName" });
verify.completions({ excludes: "itemName" });
verify.quickInfoIs("expected info");
verify.codeFix({
    description: "Fix description",
    newFileContent: "expected content after fix"
});
```

#### Completions Testing
```typescript
verify.completions({ 
    marker: "1",
    includes: { name: "foo", source: "/a", hasAction: true },
    isNewIdentifierLocation: true,
    preferences: { includeCompletionsForModuleExports: true }
});
```

#### Code Fixes Testing
```typescript
verify.codeFix({
    description: "Add missing property",
    index: 0,
    newFileContent: `class C {
    property: string;
    method() { this.property = "value"; }
}`
});
```

#### Formatting
```typescript
format.document();
verify.currentLineContentIs("formatted content");
```

### Simple Example
```typescript
/// <reference path='fourslash.ts'/>

////interface User {
////    name: string;
////}
////

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/TypeScript](https://github.com/microsoft/TypeScript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
