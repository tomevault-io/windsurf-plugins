---
trigger: always_on
description: This document provides essential context for LLM agents working on the KdlSharp project. It serves as an entry point for understanding the codebase, project structure, and development workflow.
---

# Agents Instructions

This document provides essential context for LLM agents working on the KdlSharp project. It serves as an entry point for understanding the codebase, project structure, and development workflow.

## Project Overview

**KdlSharp** is a feature-rich C# library for parsing, manipulating, and serializing [KDL (KDL Document Language)](https://kdl.dev/) documents. KDL is a human-friendly, node-based document language similar to XML/HTML but with cleaner syntax, designed for configuration files and data serialization.

### Key Features
- **KDL v2.0 Support**: Full support for the current specification (with v1.0 legacy support)
- **POCO Serialization**: Maps .NET objects to/from KDL documents
- **Document Model API**: Programmatically parse and manipulate KDL documents
- **Query Language**: CSS-selector-like queries (based on KDL Query spec)
- **Schema Validation**: Basic validation against schemas (work in progress)
- **High Performance**: Efficient parsing with optimized lexer
- **Zero Dependencies**: Core library has no external dependencies
- **Cross-Platform**: .NET Standard 2.1 compatible

### Target Framework
- **Library (KdlSharp/)**: .NET Standard 2.1 (compatible with .NET 6+, Framework 4.7.2+)
- **Tests/Demo/Benchmarks**: .NET 9.0

## Project Structure

```
KdlSharp/
├── .github/                       # GitHub Actions workflows
│   └── workflows/                 # CI/CD pipeline definitions
│       ├── build.yaml             # Build and test automation
│       └── release.yaml           # Release and NuGet publish automation
│
├── KdlSharp/                      # Main library (NuGet package)
│   ├── Exceptions/                # Exception types (parse, validation, query, schema)
│   ├── Extensions/                # LINQ-like extension methods for querying
│   ├── Formatting/                # KDL formatting and writing
│   ├── Parsing/                   # Lexer, parser, and reader for KDL text
│   ├── Query/                     # KDL Query Language implementation
│   ├── Schema/                    # KDL Schema validation (basic support)
│   │   └── Rules/                 # Validation rules (string, number, generic, structural)
│   ├── Serialization/             # POCO serialization/deserialization
│   │   ├── Converters/            # Type converters for serialization
│   │   ├── Metadata/              # Attributes and metadata for mapping
│   │   └── Reflection/            # Reflection-based type mapping
│   ├── Settings/                  # Parser and formatter settings
│   ├── Utilities/                 # Number parsing, string escaping
│   ├── Values/                    # KDL value types (string, number, bool, null)
│   ├── KdlAnnotation.cs           # Type annotations for values
│   ├── KdlDocument.cs             # Root document type
│   ├── KdlNode.cs                 # Node with name, args, props, children
│   ├── KdlProperty.cs             # Key-value property
│   ├── KdlValue.cs                # Base value type
│   ├── KdlVersion.cs              # Version enumeration (V1, V2)
│   ├── SourcePosition.cs          # Line/column tracking for errors
│   └── ValidationResult.cs        # Schema validation result
│
├── KdlSharp.Tests/                # Unit tests (xUnit, .NET 9)
│   ├── OfficialTests/             # Official KDL specification tests
│   │   ├── manifest.json          # Test manifest from official repo
│   │   ├── OfficialTestRunner.cs  # Runner for official tests
│   │   └── README.md              # Documentation for official tests
│   ├── QueryTests/                # Query language tests
│   ├── SchemaTests/               # Schema validation tests
│   ├── SerializerTests/           # POCO serialization tests
│   ├── ReaderTests/               # Low-level reader tests
│   ├── BasicParsingTests.cs       # Core parsing functionality
│   └── VersioningTests.cs         # v1/v2 compatibility tests
│
├── KdlSharp.Demo/                 # Working examples (.NET 9)
│   ├── Examples/                  # Example files demonstrating features
│   ├── Program.cs                 # Demo runner
│   └── README.md                  # Demo documentation
│
├── KdlSharp.Benchmarks/           # BenchmarkDotNet performance tests (.NET 9)
│   └── Program.cs                 # Benchmark runner
│
├── specs/                         # Git submodule: https://github.com/kdl-org/kdl
│   ├── SPEC.md                    # KDL v2.0 specification
│   ├── SPEC_v1.md                 # KDL v1.0 specification (legacy)
│   ├── QUERY-SPEC.md              # Query language specification
│   ├── SCHEMA-SPEC.md             # Schema language specification
│   ├── tests/                     # Official compatibility test suite
│   └── examples/                  # Example KDL documents
│
├── .editorconfig                  # Code style rules
├── .gitignore                     # Git ignore patterns
├── .gitmodules                    # Submodule configuration
├── AGENTS.md                      # This file (development guide)
├── Directory.Build.props          # Shared MSBuild configuration
├── KdlSharp.sln                   # Solution file
├── LICENSE                        # MIT License

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreyAkinshin/KdlSharp](https://github.com/AndreyAkinshin/KdlSharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
