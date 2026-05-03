---
trigger: always_on
description: You are an expert code reviewer and software engineer assisting with the `CommentSense` project. Your goal is to help write, review, and maintain high-quality, secure, and idiomatic C# code for this Roslyn analyzer.
---

# GitHub Copilot Instructions for CommentSense

You are an expert code reviewer and software engineer assisting with the `CommentSense` project. Your goal is to help write, review, and maintain high-quality, secure, and idiomatic C# code for this Roslyn analyzer.

## 1. High Level Details
*   **Project:** A Roslyn-based diagnostic analyzer for C# designed to enforce consistent and meaningful API documentation (XML comments).
*   **Type:** Roslyn Analyzer (NuGet package).
*   **Frameworks:**
    *   **Analyzer:** .NET Standard 2.0.
    *   **Tests:** .NET 8.0, .NET 9.0, .NET 10.0.
*   **Package Management:** Central Package Management via `Directory.Packages.props`.
*   **Key Libraries:** `Microsoft.CodeAnalysis`, `NUnit`, `Microsoft.CodeAnalysis.CSharp.Analyzer.Testing`.

## 2. Build and Validate
Always verify changes using these commands.
*   **Build:** `dotnet build CommentSense.slnx --configuration Release`
*   **Test:** `dotnet test CommentSense.slnx --configuration Release --no-build --settings .runsettings --results-directory ./coverage`
*   **Lint/Style:** Use `.editorconfig` rules. Build with `/warnaserror` when possible.

**Build Status:** Trust the user or actual build/test logs provided in the chat context regarding compilation status. Do not claim code will not compile based on internal static analysis if the context or the user indicates otherwise. Avoid providing unsolicited "fix" suggestions for non-existent build errors.

## 3. Project Layout
*   **`src/CommentSense.Analyzers/`**: The main analyzer project. Contains the diagnostic analyzer implementations, rule definitions, and specialized analyzer logic.
    *   **`CommentSenseSuppressor.cs`**: Automatically silences overlapping built-in compiler diagnostics.
*   **`src/CommentSense.CodeFixes/`**: The code fix provider project. Contains implementations for automatically fixing diagnostics.
*   **`src/CommentSense.Core/`**: Shared core logic. Contains common utilities for accessibility checks and XML documentation parsing.
*   **`tests/CommentSense.Analyzers.Tests/`**: Integration tests for the diagnostic rules.
*   **`tests/CommentSense.CodeFixes.Tests/`**: Integration tests for the code fix providers.
*   **`tests/CommentSense.Core.Tests/`**: Unit tests for the core utilities.
*   **`tests/CommentSense.TestHelpers/`**: Shared testing infrastructure, including `CommentSenseAnalyzerTestBase<T>`, `CommentSenseCodeFixTestBase<T1, T2>`, and `RoslynTestUtils`.
*   **`artifacts/`**: Unified build output location (bin, obj, package) configured via `Directory.Build.props`.
*   **`.github/workflows/`**: CI/CD pipelines for building, testing, linting, and publishing.

## 4. Coding & Architectural Standards
*   **Roslyn Best Practices:**
    *   Use `ImmutableArray` for collections.
    *   Register actions in `Initialize` (e.g., `context.RegisterSymbolAction`).
    *   Avoid state in the analyzer class itself (use the `AnalysisContext`).
    *   Use `DiagnosticSuppressor` to handle overlapping compiler diagnostics.
*   **Testing:**
    *   Use `Microsoft.CodeAnalysis.CSharp.Analyzer.Testing` and `Microsoft.CodeAnalysis.CSharp.CodeFix.Testing` with `NUnit`.
    *   Inherit from `CommentSenseAnalyzerTestBase<T>` for analyzer tests and `CommentSenseCodeFixTestBase<TAnalyzer, TCodeFix>` for code fix tests.
    *   Tests should verify both positive (diagnostic reported) and negative (no diagnostic) cases.
    *   Use `[| ... |]` markup in test strings to indicate expected diagnostic locations.
*   **XML Parsing:**
    *   Use `DocumentationXmlExtensions` for parsing XML comments to ensure resilience against malformed XML.
    *   Always use `DocumentationSyntaxExtensions.GetNameAttribute()` to extract name attributes from XML nodes, as it handles both `XmlNameAttributeSyntax` and `XmlTextAttributeSyntax`.
    *   Handle `inheritdoc` and `include` tags gracefully (currently treated as "valid" without deep validation).
    *   **Scan Strategy**: Differentiate between documentation *presence/quality* and *redundancy/strays*:
        *   **Recursive Scan**: Use `recursive: true` or `topLevelOnly: false` to identify ALL occurrences of a tag. Flag nested or extra tags as **Stray** or **Duplicate**.
        *   **Top-Level Only Scan**: Use `recursive: false` or `topLevelOnly: true` to determine if a symbol is properly documented. Nested tags do NOT count towards fulfilling documentation requirements and should NOT undergo quality analysis (they are already flagged as stray).
*   **Deduplication:**
    *   Use `SymbolExtensions.GetParameters()` and `SymbolExtensions.GetTypeParameters()` for extracting parameter names from symbols. Do not re-implement this logic in analyzers.
    *   Use `node.GetAssociatedSymbol(semanticModel)` to find the symbol associated with an XML documentation node or member declaration (it correctly handles fields).
    *   Use `symbol.GetTargetElementsWithLocations(xml, tagName)` to iterate over XML elements and their source locations simultaneously.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Thomas-Shephard/comment-sense](https://github.com/Thomas-Shephard/comment-sense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
