---
trigger: always_on
description: Blake is a Blazor-based static site generator that embraces **Occam's Razor** and **convention-over-configuration** principles. It allows developers to create static websites using familiar .NET technologies: Markdown files, Razor templates, and Blazor components.
---

# Blake - Copilot Instructions

Blake is a Blazor-based static site generator that embraces **Occam's Razor** and **convention-over-configuration** principles. It allows developers to create static websites using familiar .NET technologies: Markdown files, Razor templates, and Blazor components.

## Development Guidelines

**Always search Microsoft documentation (MS Learn) when working with .NET, Windows, or Microsoft features, or APIs.** Use the `microsoft_docs_search` tool to find the most current information about capabilities, best practices, and implementation patterns before making changes.

## Project Philosophy

Blake is guided by simplicity and minimalism:

- **Minimal assumptions**: Use familiar .NET and Blazor patterns
- **Convention-over-configuration**: Folder structure determines behavior
- **Transparency**: Developers should understand their build process
- **Familiarity**: Leverage existing .NET/Blazor knowledge

**Core Principle**: "Bake your Blazor into beautiful static sites" with the fewest assumptions and maximum developer control.

## Architecture Overview

### Project Structure

```
Blake/
├── src/
│   ├── Blake.CLI/              # Command-line interface (`blake` tool)
│   ├── Blake.Types/            # Core types (PageModel, SiteTemplate)
│   ├── Blake.BuildTools/       # Build engine, plugin system, BlakeContext
│   └── Blake.MarkdownParser/   # Markdown parsing and rendering
├── tests/
│   └── Blake.BuildTools.Tests/ # Unit tests
├── TemplateRegistry.json      # Community template registry
└── Blake.sln                  # Solution file
```

### Key Components

1. **Blake.CLI**: Main entry point providing `bake`, `new`, `serve`, and `init` commands
2. **Blake.BuildTools**: Core build functionality, plugin architecture, and BlakeContext
3. **Blake.Types**: Shared types including `PageModel` for content metadata
4. **Blake.MarkdownParser**: Handles Markdown parsing with frontmatter support

### Build Flow

```plaintext
Markdown + template.razor → blake bake → .generated/*.razor → dotnet run → Blazor app
```

## Core Concepts

### Convention-Based Templates

- Each content folder (e.g., `/Posts`, `/Pages`) can contain a `template.razor` file
- Templates use standard Razor syntax with access to `PageModel` data
- Generated pages include proper `@page` routing directives
- Content is rendered using folder structure for URL generation

### Plugin System

Blake supports extensibility through the `IBlakePlugin` interface:

```csharp
public interface IBlakePlugin
{
    Task BeforeBakeAsync(BlakeContext context, ILogger? logger = null);
    Task AfterBakeAsync(BlakeContext context, ILogger? logger = null);
}
```

**Plugin Development Guidelines:**
- Plugins have access to `BlakeContext` containing project metadata and page collections
- Use `BeforeBakeAsync` for pre-processing (e.g., content validation)
- Use `AfterBakeAsync` for post-processing (e.g., adding metadata, external integrations)
- Example plugins: `BlakePlugin.ReadTime` (reading time calculation), `BlakePlugin.DocsRenderer` (TOC generation)

### Template Registry

Community templates are managed via `TemplateRegistry.json`:
- Templates include metadata: name, description, author, repository URL
- CLI supports `blake new --template <name>` and `blake new --list`
- Templates are Git repositories that can be cloned and customized

## Development Workflows

### Building the Project

**Note**: Blake targets .NET 9.0, ensure you have the correct SDK installed. If .NET 9.0 is not available in the environment, install it using:
```bash
curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel 9.0
export PATH="$HOME/.dotnet:$PATH"
```

```bash
# Build the solution
dotnet build Blake.sln

# Run tests
dotnet test

# Pack NuGet packages locally (for development)
./Build-LocalPackages.ps1
```

### CLI Development

When working on the CLI (`Blake.CLI`):
- Entry point is `Program.cs` with command routing
- Commands include: `init`, `bake`, `serve`, `new`
- Use structured logging for user feedback
- Support verbosity levels and error handling
- Follow existing patterns for argument parsing

### Plugin Development

For new plugins:
1. Reference `Blake.BuildTools` for `IBlakePlugin` interface
2. Access project data through `BlakeContext`
3. Use `MarkdownPages` (before baking) or `GeneratedPages` (after baking)
4. Add metadata to `PageModel.Metadata` dictionary for template access
5. Follow existing plugin patterns (see ReadTime and DocsRenderer examples)

### Template Development

For new site templates:
1. Create a standard Blazor WASM project
2. Add Blake MSBuild integration to `.csproj`
3. Include `template.razor` files in content folders
4. Use YAML frontmatter in Markdown files for metadata
5. Reference the generated `GeneratedContentIndex` for navigation

## Code Conventions

### General Guidelines

- Follow standard .NET coding conventions and naming patterns
- Use nullable reference types throughout the codebase
- Prefer explicit error handling over exceptions where appropriate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matt-goldman/blake](https://github.com/matt-goldman/blake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
