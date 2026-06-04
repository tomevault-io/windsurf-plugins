---
trigger: always_on
description: AST Builder is a C# .NET 10.0 solution that provides Abstract Syntax Tree (AST) construction capabilities for the Fifth programming language. It includes an ANTLR-based parser, code generation for AST builders and visitors, and a compiler with various language transformations.
---

# AST Builder for Fifth Language

AST Builder is a C# .NET 10.0 solution that provides Abstract Syntax Tree (AST) construction capabilities for the Fifth programming language. It includes an ANTLR-based parser, code generation for AST builders and visitors, and a compiler with various language transformations.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap, Build, and Test
```bash
# Prerequisites: .NET 10.0 SDK and Java 17+ are required and available
# Verify prerequisites
dotnet --version  # Should show 10.0.x (global.json uses 10.0.100)
java -version     # Should show Java 17+ for ANTLR

# Initial setup and build (run these commands in sequence)
dotnet restore fifthlang.sln                      # Takes ~70 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
dotnet build fifthlang.sln                        # Takes ~60 seconds. NEVER CANCEL. Set timeout to 120+ seconds.

# Optional: Use just
just build-all                                     # Takes ~25 seconds. NEVER CANCEL. Set timeout to 60+ seconds.

# Run tests (default: full suite for regressions)
dotnet test fifthlang.sln                           # Default for regression checks. NEVER CANCEL. Set timeout to 5+ minutes.
# Optional fast smoke (subset)
dotnet test test/ast-tests/ast_tests.csproj        # Quick subset when iterating locally.

# Run AST code generator separately
# Prefer just for quick tasks
just run-generator                                 # Takes ~5 seconds.
# OR
dotnet run --project src/ast_generator/ast_generator.csproj -- --folder src/ast-generated
```

**CRITICAL BUILD NOTES:**
- **NEVER CANCEL** any build operations - they can take up to 2 minutes
- ANTLR grammar compilation happens automatically during parser project build
- AST code generation runs automatically before compilation via MSBuild targets

### Project Structure
```
src/
├── ast-model/          # Core AST model definitions and type system
├── ast-generated/      # Auto-generated AST builders and visitors  
├── ast_generator/      # Code generator that creates AST infrastructure
├── code_generator/     # IL code generator and emission pipeline
├── parser/             # ANTLR-based parser with split grammar
├── compiler/           # Main compiler with language transformations
└── fifthlang.system/   # Built-in system functions

test/
└── ast-tests/          # TUnit tests with .5th code samples
└── runtime-integration-tests/          # TUnit tests for end-to-end verification
```

## Validation

Never under any circumstances mask a failing test with a catch-all try-catch block. It is better to transparently allow tests to fail to properly reflect the state of the code base.

### Always Validate Changes
After making any changes to the codebase:

1. **Build validation:**
   ```bash
      dotnet build fifthlang.sln  # NEVER CANCEL - wait up to 2 minutes
   ```

2. **Test validation (full suite for regressions):**
   ```bash
   dotnet test fifthlang.sln  # Default regression gate – runs all tests
   ```

   Optional fast smoke while iterating locally:
   ```bash
   dotnet test test/ast-tests/ast_tests.csproj  # Quick subset; follow with full suite before commit/PR
   ```

3. **Manual AST functionality test:**
   Create a simple test to verify AST builders work:
   ```csharp
   using ast;
   using ast_generated;
   
   var intLiteral = new Int32LiteralExp { Value = 42 };
   var builder = new Int32LiteralExpBuilder();
   var result = builder.Build();
   // Should complete without errors
   ```

### Expected Build Warnings
The following warnings are normal and can be ignored:
- ANTLR warning: "rule expression contains an assoc terminal option in an unrecognized location"
- Various C# nullable reference warnings throughout the codebase
- Switch expression exhaustiveness warnings in parser

## Common Tasks

### Code Generation
The AST generator is central to this project:
```bash
# Regenerate AST builders and visitors
dotnet run --project src/ast_generator/ast_generator.csproj -- --folder src/ast-generated

# The generator reads from src/ast-model/AstMetamodel.cs and generates:
# - builders.generated.cs          (Builder pattern classes)
# - visitors.generated.cs          (Visitor pattern classes)
# - rewriter.generated.cs          (Rewriter pattern for lowering - NEW)
# - il.builders.generated.cs       (IL-specific builders)
# - il.rewriter.generated.cs       (IL rewriter pattern - NEW)
# - typeinference.generated.cs     (Type inference support)
```

### Choosing the Right Visitor/Rewriter Pattern

When implementing AST transformations, choose the appropriate pattern based on your needs:

**Use `BaseAstVisitor`** (read-only) when:
- ✅ Analyzing the AST without modifications (e.g., symbol table building, diagnostics)
- ✅ Collecting information or metrics
- ✅ Validation passes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fifthlang/fifthlang](https://github.com/fifthlang/fifthlang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
