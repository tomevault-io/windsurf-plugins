---
trigger: always_on
description: When generating code for this repository:
---

# GitHub Copilot Instructions for plantuml-libs

## Priority Guidelines

When generating code for this repository:

1. **Version Compatibility**: Respect TypeScript 5.0+, Node.js 22.7+, and all npm dependencies as specified in package.json
2. **Context Files**: Follow patterns and standards defined in the .github/copilot directory
3. **Codebase Patterns**: Scan existing code in the source directory for established patterns
4. **Architectural Consistency**: Maintain the monolithic package-based architecture with clear separation between generator and library concerns
5. **Code Quality**: Prioritize maintainability, testability, and security in all generated code

## Technology Stack

### Core Technologies
- **TypeScript**: 5.0.3
  - Strict mode enabled
  - ES2021 target
  - Node module resolution
  - ESM module format
  - allowSyntheticDefaultImports: true
- **Node.js**: 22.7.4
- **Build System**: npm scripts
- **Code Generation**: ts-node for runtime TypeScript execution

### Key Dependencies
- **CLI Framework**: yargs (command-line argument parsing)
- **File I/O**: fs-extra (promise-based file operations)
- **Utilities**: lodash, moment, glob
- **Parsing/Generation**: cheerio, csv-parse, html-minifier-terser, marked, yaml
- **Versioning**: standard-version (semantic versioning)

## Code Organization & Patterns

### Project Structure
```
source/
├── library/
│   ├── index.ts              # Main library factory and exports
│   └── packages/             # Individual package factories (AWS, Azure, C4, etc.)
│       └── {package-name}/
│           └── index.ts      # Package-specific factory
└── generator/
    ├── website/              # Website generation pipeline (ETL stages)
    │   ├── index.ts          # Entry point with yargs CLI
    │   ├── stage.ts          # Stage interface
    │   ├── extract.ts        # Extract stage
    │   ├── transform.ts      # Transform stage
    │   ├── load.ts           # Load stage
    │   ├── config.ts         # Configuration interface
    │   ├── resource.ts       # Resource types
    │   └── ...               # Supporting modules
    └── workdir/              # Work directory generation
```

### Two-Tier Architecture: Library and Generator

**Library** (source/library/):
- Implements factory pattern for each technology package (AWS, Azure, C4, etc.)
- Each factory generates raw PlantUML sprite and icon resources
- Packages are independently designed but share common patterns
- Run with: `npm run generate:workdir`

**Generator** (source/generator/):
- **workdir/**: Orchestrates library packages, produces `.workdir/` manifest
  - Aggregates all packages into a single `library.yaml` manifest
  - Copies supporting resources (templates, assets)
  - Used by: `npm run generate:workdir`
  
- **website/**: ETL pipeline for rendering documentation
  - **Extract**: Parses `.workdir/library.yaml` and asset manifests
  - **Transform**: Processes resources, generates Markdown documentation
  - **Load**: Writes distribution/* with PlantUML files and docs
  - Used by: Docker container (plantuml-generator image)
  - Validates structure with: `npm run generate:website`

The full build (scripts/generate-library.sh) chains: workdir → website → distribution/

### Architecture Patterns
- **Pipeline/ETL Pattern**: Website generation uses Extract → Transform → Load stages
- **Factory Pattern**: Each package implements a factory interface
- **Configuration-Driven**: Extensive use of yargs for CLI configuration
- **Promise-Based**: Async/await throughout; no callbacks
- **Stream-Based Processing**: Large data processing uses Node.js streams

### Naming Conventions
- **Class Names**: PascalCase (e.g., `ExtractStage`, `TransformStage`)
- **Interface Names**: PascalCase with optional `I` prefix not used (e.g., `Stage`, `Config`)
- **Variables**: camelCase for local variables and parameters
- **Constants**: UPPER_SNAKE_CASE (e.g., `PACKAGE_FACTORIES`)
- **Imports**: Use short aliases for common libraries
  - `import P from "path"` (path module)
  - `import Fe from "fs-extra"`
  - `import Fx from "fs-extra".promises`
  - `import U from "util"`

### Interface Patterns
- Define interfaces inline with implementations
- Use `readonly` for immutable properties
- Declare generic interfaces for reusable patterns (e.g., `Stage<I, O>`)
- Export interfaces alongside implementations

### Class Patterns
- Private constructor with static factory method: `static create(...): ClassName`
- Implement interfaces explicitly
- Constructor injection for dependencies
- Methods marked `async` return `Promise<T>`
- Use object destructuring in method signatures

Example:
```typescript
export class ExtractStage implements Stage<void, ExtractStageOutput> {
  constructor(readonly config: Config, readonly input: undefined) {}

  static create(config: Config): ExtractStage {
    return new ExtractStage(config, undefined)
  }

  async execute(): Promise<ExtractStageOutput> {
    // implementation
  }
}
```

### Import Organization
1. Standard library imports (path, util, stream)
2. External package imports (yargs, fs-extra, glob)
3. Local module imports (relative paths)
4. No blank lines between import groups (contrary to many style guides)

Example:
```typescript
import P from "path"
import yargs from "yargs"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmorin/plantuml-libs](https://github.com/tmorin/plantuml-libs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
