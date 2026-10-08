---
trigger: always_on
description: This document provides instructions for AI agents on how to work with and extend the Make CLI repository. The CLI is a standalone command-line tool that interacts with the Make automation platform. It depends on `@makehq/sdk` for all API access, types, and tool definitions.
---

# Make CLI - AI Agents Guide

## Overview

This document provides instructions for AI agents on how to work with and extend the Make CLI repository. The CLI is a standalone command-line tool that interacts with the Make automation platform. It depends on `@makehq/sdk` for all API access, types, and tool definitions.

## Repository Structure

```
make-cli/
├── src/
│   ├── index.ts        # Executable entry point: sets up Commander, registers all commands
│   ├── commands.ts     # Builds CLI commands from @makehq/sdk tool definitions
│   ├── auth.ts         # Resolves API key and zone from flags, env vars, or config file
│   ├── config.ts       # Reads/writes local credentials file (~/.config/make-cli/config.json)
│   ├── login.ts        # Hand-crafted login, logout, whoami commands
│   ├── output.ts       # Output formatting: json, compact, table
│   ├── categories.ts   # Display titles and groupings for command categories
│   └── version.ts      # Auto-generated version constant (from scripts/build-version.mjs)
├── test/
│   ├── commands.spec.ts  # Unit tests for CLI utilities and command building
│   ├── auth.spec.ts      # Unit tests for resolveAuth config file fallback
│   └── config.spec.ts    # Unit tests for config file path resolution and I/O
├── scripts/
│   └── build-version.mjs     # Writes src/version.ts from package.json version
└── dist/                     # Compiled output (auto-generated)
    └── index.js              # The executable CLI (ESM with shebang)
```

## Dependency on `@makehq/sdk`

All API functionality comes from the `@makehq/sdk` package. The CLI imports:

| Import        | Source              | Purpose                                             |
| ------------- | ------------------- | --------------------------------------------------- |
| `Make`        | `@makehq/sdk`       | API client — instantiated per command invocation    |
| `MakeError`   | `@makehq/sdk`       | Typed API error with `statusCode` and `message`     |
| `JSONValue`   | `@makehq/sdk`       | Generic JSON value type                             |
| `MakeTools`   | `@makehq/sdk/tools` | Array of all Make SDK tool definitions              |
| `MakeTool`    | `@makehq/sdk/tools` | Type describing a single tool                       |
| `JSONSchema`  | `@makehq/sdk/tools` | JSON Schema type for tool input parameters          |

## How the CLI Works

The CLI uses an **auto-discovery pattern**: it reads the `MakeTools` array from `@makehq/sdk/tools` and dynamically registers each tool as a CLI subcommand. No command wiring is done by hand.

### Command registration flow

1. `src/index.ts` creates a Commander program with global flags (`--api-key`, `--zone`, `--output`)
2. It calls `buildCommands(program, MakeTools)` from `src/commands.ts`
3. `buildCommands` groups tools by `tool.category` and creates nested subcommands:
    - Category → top-level command (e.g. `scenarios`, `data-stores`, `sdk-apps`)
    - Tool action → subcommand (e.g. `list`, `get`, `create`)
4. Each subcommand's options are derived from `tool.inputSchema.properties`
5. On execution, the tool's `execute(make, args)` function is called

### Positional argument for resource-level actions

Tools that operate on a single resource (typically `get` / `update` / `delete` / action-style tools) declare the owning input property via `tool.resourceId` (e.g. `dataStructureId` for `data-structures_get`, `executionId` for `executions_get`, `key` for `data-store-records_update`). For these tools the CLI exposes that value as a positional argument. The original descriptive long-form flag is kept as an alternative for scripted / explicit use, so both of these work and map to the same SDK input:

```
make-cli data-structures get 178
make-cli data-structures get --data-structure-id=178
```

When the tool also declares a parent scope (`tool.scopeId`), that stays a named flag — only the resource's own id becomes positional:

```
make-cli executions get abc --scenario-id=925
```

Behavior details:

- The positional argument is registered as optional at the Commander level (`[resource-id]`) so either invocation style parses cleanly. Presence is enforced in the action handler based on the JSON Schema's `required` list.
- Supplying the value both positionally and via the flag is rejected with an explicit error.
- The positional is not registered when `tool.resourceId` is unset (collection-level `list` / `create`) or when it points at a property that isn't part of the schema.
- Generated help text shows the positional in the Usage line and in an `Arguments:` section; built-in examples are rendered using the positional form.

See `deriveSelfIdentifier` and `registerToolAsCommand` in `src/commands.ts`.

### Tool name → CLI command mapping

Tool names follow `{category}_{action}` where category dots become hyphens:

```
scenarios_list          → make scenarios list
data-stores_get         → make data-stores get
sdk-apps_get-section    → make sdk-apps get-section
```

### Auth resolution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [integromat/make-cli](https://github.com/integromat/make-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
