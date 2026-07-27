---
trigger: always_on
description: antlr-ng (ANTLR Next Generation) is a TypeScript-based parser generator that translates grammar files into parser and lexer classes for multiple target languages. It's a production-ready port and enhancement of ANTLR4, supporting 10 target languages: TypeScript, Java, C++, C#, Go, Python3, Dart, Swift, JavaScript, and PHP.
---

# GitHub Copilot Instructions for antlr-ng

## Project Overview

antlr-ng (ANTLR Next Generation) is a TypeScript-based parser generator that translates grammar files into parser and lexer classes for multiple target languages. It's a production-ready port and enhancement of ANTLR4, supporting 10 target languages: TypeScript, Java, C++, C#, Go, Python3, Dart, Swift, JavaScript, and PHP.

**Key characteristics:**
- Production-ready tool generating parsers from ANTLR grammar files
- Supports Unicode 16 for improved internationalization
- Runs in Node.js environment with plans for browser support
- Generates parsers, lexers, listeners, and visitors for multiple programming languages

## Architecture

The codebase is organized into several key directories:

- **src/tool/**: Core tool functionality, grammar processing, error handling
- **src/parse/**: Grammar parsing and AST building
- **src/semantics/**: Semantic analysis and validation
- **src/codegen/**: Code generation engine and target-specific generators
  - **src/codegen/model/**: Output model classes for code generation
  - **src/codegen/target/**: Language-specific target implementations (Java, TypeScript, Python3, etc.)
- **src/automata/**: ATN (Augmented Transition Network) construction and analysis
- **src/analysis/**: Grammar analysis and optimization
- **src/misc/**: Utility classes and helpers
- **src/support/**: Support functions and interfaces
- **src/tree/**: Parse tree representations and walkers
- **tests/**: Test suite using Vitest
- **templates/**: Code generation templates for target languages

## Code Style and Standards

### TypeScript Configuration

- **Target**: ES2022 with Node16 module resolution
- **Module system**: ESM (ES Modules) - always use `.js` extensions in imports
- **Strict mode**: Enabled with strict null checks, no implicit any, no implicit this
- **Declaration files**: Generated during build (emitDeclarationOnly: true)

### ESLint Rules (Key Requirements)

1. **Line length**: Maximum 120 characters
2. **Quotes**: Use double quotes, allow template literals and escaped quotes
3. **Indentation**: 4 spaces
4. **Semicolons**: Always required
5. **Brace style**: 1tbs (one true brace style), no single-line blocks
6. **Curly braces**: Required for all control statements
7. **Arrow functions**: Always use block body with explicit return
8. **Naming conventions**:
   - Classes, enums, type aliases, interfaces: PascalCase
   - Variables, functions, methods: camelCase
   - Enum members: PascalCase
   - Properties: camelCase or UPPER_CASE for constants
9. **Empty lines**: Always add blank line before return statements
10. **Member ordering**: Static fields → instance fields → constructor → public methods → private methods
11. **Explicit accessibility**: Always specify public/protected/private modifiers

### Code Patterns to Follow

```typescript
// ✅ Correct: Arrow function with block body
const transform = (value: string): string => {
    return value.toUpperCase();
};

// ❌ Wrong: Arrow function without block body
const transform = (value: string): string => value.toUpperCase();

// ✅ Correct: Proper spacing and blank line before return
public processGrammar(grammar: Grammar): void {
    const symbols = this.extractSymbols(grammar);
    this.validateSymbols(symbols);

    return;
}

// ✅ Correct: Import with .js extension
import { Grammar } from "./Grammar.js";

// ❌ Wrong: Import without .js extension
import { Grammar } from "./Grammar";
```

## Testing Guidelines

### Test Framework: Vitest

- Test files use `.spec.ts` extension
- Tests located in `tests/` directory
- Test timeout: 10 seconds (configurable via testTimeout)
- Use concurrent test execution when possible
- Test file pattern: `TestXxx.spec.ts` where Xxx describes the feature

### Running Tests

```bash
npm test                          # Run all tests
npm run generate-test-parsers    # Generate test grammar parsers
```

### Test Structure

```typescript
import { describe, it, expect } from "vitest";

describe("FeatureName", () => {
    it("should handle basic case", () => {
        // Arrange
        const input = "test";

        // Act
        const result = processInput(input);

        // Assert
        expect(result).toBe("expected");
    });
});
```

## Build and Development Workflow

### Build Process

```bash
npm run build                    # Full build pipeline
npm run generate-version-file    # Generate version.ts
npm run generate-antlr-parser    # Generate ANTLR grammar parsers
npm run esbuild                  # Bundle with esbuild
tsc -p tsconfig.json            # TypeScript compilation (declarations only)
```

### Development Commands

```bash
npm run TestRig                  # Run TestRig tool
npm run interpreter              # Run interpreter
npm run generate-parser          # Generate parser from grammar
npm run generate-docs            # Generate API documentation with TypeDoc
```

### Code Generation

When working with grammars:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antlr-ng/antlr-ng](https://github.com/antlr-ng/antlr-ng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
