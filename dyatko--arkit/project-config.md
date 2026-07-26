---
trigger: always_on
description: This document provides a comprehensive overview of Arkit for AI agents to quickly understand the project structure, functionality, and codebase.
---

# Arkit - AI Agent Documentation

This document provides a comprehensive overview of Arkit for AI agents to quickly understand the project structure, functionality, and codebase.

## Project Overview

**Arkit** is a static code analysis and visualization tool written in TypeScript that automatically generates architecture diagrams from JavaScript, TypeScript, Flow, Vue, and Nuxt codebases.

- **Repository**: https://github.com/dyatko/arkit
- **Website**: https://arkit.pro
- **NPM**: https://www.npmjs.com/arkit
- **License**: MIT
- **Current Version**: 2.0.7

## What Does Arkit Do?

Arkit analyzes source code files in a project, identifies dependencies between modules, groups them into architectural components, and generates visual diagrams in multiple formats (SVG, PNG, PlantUML).

### Key Use Cases

1. **Architecture Documentation**: Auto-generate and maintain up-to-date architecture diagrams
2. **Code Understanding**: Quickly visualize dependencies in unfamiliar codebases
3. **Onboarding**: Help new team members understand project structure
4. **CI/CD Integration**: Keep architecture documentation synchronized with code
5. **Code Reviews**: Visualize impact of changes on system architecture

## How It Works

### Processing Pipeline

```
CLI/API Input → Config → Parser → Generator → PlantUML → Converter → SVG/PNG Output
```

1. **Configuration** (`config.ts`): Loads CLI args, JSON config, or package.json settings
2. **Parser** (`parser.ts`): Uses `ts-morph` to analyze TypeScript/JavaScript files and extract imports/exports
3. **Generator** (`generator.ts`): Organizes files into component groups and builds dependency graph
4. **PUML** (`puml.ts`): Converts dependency graph to PlantUML format
5. **Converter** (`converter.ts`): Converts PlantUML to SVG/PNG locally using node-plantuml
6. **Filesystem** (`filesystem.ts`): Saves generated diagrams to disk

## Architecture & Source Files

### Core Components

#### `src/arkit.ts` - Main Entry Point
- Orchestrates the entire pipeline
- Exports main `arkit()` function and helper functions
- Manages progress bar display

#### `src/cli.ts` - Command Line Interface
- Uses `yargs` for argument parsing
- Defines CLI options: directory, config, output, first, exclude
- Default output: `arkit.svg`

#### `src/config.ts` - Configuration Management
- Loads configuration from multiple sources (CLI, JSON, package.json)
- Validates against JSON schema
- Merges and normalizes configuration options
- Supports component grouping and output customization

#### `src/parser.ts` - Code Analysis
- Uses `ts-morph` (TypeScript Compiler API wrapper)
- Analyzes source files to extract:
  - Import statements
  - Export declarations
  - Module dependencies
  - File paths and patterns
- Supports TypeScript, JavaScript, Flow, Vue, Nuxt

#### `src/generator.ts` - Graph Generation
- Transforms parsed files into component groups
- Builds dependency graph between components
- Applies filtering rules (exclude patterns)
- Organizes output according to configuration

#### `src/puml.ts` - PlantUML Generation
- Converts component graph to PlantUML syntax
- Handles component grouping and relationships
- Generates PlantUML markup for diagrams

#### `src/converter.ts` - Format Conversion
- Converts PlantUML to SVG/PNG locally using `node-plantuml`
- Handles direct PUML output
- Manages local Java process execution for PlantUML rendering
- Provides helpful error messages if Java is not installed

#### `src/filesystem.ts` - File Operations
- File discovery and pattern matching
- File reading and writing
- Path resolution

#### `src/utils.ts` - Utility Functions
- Path manipulation
- Array helpers
- Logging utilities

#### `src/logger.ts` - Logging
- Uses `pino` logger
- Supports different log levels via LEVEL environment variable

#### `src/types.ts` - TypeScript Type Definitions
- Core interfaces and types used throughout the application

#### `src/schema.ts` - Configuration Schema
- JSON schema definition for configuration validation
- Used to generate `schema.json` for IDE autocomplete

## Configuration Options

### CLI Arguments

```bash
-d, --directory   # Working directory (default: ".")
-c, --config      # Config file path (default: "arkit.json")
-o, --output      # Output path or type: svg, png, puml (default: "arkit.svg")
-f, --first       # File patterns to prioritize in graph
-e, --exclude     # File patterns to exclude (default: test files, dist, etc.)
```

### JSON Configuration (`arkit.json`)

```json
{
  "$schema": "https://arkit.pro/schema.json",
  "excludePatterns": ["test/**", "**/*.test.*"],
  "components": [
    {
      "type": "Component",
      "patterns": ["**/*.ts"]
    }
  ],
  "output": [
    {
      "path": "arkit.svg",
      "groups": [
        {
          "first": true,
          "components": ["Component"]
        }
      ]
    }
  ]
}
```

### package.json Configuration

Configuration can also be placed in `package.json` under the `"arkit"` key.

## Dependencies

### Production Dependencies
- **ts-morph**: TypeScript Compiler API wrapper for code analysis
- **yargs**: CLI argument parsing
- **pino**: Fast logger
- **nanomatch**: File pattern matching (globbing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dyatko/arkit](https://github.com/dyatko/arkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
