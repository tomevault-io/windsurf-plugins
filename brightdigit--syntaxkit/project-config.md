---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SyntaxKit is a Swift package that provides a declarative DSL for generating Swift code using result builders. Built on SwiftSyntax, it allows programmatic creation of Swift code structures (structs, enums, classes, functions) in a type-safe manner.

## Essential Commands

### Build & Test
```bash
# Build the package
swift build

# Run all tests
swift test

# Run specific test
swift test --filter TestName

# Run tests with coverage
swift test --enable-code-coverage
```

### Code Quality
```bash
# Run comprehensive linting (SwiftFormat, SwiftLint, Periphery)
./Scripts/lint.sh

# Format code only (skip other checks)
LINT_MODE=NONE ./Scripts/lint.sh
```

### Documentation
```bash
# Generate DocC documentation
swift package generate-documentation
```

## Architecture

### Core Design Patterns
- **Result Builders**: Declarative DSL using `@resultBuilder` for Swift code generation
- **Protocol-Oriented**: `CodeBlock` protocol as foundation for all syntax elements
- **SwiftSyntax Integration**: All components generate native SwiftSyntax AST nodes

### Key Protocols
- `CodeBlock` - Core protocol for all syntax elements
- `PatternConvertible` - For pattern matching constructs  
- `TypeRepresentable` - For type system integration

### Source Organization
```
Sources/SyntaxKit/
├── Core/           # Fundamental protocols and builders
├── Declarations/   # Type declarations (Class, Struct, Enum, etc.)
├── Expressions/    # Swift expressions and operators
├── Functions/      # Function definitions and method calls
├── Variables/      # Variable and property declarations
├── ControlFlow/    # Control flow constructs (Switch, If, For)
├── Collections/    # Array, dictionary helpers
├── Parameters/     # Function parameter handling
├── Patterns/       # Pattern matching constructs
├── Utilities/      # Helper functions and extensions
└── ErrorHandling/  # Error handling constructs
```

## Development Workflow

### Adding New Syntax Elements
1. Create source file in appropriate subdirectory
2. Implement `CodeBlock` protocol
3. Add corresponding unit tests in `Tests/SyntaxKitTests/Unit/`
4. Run `./Scripts/lint.sh` to ensure code quality
5. Run `swift test` to verify functionality

### Package Dependencies
- **SwiftSyntax** (601.0.1+) - Apple's Swift syntax parser
- **SwiftOperators** - Operator handling
- **SwiftParser** - Swift code parsing
- **SwiftDocC Plugin** (1.4.0+) - Documentation generation

### Quality Tools
- **SwiftFormat** (602.0.0) - Code formatting
- **SwiftLint** (0.63.2) - Static analysis (90+ opt-in rules)
- **Periphery** (3.7.2) - Unused code detection

## Project Structure

### Products
1. **SyntaxKit Library** - Main DSL library
2. **skit Executable** - Command-line tool for parsing Swift code to JSON

### Platform Support
- Swift 6.0+ required
- Xcode 16.0+ for development

### Testing
- Uses modern Swift Testing framework (`@Test` syntax)
- Tests organized by component in `Tests/SyntaxKitTests/Unit/`
- Integration tests in `Tests/SyntaxKitTests/Integration/`
- Comprehensive CI/CD with GitHub Actions

## SwiftSyntax Reference

> **Full Documentation**: [SwiftSyntax 601.0.1 Documentation](https://swiftpackageindex.com/swiftlang/swift-syntax/601.0.1/documentation/swiftsyntax)  
> **Local Reference**: [docs/SwiftSyntax-LLM.md](Docs/SwiftSyntax-LLM.md) - Complete SwiftSyntax API reference (590KB)

### Core Concepts
SwiftSyntax is Apple's source-accurate tree representation of Swift source code, enabling parsing, inspection, generation, and transformation of Swift code programmatically.

### Key Types & Protocols

#### Syntax Foundation
- `Syntax` - Base protocol for all syntax nodes
- `SyntaxProtocol` - Protocol all syntax nodes conform to
- `SyntaxCollection` - Collection of syntax nodes
- `SyntaxChildren` - Collection of child syntax nodes

#### Tokens & Trivia
- `TokenSyntax` - Single token representation
- `TokenKind` - Enumerates Swift language token types
- `Trivia` - Whitespace, comments, and other non-semantic content
- `TriviaPiece` - Individual trivia element
- `SourcePresence` - Indicates if node was found in source

#### Major Syntax Categories

**Declarations (`DeclSyntax`)**
- `ClassDeclSyntax` - Class declarations
- `StructDeclSyntax` - Struct declarations  
- `EnumDeclSyntax` - Enum declarations
- `ProtocolDeclSyntax` - Protocol declarations
- `FunctionDeclSyntax` - Function declarations
- `VariableDeclSyntax` - Variable declarations
- `ImportDeclSyntax` - Import statements
- `ExtensionDeclSyntax` - Extension declarations
- `TypeAliasDeclSyntax` - Type alias declarations
- `AssociatedTypeDeclSyntax` - Associated type declarations
- `OperatorDeclSyntax` - Operator declarations
- `PrecedenceGroupDeclSyntax` - Precedence group declarations
- `MacroDeclSyntax` - Macro declarations
- `MacroExpansionDeclSyntax` - Macro expansion declarations

**Expressions (`ExprSyntax`)**
- `FunctionCallExprSyntax` - Function calls
- `MemberAccessExprSyntax` - Member access (dot notation)
- `SubscriptCallExprSyntax` - Subscript calls
- `BinaryOperatorExprSyntax` - Binary operators
- `PrefixOperatorExprSyntax` - Prefix operators

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brightdigit/SyntaxKit](https://github.com/brightdigit/SyntaxKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
