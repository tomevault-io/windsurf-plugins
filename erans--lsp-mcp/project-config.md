---
trigger: always_on
description: This directory contains a comprehensive LSP (Language Server Protocol) to MCP (Model Context Protocol) bridge that provides Claude Code with advanced code understanding capabilities.
---

# LSP-MCP Bridge for Claude Code

This directory contains a comprehensive LSP (Language Server Protocol) to MCP (Model Context Protocol) bridge that provides Claude Code with advanced code understanding capabilities.

Built with TypeScript for modern, type-safe development.

## LSP Specification

**Implementation based on Language Server Protocol Specification 3.17.0**
- Specification URL: https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/
- Provides semantic code understanding that surpasses basic text-based tools

## How to Use

The bridge uses a YAML configuration file to define LSP servers. You need to:

1. **Create or customize a config file** with your LSP server paths
2. **Run the bridge** with the config file and desired LSP server key

### Configuration File Setup

The bridge requires a YAML configuration file that maps server keys to commands or HTTP URLs. A sample `lsp-servers.yaml` is provided:

```yaml
# LSP Server Configuration
# Define LSP servers with their commands or HTTP endpoints
# Key: server identifier, Value: command or HTTP URL

typescript: "typescript-language-server --stdio"
python: "pylsp"
go: "gopls"
rust: "rust-analyzer"
cpp: "clangd"

# HTTP-based servers
typescript-http: "http://localhost:8080"
python-remote: "http://lsp-server.internal:3333"
```

**Important**: You may need to update the paths in the config file to match your system's LSP server installations, or create your own config file with the correct paths.

### Running the Bridge

```bash
# Using the provided config file
./dist/cli.js --config lsp-servers.yaml --lsp typescript

# Using a custom config file
./dist/cli.js --config /path/to/my-lsp-config.yaml --lsp python

# With workspace and verbose logging
./dist/cli.js --config lsp-servers.yaml --lsp go --workspace /path/to/project --verbose
```

### Examples for Different Languages

```bash
# TypeScript/JavaScript development
./dist/cli.js --config lsp-servers.yaml --lsp typescript

# Python development  
./dist/cli.js --config lsp-servers.yaml --lsp python

# Go development
./dist/cli.js --config lsp-servers.yaml --lsp go

# Rust development
./dist/cli.js --config lsp-servers.yaml --lsp rust

# C++ development
./dist/cli.js --config lsp-servers.yaml --lsp cpp

# HTTP-based LSP server
./dist/cli.js --config lsp-servers.yaml --lsp typescript-http
```

## Available Tools

The bridge provides 22 LSP-powered tools that replace and enhance basic code analysis:

### Core Navigation (Replaces Grep/Find)
- `lsp_goto_definition` - Precise definition location
- `lsp_goto_declaration` - Declaration navigation
- `lsp_goto_implementation` - Implementation finding
- `lsp_goto_type_definition` - Type definition lookup
- `lsp_find_references` - Workspace-wide reference search

### Symbol Discovery (Enhanced Search)
- `lsp_document_symbols` - Structured symbol listing
- `lsp_workspace_symbols` - Intelligent symbol search
- `lsp_document_highlight` - Symbol occurrence highlighting

### Code Intelligence (Beyond Basic Tools)
- `lsp_hover` - Rich documentation and type information
- `lsp_completion` - Context-aware code completion
- `lsp_signature_help` - Function signature assistance
- `lsp_code_action` - Quick fixes and refactoring
- `lsp_code_lens` - Inline actionable information

### Advanced Analysis
- `lsp_semantic_tokens` - Semantic syntax highlighting
- `lsp_inlay_hints` - Type annotations and parameter names
- `lsp_folding_range` - Code structure analysis
- `lsp_selection_range` - Smart selection capabilities
- `lsp_document_link` - Reference and URL detection

### Editing & Formatting
- `lsp_format_document` - Language-specific formatting
- `lsp_format_range` - Partial document formatting  
- `lsp_rename` - Safe project-wide renaming

### Workspace Operations
- `lsp_execute_command` - Server-specific commands

## Advantages over Basic Tools

| LSP Tool | Replaces | Advantage |
|----------|----------|-----------|
| `lsp_goto_definition` | grep, find | Semantic understanding vs text matching |
| `lsp_find_references` | grep -r | Cross-file analysis with context |
| `lsp_document_symbols` | grep, regex | Structured symbol information |
| `lsp_hover` | manual lookup | Rich docs, types, examples |
| `lsp_completion` | manual typing | Context-aware suggestions |
| `lsp_rename` | find+replace | Safe refactoring across project |

## Development Commands

### Quick Start with npm
```bash
# Install dependencies
npm install

# Build and run with different language servers (uses lsp-servers.yaml)
npm run build
npm run dev:ts        # TypeScript development (default)
npm run dev:python    # Python development
npm run dev:go        # Go development  
npm run dev:rust      # Rust development
npm run dev:cpp       # C++ development

# Development workflow
npm run build         # Build the application
npm run test          # Run tests
npm run lint          # Lint code
npm run format        # Format code
npm run type-check    # Type checking
npm run clean         # Clean artifacts
```

### Quick Start with Makefile
```bash
# Install and build
make install
make build


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erans/lsp-mcp](https://github.com/erans/lsp-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
