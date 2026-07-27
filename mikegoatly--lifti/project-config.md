---
trigger: always_on
description: LIFTI is a lightweight, in-memory full text search indexing library for .NET. The project provides high-performance text indexing and querying capabilities with support for loose text, object properties, powerful query syntax, thesaurus support, and binary serialization.
---

# LIFTI - Lightweight Full Text Indexing for .NET

## Project Overview

LIFTI is a lightweight, in-memory full text search indexing library for .NET. The project provides high-performance text indexing and querying capabilities with support for loose text, object properties, powerful query syntax, thesaurus support, and binary serialization.

### Key Features

- In-memory full text indexing
- Object property indexing with fluent configuration
- Advanced query language with operators (AND, OR, NOT, NEAR, wildcards, fuzzy matching)
- Thesaurus support for synonym expansion
- Score boosting (freshness, magnitude, field-level)
- Binary serialization for index persistence
- Support for .NET 8.0 and 9.0

## Architecture

### Core Components

1. **FullTextIndex<TKey>**: Main entry point for creating and querying indexes
2. **FullTextIndexBuilder<TKey>**: Fluent builder for configuring indexes
3. **IndexNode**: Tree-based index structure using a character-based trie
4. **Query System**: Complex query parsing and execution with scoring
5. **Tokenization**: Customizable text tokenization with stemming support
6. **Serialization**: Binary serialization for index persistence

### Key Namespaces

- `Lifti`: Core index types and builders
- `Lifti.Querying`: Query parsing, execution, and scoring
- `Lifti.Tokenization`: Text tokenization, stemming, and text extraction
- `Lifti.Serialization`: Binary serialization support

## Coding Standards

### General Principles

1. **Nullable Reference Types**: Enabled project-wide. Always annotate nullability correctly.
2. **Performance First**: LIFTI is a performance-critical library. Consider memory allocations, object pooling, and algorithmic efficiency.
3. **Immutability**: Prefer immutable data structures where possible. Use mutation patterns with care.
4. **Documentation**: All public APIs must have XML documentation comments.
5. **Backwards Compatibility**: Maintain serialization compatibility across versions when possible.

### Code Style

- Follow the `.editorconfig` settings (4-space indentation, CRLF line endings)
- Use `var` for local variables
- Use expression-bodied members for properties and simple methods
- Prefer `this.` qualification for fields, properties, and methods
- Use file-scoped namespaces (C# 10+) when appropriate
- Enable and address all compiler warnings (`TreatWarningsAsErrors=True`)

### Exception Handling

**CRITICAL**: All exception messages must be defined in `ExceptionMessages.resx` resource file:

1. **Never use hard-coded exception messages** - Always define them in `src/Lifti.Core/ExceptionMessages.resx`
2. **Add the resource entry** with a descriptive name (e.g., `EndAnchorWithoutPrecedingText`)
3. **Update the Designer.cs file** - Add the corresponding property to `src/Lifti.Core/ExceptionMessages.Designer.cs`:
   ```csharp
   /// <summary>
   ///   Looks up a localized string similar to [Your message here].
   /// </summary>
   internal static string YourExceptionName {
       get {
           return ResourceManager.GetString("YourExceptionName", resourceCulture);
       }
   }
   ```
4. **Use the resource in code** - Reference it via `ExceptionMessages.YourExceptionName`
5. **XML-encode special characters** in the resx file (e.g., `&lt;` for `<`, `&gt;` for `>`)

Example:

```csharp
// ❌ WRONG - Hard-coded message
throw new QueryParserException("End anchor (>>) must be preceded by text");

// ✅ CORRECT - Using resource
throw new QueryParserException(ExceptionMessages.EndAnchorWithoutPrecedingText);
```

### Naming Conventions

- Interfaces: `IFullTextIndex`, `IIndexNavigator`
- Builders: `FullTextIndexBuilder`, `ThesaurusBuilder`
- Options: `IndexOptions`, `QueryParserOptions`
- Results: `SearchResult`, `SearchResults`
- Factories: `IIndexScorerFactory`, `IIndexNodeFactory`

### Pattern Usage

1. **Builder Pattern**: Used extensively for configuration (see `FullTextIndexBuilder`, `ThesaurusBuilder`)
2. **Factory Pattern**: For creating complex objects (see `IIndexNodeFactory`, `IIndexScorerFactory`)
3. **Pool Pattern**: For reusable objects (see `IndexNavigatorPool`, `SharedPool<T>`)
4. **Mutation Pattern**: For safe concurrent modifications (see `IndexMutation`, `IndexNodeMutation`)
5. **Snapshot Pattern**: For consistent read views during writes (see `IIndexSnapshot`)

## Testing Standards

### Test Framework

- **xUnit** for all unit tests
- **FluentAssertions** for expressive assertions
- Target frameworks: net8.0, net9.0

### Test Organization

1. Test classes should match source class names with `Tests` suffix
2. Inherit from base classes when common setup is needed (e.g., `QueryTestBase`, `MutationTestBase`)
3. Use descriptive test method names that explain the scenario
4. Group related tests using nested classes or theory data
5. Where possible, create a `sut` (system under test) field, initialized in the constructor that can be reused across test methods

### Test Patterns

```csharp
public class FeatureTests
{
    private readonly SystemUnderTest sut;

    public FeatureTests()
    {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikegoatly/lifti](https://github.com/mikegoatly/lifti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
