---
trigger: always_on
description: transforms accumulated args in-place
---

# CLI-Forge Architecture

A type-safe CLI builder library for Node.js with automatic help generation, middleware support, and comprehensive documentation tooling.

## Repository Structure

```
packages/
├── cli-forge/          # High-level CLI builder (user-facing API)
├── parser/             # Low-level argument parser (type inference engine)
examples/               # Runnable examples with YAML front-matter
e2e/                    # End-to-end tests (runs examples)
docs-site/              # Docusaurus documentation site
type-tests/             # TypeScript type inference validation
tools/scripts/          # Build and collection utilities
```

---

## CLI Layer → Parser Layer

The architecture follows a **layered design** where `cli-forge` wraps `@cli-forge/parser`:

```
┌─────────────────────────────────────────────────────────────┐
│                       cli-forge                             │
│  - Command tree management                                  │
│  - Handler execution with middleware                        │
│  - Help formatting & interactive shell                      │
│  - Configuration providers                                  │
│  - TestHarness for testing                                  │
├─────────────────────────────────────────────────────────────┤
│                     @cli-forge/parser                       │
│  - Type-safe option registration                            │
│  - Argument tokenization & parsing                          │
│  - Value coercion & validation                              │
│  - Environment variable population                          │
│  - Config file loading with extends                         │
└─────────────────────────────────────────────────────────────┘
```

### Parser Layer (`packages/parser`)

The parser is the **type inference engine**. It provides:

- **ArgvParser<TArgs>** - Core class that accumulates types as options are added
- **Option types** - `string`, `number`, `boolean`, `array`, `object` with full TypeScript inference
- **Validation** - `choices`, `validate`, `required`, `conflicts`, `implies`
- **Value sources** - CLI args, environment variables, config files, defaults

Key files:
- `lib/parser.ts` - Main `ArgvParser` class
- `lib/option-types/` - Type definitions for each option kind
- `lib/parsers/` - Runtime value parsers (string, number, boolean, array, object)
- `lib/config-files/` - Configuration file loading with inheritance

**Type inference flow:**
```typescript
parser()                                    // ArgvParser<ParsedArgs>
  .option('name', { type: 'string' })       // ArgvParser<{ name: string }>
  .option('port', { type: 'number' })       // ArgvParser<{ name: string; port: number }>
  .option('config', {
    type: 'object',
    properties: {
      host: { type: 'string' },
      db: { type: 'string', required: true }
    }
  })  // ArgvParser<{ name: string; port: number; config: { host?: string; db: string } }>
```

### CLI Layer (`packages/cli-forge`)

The CLI layer adds **command management** and **execution semantics**:

- **InternalCLI** - Wraps `ArgvParser`, adds command tree
- **Command registration** - Subcommands with builders and handlers
- **Middleware** - Transform args before handler execution
- **Help formatting** - Automatic help text generation
- **Interactive shell** - REPL for command exploration

Key files:
- `lib/public-api.ts` - Main `CLI<TArgs>` interface
- `lib/internal-cli.ts` - `InternalCLI` implementation
- `lib/format-help.ts` - Help text formatting
- `lib/middleware/zod.ts` - Zod validation middleware

**Usage pattern:**
```typescript
import { cli } from 'cli-forge';

const app = cli('my-app')
  .option('verbose', { type: 'boolean', alias: ['v'] })
  .command('serve', {
    builder: (cmd) => cmd
      .option('port', { type: 'number', default: 3000 }),
    handler: (args) => {
      // args.verbose, args.port fully typed
    }
  });

await app.forge();
```

### Lazy Subcommand Building

Subcommands are built **on-demand** via the parser's `unmatchedParser` callback:

1. User runs: `my-app serve --port 8080`
2. Parser encounters `serve` as unmatched argument
3. CLI looks up `serve` command, calls its builder
4. Builder registers `--port` option
5. Parser continues with augmented options

This enables deep command hierarchies without upfront registration cost.

---

## Examples System

Examples serve **three purposes**: documentation, e2e tests, and playground.

### Example Structure

**Single-file examples** (`examples/*.ts`):
```typescript
// ---
// id: basic-cli
// title: Basic CLI
// description: |
//   A simple CLI demonstrating basic options.
// commands:
//   - command: '{filename} --name World'
//     assertions:
//       - contains: 'Hello, World!'
// ---

import { cli } from 'cli-forge';

cli('hello')
  .option('name', { type: 'string', default: 'World' })
  .forge();
```

**Multi-file examples** (`examples/*/meta.yml` + files):
```yaml
id: configuration-files
title: Configuration Files
description: Loading config from JSON files
entryPoint: ./cli.ts
fileMap:
  './cli.ts': 'cli.ts'
  './config.json': 'config.json'
commands:
  - command: '{filename}'
```

### Documentation Generation

The `docs-site/src/plugins/examples-plugin.ts` Docusaurus plugin:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentEnder/cli-forge](https://github.com/AgentEnder/cli-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
