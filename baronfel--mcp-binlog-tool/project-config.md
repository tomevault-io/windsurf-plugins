---
trigger: always_on
description: This document provides guidance and expectations for AI assistants (Claude, Copilot, etc.) working on the mcp-binlog-tool project. It captures the established patterns, standards, and architectural decisions from the project's development.
---

# Claude Development Guide for mcp-binlog-tool

This document provides guidance and expectations for AI assistants (Claude, Copilot, etc.) working on the mcp-binlog-tool project. It captures the established patterns, standards, and architectural decisions from the project's development.

## Project Overview

This is a Model Context Protocol (MCP) server for analyzing MSBuild binary log files (.binlog). The server provides comprehensive tools for build performance analysis, diagnostic extraction, search capabilities, and build introspection.

## Architecture & Organization

### Feature-Based Structure
- Features are organized in separate directories under `binlog.mcp/Features/`
- Each feature contains:
  - `Models.cs` - Data models and utility methods
  - `*Extensions.cs` - Service registration and DI setup
  - `*Tool.cs` - MCP tool implementations
- Features communicate through shared infrastructure in `Infrastructure/`

### Current Feature Areas
- **AnalyzerAnalysis** - Roslyn analyzer performance analysis
- **BinlogLoading** - Core binlog loading and caching
- **BuildAnalysis** - Build-level analysis and prompts
- **DiagnosticAnalysis** - Error and warning extraction
- **EvaluationAnalysis** - Project evaluation analysis
- **ProjectAnalysis** - Project-level performance analysis
- **SearchAnalysis** - Powerful freetext search capabilities
- **TargetAnalysis** - Target execution analysis
- **TaskAnalysis** - Task-level analysis and timing
- **TimelineAnalysis** - Build timeline and parallelization analysis

## Coding Standards

### Data Models
- **Use record structs** for data transfer objects and API models
- **Use enums** for classifications (e.g., `DiagnosticSeverity.Error`)
- **Use nullable reference types** consistently
- **Prefer ID-based references** over string names to reduce data duplication
- **Include comprehensive Description attributes** for all public properties

### Tool Implementation
- **McpServerTool attributes** must include:
  - `Name` - kebab-case tool name
  - `Title` - Human-readable title
  - `Idempotent = true` - All our tools are read-only
  - `UseStructuredContent = true` - For performance
  - `ReadOnly = true` - All tools are read-only
- **Parameter descriptions** must be comprehensive and include:
  - Data types and constraints
  - Default values where applicable
  - Optional parameter indicators
  - Usage examples for complex parameters
- **Support filtering and pagination** where appropriate:
  - `maxResults` parameters for large result sets
  - Project/target/task ID filtering arrays
  - Boolean flags for including/excluding data
  - Optional detail levels to control response size

### Performance Patterns
- **Single-pass processing** - Process data in one iteration when possible
- **Caching** - Use static caches for expensive computations (see ProjectBuildTimeCache)
- **Lazy loading** - Only load data when needed
- **Efficient data structures** - Use dictionaries and arrays, avoid nested loops
- **Optional detail exclusion** - Provide flags to exclude expensive-to-serialize data

### MSBuild Integration
- **Use proper node types** from MSBuildStructuredLog:
  - `AbstractDiagnostic` for diagnostics (not message content inference)
  - `Project`, `Target`, `Task` for structural nodes
  - `Build.StringTable.Instances` for search operations
- **Leverage existing APIs** instead of manual parsing
- **Handle nullable properties** from MSBuild node types properly

## JSON Serialization

### Context Management
- **Register all new types** in `BinlogJsonContext.cs`
- **Use JsonSerializable attributes** for source generation
- **Follow naming conventions** for property serialization
- **Handle nullable types** appropriately in JSON contracts

## Documentation Standards

### Tool Documentation (PACKAGE_README.md)
- **Organize by feature area** with clear section headers
- **Alphabetical ordering** within each feature area
- **Comprehensive parameter documentation** including:
  - Parameter name, type, and description
  - Default values and optional indicators
  - Array types and filtering capabilities
- **Return type documentation** with structure details
- **Include use cases and examples** for complex features
- **Cross-reference related tools** where appropriate

### Query Language Documentation
- **Provide comprehensive syntax documentation** (see `search_binlog` tool)
- **Include practical examples** with real-world scenarios
- **Document shortcuts and advanced features**
- **Explain hierarchical and filtering capabilities**

### Release Documentation
- **CHANGELOG.md updates** must include:
  - Version number and release date
  - Detailed feature descriptions under "Added" section
  - Performance improvements under "Changed" section
  - Breaking changes clearly marked
- **README.md updates** for major feature additions
- **Version synchronization** across all files (server.json, CHANGELOG.md)

## API Design Principles

### Consistency
- **Consistent parameter naming** across similar tools
- **Consistent return value structures** for similar operations
- **Consistent error handling** patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baronfel/mcp-binlog-tool](https://github.com/baronfel/mcp-binlog-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
