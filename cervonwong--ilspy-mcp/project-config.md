---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**ILSpy MCP — Feature Parity**

An MCP (Model Context Protocol) server that exposes ILSpy's .NET decompilation and static analysis capabilities as tools for AI assistants. Currently has 8 tools covering basic type inspection (~25-30% of ILSpy GUI functionality). This milestone extends it to full reverse engineering feature parity — cross-reference tracing, IL output, assembly metadata, string search, resource extraction, and bulk decompilation.

**Core Value:** AI assistants can perform complete .NET static analysis workflows — not just read code, but trace execution, find usages, search strings, and navigate across types and assemblies.

### Constraints

- **Tech stack**: C#, .NET, ICSharpCode.Decompiler, System.Reflection.Metadata, MCP SDK — no new runtime dependencies
- **Architecture**: Follow existing layered pattern (Domain/Infrastructure/Application/Transport)
- **Testing**: Critical-path tests for P0 features and all bug fixes to ensure nothing breaks
- **Compatibility**: Must not break existing 8 tools during upgrades
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Current Stack (Baseline)
| Technology | Current Version | Target Version | Breaking? |
|------------|----------------|----------------|-----------|
| ICSharpCode.Decompiler | 9.1.0.7988 | 10.0.0.8330 | YES |
| ModelContextProtocol | 0.4.0-preview.3 | 1.2.0 | YES |
| Microsoft.Extensions.Hosting | 8.0.0 | 10.0.0 | Minor |
| .NET Runtime | net9.0 | net9.0 (keep) | NO |
| xUnit | 2.9.2 | 2.9.3 (keep v2) | NO |
| FluentAssertions | 8.8.0 | 8.9.0 | NO |
## Recommended Stack
### Core Decompilation
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| ICSharpCode.Decompiler | 10.0.0.8330 | C# decompilation, IL disassembly, type system | Just released (2026-04-06). Still targets netstandard2.0, compatible with net9.0. Adds `IDecompilerTypeSystem` interface on `CSharpDecompiler` constructor for better testability, new `ExpandParamsArguments` and `AlwaysMoveInitializer` settings. |
| System.Reflection.Metadata | 9.0.0+ (transitive) | IL scanning, metadata reading, cross-reference analysis | Comes transitively via ICSharpCode.Decompiler. Provides `MetadataReader`, `MethodBodyBlock.GetILReader()`, `ILOpCode` enum, `BlobReader` for raw IL bytecode scanning. This is the engine for string search (`ldstr`), constant search (`ldc.*`), and cross-reference tracing. |
### MCP Server
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| ModelContextProtocol | 1.2.0 | MCP server framework with tool registration | Stable release (GA since Feb 2025). The `[McpServerToolType]` / `[McpServerTool]` attribute pattern and `WithToolsFromAssembly()` builder survive from 0.4.0-preview -- the core pattern used in this project is preserved. |
| ModelContextProtocol.Core | 1.2.0 (transitive) | Core protocol types | Pulled in automatically by ModelContextProtocol package. |
| Microsoft.Extensions.Hosting | 10.0.0 | DI, configuration, lifecycle management | Required by MCP SDK 1.2.0 (depends on Microsoft.Extensions.Hosting.Abstractions >= 10.0.5). Must upgrade from 8.0.0. |
| Microsoft.Extensions.Logging.Console | 10.0.0 | Stderr logging for MCP transport | Keep aligned with Hosting version. |
### Testing
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| xUnit | 2.9.3 | Test framework | Stay on v2. xUnit v3 (3.2.2) is stable but requires project restructuring (test projects become executables, package renames to `xunit.v3`, attribute API changes). Not worth the migration churn during a feature milestone. Upgrade separately later. |
| FluentAssertions | 8.9.0 | Assertion library | Minor patch update. No breaking changes from 8.8.0. |
| Microsoft.NET.Test.Sdk | 17.12.0 | Test runner infrastructure | Keep current. Compatible with xUnit 2.x on net9.0. |
| coverlet.collector | 6.0.2 | Code coverage | Keep current. Works with xUnit 2.x. |
### Infrastructure (No New Dependencies)
| Technology | Version | Purpose | Why NOT add new packages |
|------------|---------|---------|--------------------------|
| System.Reflection.Metadata | (transitive) | IL scanning for cross-refs, string search, constant search | Already a dependency of ICSharpCode.Decompiler. Provides everything needed: `MetadataReader`, `MethodBodyBlock`, `BlobReader`, `ILOpCode`. No wrapper library needed. |
| ICSharpCode.Decompiler.Disassembler | (built-in namespace) | IL/CIL output for types and methods | Part of ICSharpCode.Decompiler package. `ReflectionDisassembler` class with `DisassembleType()` and `DisassembleMethod()`. No additional package. |
## Breaking Changes: Migration Guide
### ICSharpCode.Decompiler 9.1 to 10.0
| Change | Impact on This Project | Action Required |
|--------|----------------------|-----------------|
| Removed `ITypeReference` and implementations | LOW -- Project uses `FullTypeName` and `ITypeDefinition`, not `ITypeReference` | Verify no transitive usage in type hierarchy code |
| `ResolvedUsingScope` renamed to `UsingScope` | NONE -- Project does not use using scope APIs | No action |
| Removed `UnresolvedUsingScope` | NONE -- Not used | No action |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cervonwong/ILSpy-MCP](https://github.com/cervonwong/ILSpy-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
