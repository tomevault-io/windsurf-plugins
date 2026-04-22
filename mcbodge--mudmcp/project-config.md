---
trigger: always_on
description: Mud MCP is an MCP (Model Context Protocol) server that provides AI assistants with access to MudBlazor component documentation. It clones the MudBlazor repository, parses source files using Roslyn, and exposes an indexed API via MCP tools.
---

# Mud MCP - AI Coding Agent Instructions

## Project Overview

Mud MCP is an MCP (Model Context Protocol) server that provides AI assistants with access to MudBlazor component documentation. It clones the MudBlazor repository, parses source files using Roslyn, and exposes an indexed API via MCP tools.

**Tech Stack:** .NET 10, ASP.NET Core, Roslyn, LibGit2Sharp, Aspire 13.1, xUnit + Moq

## Architecture

```
MCP Tools (12 tools) → ComponentIndexer → Parsing Services → GitRepositoryService
                              ↓
                      In-memory index of ~85 components
```

**Key services:**
- [ComponentIndexer.cs](../src/MudBlazor.Mcp/Services/ComponentIndexer.cs) - Builds/queries the component index
- [XmlDocParser.cs](../src/MudBlazor.Mcp/Services/Parsing/XmlDocParser.cs) - Parses C# source using Roslyn
- [GitRepositoryService.cs](../src/MudBlazor.Mcp/Services/GitRepositoryService.cs) - Clones/updates MudBlazor repo
- [Tools/](../src/MudBlazor.Mcp/Tools/) - MCP tool implementations with `[McpServerTool]` attributes

## Build & Test Commands

```bash
# Build (from repo root)
dotnet build

# Run tests
dotnet test --no-build

# Run server (HTTP transport on localhost:5180)
cd src/MudBlazor.Mcp && dotnet run

# Run server (stdio transport for CLI clients)
cd src/MudBlazor.Mcp && dotnet run -- --stdio

# Run with Aspire dashboard (OpenTelemetry, health checks, service discovery)
cd src/MudBlazor.Mcp.AppHost && dotnet run
```

## Configuration

Configuration via `appsettings.json` with these key sections:

```json
{
  "MudBlazor": {
    "Repository": {
      "Url": "https://github.com/MudBlazor/MudBlazor.git",
      "Branch": "main",
      "LocalPath": "./data/mudblazor-repo"
    },
    "Cache": {
      "RefreshIntervalMinutes": 60,
      "ComponentCacheDurationMinutes": 30,
      "ExampleCacheDurationMinutes": 120
    },
    "Parsing": {
      "IncludeInternalComponents": false,
      "IncludeDeprecatedComponents": true,
      "MaxExamplesPerComponent": 20
    }
  }
}
```

Options are bound to strongly-typed classes in [Configuration/MudBlazorOptions.cs](../src/MudBlazor.Mcp/Configuration/MudBlazorOptions.cs).

## Aspire Integration (13.1)

The project uses .NET Aspire for orchestration and observability:

**AppHost** ([MudBlazor.Mcp.AppHost](../src/MudBlazor.Mcp.AppHost/Program.cs)):
```csharp
var builder = DistributedApplication.CreateBuilder(args);
builder.AddProject<Projects.MudBlazor_Mcp>("mudblazor-mcp");
builder.Build().Run();
```

**ServiceDefaults** ([Extensions.cs](../src/MudBlazor.Mcp.ServiceDefaults/Extensions.cs)) provides:
- OpenTelemetry (metrics, tracing, logging)
- Health checks (`/health`, `/alive`)
- Service discovery with resilience handlers
- OTLP exporter when `OTEL_EXPORTER_OTLP_ENDPOINT` is set

To add Aspire defaults to a service: `builder.AddServiceDefaults()`.

## Roslyn Parsing Deep Dive

The parsing layer extracts component metadata from C# source using Roslyn's syntax API.

### XmlDocParser - Core C# Parsing

[XmlDocParser.cs](../src/MudBlazor.Mcp/Services/Parsing/XmlDocParser.cs) extracts component info:

```csharp
// Parse syntax tree from source
var syntaxTree = CSharpSyntaxTree.ParseText(sourceCode);
var root = syntaxTree.GetRoot();

// Find public class declaration
var classDeclaration = root.DescendantNodes()
    .OfType<ClassDeclarationSyntax>()
    .FirstOrDefault(c => c.Modifiers.Any(m => m.IsKind(SyntaxKind.PublicKeyword)));
```

**Key extraction patterns:**
- **Parameters**: Properties with `[Parameter]` or `[CascadingParameter]` attributes
- **Events**: Properties of type `EventCallback` or `EventCallback<T>` with `[Parameter]`
- **Methods**: Public methods excluding lifecycle overrides (`OnInitialized`, `Dispose`, etc.)
- **XML docs**: Extracted from `DocumentationCommentTriviaSyntax` leading trivia

**Regex patterns** (generated via `[GeneratedRegex]` for performance):
- `CategoryTypesRegex` - Extracts `CategoryTypes.xxx` values
- `GenericArgumentRegex` - Extracts `<T>` from `EventCallback<T>`

### RazorDocParser - Documentation Extraction

[RazorDocParser.cs](../src/MudBlazor.Mcp/Services/Parsing/RazorDocParser.cs) parses `*Page.razor` files:
- Extracts `Title` and `SubTitle` from `<DocsPageHeader>` components
- Finds `<DocsPageSection>` blocks for structured content
- Identifies related components via `href="/components/..."` links

### ExampleExtractor - Code Examples

[ExampleExtractor.cs](../src/MudBlazor.Mcp/Services/Parsing/ExampleExtractor.cs) finds examples in:
`Docs/Pages/Components/{ComponentName}/*Example*.razor`

Splits files into markup and `@code` blocks, cleans directives (`@page`, `@using`).

### CategoryMapper - Component Organization

[CategoryMapper.cs](../src/MudBlazor.Mcp/Services/Parsing/CategoryMapper.cs) maps components to categories:
- Hardcoded category definitions from MudBlazor's `MenuService`
- Pattern-based inference: `*Button*` → "Buttons", `*Field*` → "Form Inputs"

## Code Patterns

### MCP Tools Pattern
Tools are static methods with DI parameters and `[McpServerTool]` + `[Description]` attributes:

```csharp
[McpServerToolType]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcbodge/MudMCP](https://github.com/mcbodge/MudMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
