---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AppRefiner is a Windows Forms application that enhances PeopleSoft's Application Designer with modern development features including code folding, linting, syntax highlighting, and refactoring tools. It integrates seamlessly with Application Designer through Win32 API hooks and provides a comprehensive plugin architecture for extensibility.

## Build and Development Commands

### Building the Project
```powershell
# Framework-dependent build (requires .NET 8 runtime on target machine)
.\build.ps1

# Self-contained build (includes .NET runtime)
.\build.ps1 -SelfContained
```

### Prerequisites
- Windows with Visual Studio 2022 (C++ development tools)
- .NET 8 SDK
- PowerShell 5.1+

### Development Workflow
```powershell
# Restore dependencies
dotnet restore

# Build main project
dotnet build AppRefiner/AppRefiner.csproj

# Build C++ hook DLL (requires Visual Studio)
msbuild AppRefinerHook/AppRefinerHook.vcxproj /p:Configuration=Release /p:Platform=x64

# Parser is self-hosted in PeopleCodeParser.SelfHosted project
# No code generation required - pure C# implementation
```

**IMPORTANT FOR CLAUDE CODE USERS**: Do not attempt to build the project directly when working in WSL environments. AppRefiner is a Windows-specific application that relies on Windows Forms, Win32 APIs, and Visual Studio C++ build tools. Building should only be done on Windows with proper development tools installed.

### Project Structure
- **AppRefiner/**: Main Windows Forms application (.NET 8)
- **PeopleCodeParser.SelfHosted/**: Self-hosted C# recursive descent parser (.NET 8)
- **AppRefinerHook/**: Win32 API hook DLL (C++)
- **PluginSample/**: Example plugin implementation

## Core Architecture

### Service Layer Architecture
The application uses a service-oriented architecture with dependency injection:

- **AstService**: Manages AST parsing and caching with dependency resolution
- **SettingsService**: Centralized configuration management with JSON serialization
- **KeyboardShortcutService**: Global hotkey management
- **WinEventService**: Windows event handling and Application Designer integration

Services are constructor-injected into managers and use concurrent collections for thread safety.

### Plugin/Extension System
The core extensibility model is based on abstract base classes with the visitor pattern:

**Base Classes:**
- `BaseLintRule`: For code analysis and issue detection
- `BaseStyler`: For visual indicators and highlighting
- `ScopedRefactor`: For code transformations (unified base class)
- `BaseTooltipProvider`: For contextual information display
- `BaseCommand`: For custom commands in the command palette with keyboard shortcuts (built-in or plugin)
- `BaseLanguageExtension`: For adding properties/methods to PeopleCode types via code transformations

**Scoped Variants:**
- `ScopedLintRule<T>`: Adds variable/scope tracking
- `ScopedStyler<T>`: Scope-aware styling
- All refactors now use `ScopedRefactor` (unified base class with automatic scope tracking)

**Key Patterns:**
- Reflection-based discovery from assemblies and plugins
- Template Method pattern for extensibility
- Manager classes handle registration and lifecycle
- Priority-based execution ordering

### Database Integration
Uses Repository pattern with interface abstraction:
- `IDataManager`: Defines data access contract
- `OraclePeopleSoftDataManager`: PeopleSoft-specific implementation
- `DataManagerRequirement`: Dependency declaration system
- Connection pooling and caching for performance

### Parser and AST Architecture

AppRefiner uses a self-hosted recursive descent parser written entirely in C# with no external dependencies:

**Parser Components:**
- **PeopleCodeLexer**: Tokenizes source text with UTF-8 byte index tracking for Scintilla integration
  - Case-insensitive keyword recognition
  - System variable identification (%, &)
  - Comment and whitespace handling (trivia)
  - Comprehensive error reporting

- **PeopleCodeParser**: Recursive descent parser with advanced error recovery
  - Synchronization tokens for resilient parsing during live editing
  - Directive preprocessing support (#If, #Else, #End-If)
  - Produces strongly-typed AST nodes
  - Detailed parse error reporting

**AST Node Hierarchy:**
- **AstNode**: Base class for all AST nodes
  - Token-based source location tracking (`FirstToken`, `LastToken`, `SourceSpan`)
  - Parent-child relationships built-in
  - Visitor pattern support (`Accept(IAstVisitor)`)
  - Attributes dictionary for semantic analysis (types, errors, etc.)
  - Helper methods: `FindAncestor<T>()`, `FindDescendants<T>()`, `GetRoot()`

**Core Node Types** (in `PeopleCodeParser.SelfHosted.Nodes` namespace):
- **Program Structure**: `ProgramNode`, `AppClassNode`, `InterfaceNode`, `ImportNode`
- **Declarations**: `MethodNode`, `FunctionNode`, `PropertyNode`, `ProgramVariableNode`, `ConstantNode`
- **Statements**: `IfStatementNode`, `ForStatementNode`, `WhileStatementNode`, `TryStatementNode`, `BlockNode`, etc.
- **Expressions**: `BinaryOperationNode`, `FunctionCallNode`, `IdentifierNode`, `LiteralNode`, `AssignmentNode`, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gideon-Taylor/AppRefiner](https://github.com/Gideon-Taylor/AppRefiner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
