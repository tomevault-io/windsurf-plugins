---
trigger: always_on
description: The codebase follows a **vertical architecture** where each feature owns its full stack:
---

# VS Code QML Extension - AI Coding Guide

## Architecture: Vertical Slices Pattern

The codebase follows a **vertical architecture** where each feature owns its full stack:

```
src/
├── symbols/           # AST navigation and utilities (used by ALL providers)
│   └── ast/           # Tree-sitter AST utilities
│       ├── navigation.ts  # Generic tree-sitter utilities
│       ├── qml.ts     # QML-specific AST patterns
│       └── index.ts   # Unified export (import * as ast from 'symbols/ast')
├── models/            # Domain types (SymbolInfo, etc.)
├── services/          # Business logic (SymbolResolver - used by multiple providers)
├── providers/         # Language feature implementations (organized by feature)
│   ├── definition/    # Go-to-definition
│   ├── references/    # Find references + document highlight
│   ├── symbols/       # Document outline
│   ├── selection/     # Smart selection expansion
│   ├── semanticTokens/ # Syntax highlighting
│   └── qmldir/        # Support for qmldir module definition files
├── indexer/           # Workspace indexing (2 vertical slices)
│   ├── qml-files/     # QML file parsing & caching
│   ├── modules/       # Module resolution (qmldir parsing, Qt builtins)
│   └── shared/        # Shared indexer utilities (file watching, hashing)
└── parser/            # tree-sitter parser singleton
```

**Critical Pattern**: Each provider folder is self-contained. Adding a new feature = creating a new provider folder, not modifying existing ones.

## Core Concepts

### 1. Tree-Sitter AST Navigation

All features work by traversing the tree-sitter AST. **Never** use regex for QML parsing.

**Common node types**:
- `ui_object_definition` - QML object (e.g., `Rectangle { ... }`)
- `ui_property` - Property declaration (`property string name`)
- `ui_binding` - Property assignment (`width: 100` or `id: myRect`)
- `ui_signal` - Signal declaration
- `ui_import` - Import statement
- `ui_inline_component` - Inline component definition
- `identifier`, `property_identifier`, `type_identifier` - Name tokens

**Key utilities in `symbols/ast/navigation.ts`**:
- `nodeToRange()` - Convert tree-sitter node to VS Code Range (used everywhere)
- `getNodeAtPosition()` - Find relevant node at cursor position
- `findNode()` - Recursive search with predicate
- `traverseParents()` - Walk up the AST
- `getField()` / `getFieldText()` - Access named children (e.g., `type_name`, `name`)
- `hasAncestorChain()` - Check if node's parent chain matches a type pattern (e.g., `['nested_identifier', 'ui_object_definition']`)

**QML-specific patterns in `symbols/ast/qml.ts`**:

*AST Navigation:*
- `isIdentifierNode()` - Check if node is any identifier type
- `findParentQmlObject()` - Walk up skipping property groups
- `findContainingImport()` - Get ui_import ancestor if any
- `findImportAlias()` - Find import alias declaration
- `getLeftmostIdentifier()` - Get first identifier in nested_identifier chain (e.g., "QQC" from "QQC.Button")
- `isFirstPartOfNestedIdentifier()` - Check if node is the qualifier part of a qualified name
- `isLastPartOfNestedIdentifier()` - Check if node is the component part of a qualified name

*Semantic Parsers:*
- `parseImport()` → `ImportInfo` - Parse import statement into structured data (source, version, alias, positions)
- `parseQualifiedType()` → `QualifiedTypeInfo` - Parse qualified type from AST node (handles "QQC.Button")
- `parseQualifiedTypeName()` - Parse qualified type from string (text-only, no AST nodes)
- `isSignalHandler()` - Check if name follows signal handler pattern (onXxx)

**Import pattern**: All providers use `import * as ast from '../../symbols/ast'`
- Generic utilities: `ast.nodeToRange()`, `ast.getField()`, `ast.hasAncestorChain()`, etc.
- QML-specific: `ast.qml.isIdentifierNode()`, `ast.qml.parseImport()`, `ast.qml.isSignalHandler()`, etc.

**CRITICAL**: Node identity comparison fails! When comparing nodes (e.g., `parent.childForFieldName('type_name') === node`), the comparison will always be `false` even if they represent the same AST position. Instead, **always compare by position**:
```typescript
// ❌ WRONG - will always be false
if (parent.childForFieldName('type_name') === node) { ... }

// ✅ CORRECT - compare positions
const typeName = parent.childForFieldName('type_name');
if (typeName && node.startIndex === typeName.startIndex && node.endIndex === typeName.endIndex) { ... }
```
This applies to ALL node comparisons: `childForFieldName()`, `firstNamedChild`, `lastNamedChild`, etc.

### 2. QML-Specific AST Utilities

All QML-specific AST patterns are in `symbols/ast/qml.ts`:

- **Parent keyword resolution**: `findParentQmlObject()` - walks up skipping property groups (anchors, font)
- **Property groups**: Objects starting with lowercase (e.g., `anchors {}`) aren't real QML components
- **Import aliases**: `import QtQuick 2.15 as QQ` - alias detection via `findImportAlias()`
- **Import detection**: `findContainingImport()` - efficiently checks if node is in import statement

### 3. Indexing Architecture

**Two independent vertical slices**:

1. **QML File Indexer** (`indexer/qml-files/`):
   - Extracts: imports, exports (root component + inline components), symbols, dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LordTermor/QMLSense](https://github.com/LordTermor/QMLSense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
