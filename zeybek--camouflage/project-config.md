---
trigger: always_on
description: Project architecture, directory structure, module responsibilities, and dependency rules
---


# Project Architecture

## Directory Structure

```
camouflage/
├── src/
│   ├── core/              # Business logic
│   │   └── camouflage.ts  # Main engine
│   ├── lib/               # Pure functions
│   │   ├── decorators.ts  # Method decorators
│   │   └── text-generator.ts # Text transformation
│   ├── parsers/           # Multi-format parsers
│   │   ├── types.ts       # Parser interfaces
│   │   ├── base-parser.ts # Abstract base class
│   │   ├── env-parser.ts  # ENV/Shell parser
│   │   ├── json-parser.ts # JSON parser (nested)
│   │   ├── yaml-parser.ts # YAML parser (nested)
│   │   ├── properties-parser.ts # Properties parser
│   │   ├── toml-parser.ts # TOML parser
│   │   └── index.ts       # Parser registry
│   ├── utils/             # Helper utilities
│   │   ├── config.ts      # Configuration facade
│   │   ├── file.ts        # File operations
│   │   ├── pattern-matcher.ts # Pattern matching
│   │   └── validator.ts   # Input validation
│   ├── __tests__/         # Test files (mirrors src)
│   │   ├── core/
│   │   ├── lib/
│   │   ├── parsers/
│   │   └── utils/
│   ├── __mocks__/         # Test mocks
│   │   └── vscode.ts      # VS Code API mock
│   └── extension.ts       # Entry point
├── .github/
│   └── workflows/         # CI/CD pipelines
├── .vscode/               # VS Code workspace config
├── dist/                  # Compiled output
└── node_modules/          # Dependencies
```

## Module Responsibilities

### Core Module (`src/core/`)

**Purpose**: Contains the main business logic and state management

**Files**:

- `camouflage.ts`: Main class managing decorations, events, status bar

**Responsibilities**:

- Apply/remove decorations
- Handle editor events
- Manage extension state
- Status bar updates
- Command execution
- Delegate parsing to parser registry

**Dependencies**: Can import from `lib/`, `utils/`, `parsers/`, and VS Code API

### Parsers Module (`src/parsers/`)

**Purpose**: Multi-format file parsing with Strategy Pattern

**Files**:

- `types.ts`: Parser interface and type definitions
- `base-parser.ts`: Abstract base class with common functionality
- `env-parser.ts`: ENV, Shell script, .envrc parser
- `json-parser.ts`: JSON parser with nested key support
- `yaml-parser.ts`: YAML parser with nested key support
- `properties-parser.ts`: Properties, INI, conf parser
- `toml-parser.ts`: TOML parser
- `index.ts`: Parser registry and factory

**Rules**:

- ✅ Each parser extends `BaseParser`
- ✅ Parsers implement `Parser` interface
- ✅ Parsers are stateless (options via constructor)
- ✅ Registry handles parser selection by file extension
- ✅ Nested keys returned as dot-separated paths

**Dependencies**: Can only import Node.js built-ins and types

### Library Module (`src/lib/`)

**Purpose**: Pure, reusable functions with no side effects

**Files**:

- `decorators.ts`: Method decorators (@Log, @HandleErrors, etc.)
- `text-generator.ts`: Text transformation algorithms

**Rules**:

- ✅ Must be pure functions (same input → same output)
- ✅ No side effects (no file I/O, no global state)
- ✅ No VS Code API imports
- ✅ Fully unit testable

**Dependencies**: Can only import Node.js built-ins and other lib modules

### Utils Module (`src/utils/`)

**Purpose**: Helper functions and facades

**Files**:

- `config.ts`: Configuration access facade
- `file.ts`: File system operations
- `pattern-matcher.ts`: Pattern matching logic
- `validator.ts`: Input validation

**Rules**:

- ✅ Can have side effects (file I/O, API calls)
- ✅ Can import VS Code API
- ✅ Should be stateless where possible
- ✅ Each file has single responsibility

**Dependencies**: Can import from `lib/`, `parsers/`, and VS Code API, but not `core/`

### Tests Module (`src/__tests__/`)

**Purpose**: All test files

**Structure**: Mirrors `src/` directory structure

**Rules**:

- ✅ Every module must have corresponding test file
- ✅ Use `describe` blocks for grouping
- ✅ Use AAA pattern (Arrange, Act, Assert)
- ✅ Mock external dependencies
- ✅ Test file naming: `*.test.ts`

## Dependency Rules

### Allowed Dependencies

```
extension.ts → core/ → lib/ + utils/ + parsers/
                ↓
            VS Code API
```

### Forbidden Dependencies

- ❌ `lib/` MUST NOT import from `core/`, `utils/`, or `parsers/`
- ❌ `utils/` MUST NOT import from `core/`
- ❌ `parsers/` MUST NOT import from `core/` or `utils/`
- ❌ Circular dependencies between modules
- ❌ Direct file system access in `lib/`

## File Organization Rules

### File Naming

- `kebab-case.ts` for all files
- `*.test.ts` for test files
- `*.d.ts` for type declarations

### File Size

- Max 300 lines per file (excluding tests)
- If exceeding, split into smaller modules
- Exception: Generated files, type definitions

### Export Rules

- ✅ Named exports preferred over default exports
- ✅ One main export per file (with supporting types)
- ✅ Export only public API
- ✅ Use `export type` for type-only exports

### Import Order

1. Node.js built-ins (`import * as fs from 'fs'`)
2. External packages (`import * as vscode from 'vscode'`)
3. Internal modules - absolute paths from src root
4. Type imports (`import type { ... }`)

**Example**:

```typescript
import * as path from 'path';
import * as vscode from 'vscode';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
