---
trigger: always_on
description: **PCL (Persona Control Language)** is a domain-specific programming language and compiler for AI persona management. This is a TypeScript-based compiler project that includes:
---

# GitHub Copilot Instructions for PCL (Persona Control Language)

## Project Overview

**PCL (Persona Control Language)** is a domain-specific programming language and compiler for AI persona management. This is a TypeScript-based compiler project that includes:

- **Lexer**: Tokenization and scanning
- **Parser**: AST generation from tokens
- **Semantic Analysis**: Type checking, validation, and symbol resolution
- **Runtime**: Execution engine for PCL programs
- **Standard Library**: Built-in personas, types, and utilities
- **CLI**: Command-line interface for PCL tools

**Tech Stack**: TypeScript 5.3+, Node.js, ESM modules, Vitest for testing

### PCL Bootstrap System

**Important**: This project includes a **PCL-Lite Bootstrap** system that enables AI assistants to interpret `/persona` commands for multi-persona collaboration.

- **Bootstrap File**: `../.roadmap/bootstrap/BOOTSTRAP_EN.md`
- **Purpose**: Embedded runtime v1.0 for AI chat interfaces (ChatGPT, Claude, Gemini, etc.)
- **Personas**: 25+ built-in personas (ARCHI, SEC, DEV, DEVOPS, CRITIC, etc.) with specialized skills
- **Commands**: 120+ `/persona` commands for activation, composition, teams, workflows, and more
- **Domain Focus**: Standardization personas (STANDARD_ARCHITECT, SPEC_EDITOR, COMPLIANCE_ENGINEER, etc.)

When working on PCL code generation, reference the bootstrap specification to understand the target runtime behavior and persona system that PCL compiles to.

**Key Bootstrap Concepts**:

- **Persona Activation**: `/persona [id]` - Activate a persona with specialized capabilities
- **Team Composition**: `/team [id]` - Load pre-configured teams (e.g., security-review, dream-team, standardization)
- **Shared Skills**: Personas inherit foundation, technical, security, architecture, standards, and tools skills
- **Merge Modes**: `primary`, `consensus`, `weighted`, `sequential`, `parallel` for multi-persona responses
- **Workflow Orchestration**: Define and execute multi-step workflows with persona handoffs

---

## Core Architecture Principles

### 1. **Compiler Pipeline**

```
Source Code → Lexer → Parser → Semantic Analyzer → Codegen → Runtime
```

- Each phase is **pure and testable**
- Errors are **collected, not thrown** (use `Result<T, Error[]>` pattern)
- Transformations are **immutable** (return new AST nodes, don't mutate)
- **Position tracking** is mandatory for all AST nodes (for error messages)

### 2. **Type System**

- **Strongly typed** with TypeScript
- Use **discriminated unions** for AST nodes (e.g., `type: 'PersonaDeclaration'`)
- Leverage **branded types** for identifiers (e.g., `PersonaId`, `SkillId`)
- **Nominal typing** over structural where semantics matter

### 3. **Error Handling**

```typescript
// ✅ GOOD: Return Result type
function parse(input: string): Result<AST, ParseError[]> {
  const errors: ParseError[] = [];
  // ... collect errors
  if (errors.length > 0) {
    return { ok: false, errors };
  }
  return { ok: true, value: ast };
}

// ❌ BAD: Throw exceptions in compiler code
function parse(input: string): AST {
  throw new Error('Parse failed'); // Don't do this
}
```

---

## Coding Standards

### TypeScript Guidelines

1. **Strict Mode Always**

   ```typescript
   // tsconfig.json has strict: true
   // No implicit any, no unused variables, etc.
   ```

2. **Prefer `const` over `let`**

   ```typescript
   const tokens = lexer.scan(source); // ✅
   let tokens = lexer.scan(source); // ❌
   ```

3. **Use Readonly for Immutability**

   ```typescript
   interface ASTNode {
     readonly type: string;
     readonly position: Position;
     readonly children: readonly ASTNode[]; // ✅
   }
   ```

4. **Discriminated Unions for AST**

   ```typescript
   type Expression =
     | { type: 'Identifier'; name: string }
     | { type: 'Literal'; value: string | number }
     | { type: 'BinaryOp'; left: Expression; op: string; right: Expression };

   function evaluate(expr: Expression): Value {
     switch (
       expr.type // Type narrowing works!
     ) {
       case 'Identifier':
         return lookupVariable(expr.name);
       case 'Literal':
         return expr.value;
       case 'BinaryOp':
         return evalBinaryOp(expr);
     }
   }
   ```

5. **Brand Types for Domain Concepts**

   ```typescript
   type PersonaId = string & { readonly __brand: 'PersonaId' };
   type SkillId = string & { readonly __brand: 'SkillId' };

   function createPersonaId(id: string): PersonaId {
     return id as PersonaId;
   }
   ```

6. **Avoid `any`, Use `unknown` Instead**

   ```typescript
   function processInput(input: unknown) {
     // ✅
     if (typeof input === 'string') {
       // TypeScript knows input is string here
     }
   }

   function processInput(input: any) {
     // ❌
     // Loses all type safety
   }
   ```

### Naming Conventions

- **Files**: `kebab-case.ts` (e.g., `persona-parser.ts`)
- **Classes**: `PascalCase` (e.g., `PersonaDeclaration`)
- **Interfaces**: `PascalCase` (e.g., `Lexer`, `Parser`)
- **Functions**: `camelCase` (e.g., `parsePersona`, `validateSkills`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_PERSONAS`, `TOKEN_TYPES`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [personamanagmentlayer/pcl](https://github.com/personamanagmentlayer/pcl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
