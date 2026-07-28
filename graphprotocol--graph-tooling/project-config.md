---
trigger: always_on
description: description: Monorepo for The Graph Protocol tooling including CLI and TypeScript bindings for subgraph development
---

---
description: Monorepo for The Graph Protocol tooling including CLI and TypeScript bindings for subgraph development
globs: ["**/*.ts", "**/*.js", "**/*.json", "**/*.yaml", "**/*.md", "**/subgraph.yaml", "**/schema.graphql"]
---

# The Graph Tooling

@context {
    "type": "monorepo",
    "purpose": "subgraph_development_tools",
    "organization": "graphprotocol",
    "license": "(Apache-2.0 OR MIT)",
    "node_version": ">=20.0.0",
    "package_manager": "pnpm"
}

@structure {
    "workspace": {
        "packages": ["packages/*", "examples/*", "website"],
        "main_tools": [
            "@graphprotocol/graph-cli",
            "@graphprotocol/graph-ts"
        ]
    },
    "architecture": {
        "monorepo": "pnpm workspace",
        "build_system": "TypeScript + AssemblyScript",
        "testing": "vitest",
        "release": "changesets"
    }
}

## Core Packages

### @graphprotocol/graph-cli

@package_info {
    "name": "@graphprotocol/graph-cli",
    "description": "CLI for building and deploying subgraphs to The Graph",
    "version": "0.97.1",
    "bin": "graph",
    "main_commands": [
        "add",
        "auth",
        "build",
        "clean",
        "codegen",
        "create",
        "deploy",
        "init",
        "local",
        "publish",
        "remove",
        "test"
    ]
}

The primary CLI tool for subgraph developers providing:
- Project initialization and scaffolding
- Contract ABI integration
- Code generation from GraphQL schema
- Local development server
- Deployment to The Graph Network
- Authentication and authorization
- Testing and validation

### @graphprotocol/graph-ts

@package_info {
    "name": "@graphprotocol/graph-ts",
    "description": "TypeScript/AssemblyScript library for writing subgraph mappings",
    "version": "0.38.1",
    "language": "AssemblyScript",
    "target": "WebAssembly"
}

TypeScript bindings and runtime for subgraph mapping development:
- Type-safe blockchain data access
- Entity store operations
- Event handler utilities
- Built-in helper functions

## Development Environment

@development_setup {
    "requirements": {
        "node": ">=20.18.1",
        "pnpm": "9.x",
        "typescript": "^5.7.2"
    },
    "commands": {
        "install": "pnpm install",
        "build": "pnpm build",
        "test": "pnpm test",
        "lint": "pnpm lint"
    }
}

### Project Structure

@project_structure {
    "packages/cli/": "Graph CLI implementation",
    "packages/ts/": "TypeScript/AssemblyScript bindings",
    "examples/": "Example subgraphs for various chains",
    "website/": "Documentation website",
    "patches/": "Package patches for dependencies"
}

## Supported Networks

@supported_networks [
    "ethereum",
    "polygon", 
    "near",
    "cosmos",
    "arweave",
    "substreams"
]

## Examples

@examples {
    "ethereum": [
        "ethereum-gravatar",
        "ethereum-basic-event-handlers",
        "example-subgraph"
    ],
    "polygon": [
        "matic-lens-protocol-posts-subgraph"  
    ],
    "near": [
        "near-blocks",
        "near-receipts"
    ],
    "cosmos": [
        "cosmos-validator-rewards",
        "cosmos-validator-delegations",
        "cosmos-osmosis-token-swaps",
        "cosmos-block-filtering"
    ],
    "arweave": [
        "arweave-blocks-transactions"
    ],
    "advanced": [
        "substreams-powered-subgraph",
        "aggregations"
    ]
}

## CLI Usage Patterns

@cli_patterns {
    "initialization": {
        "command": "graph init",
        "purpose": "Create new subgraph project",
        "options": ["--product", "--network", "--contract-name"]
    },
    "development": {
        "command": "graph codegen && graph build",
        "purpose": "Generate code and build subgraph",
        "watch": "graph build --watch"
    },
    "deployment": {
        "hosted_service": "graph deploy --product hosted-service",
        "decentralized_network": "graph deploy --product subgraph-studio"
    },
    "testing": {
        "command": "graph test",
        "purpose": "Run subgraph tests"
    },
    "local_development": {
        "command": "graph local",
        "purpose": "Start local development environment"
    }
}

## TypeScript Integration

@typescript_usage {
    "mapping_files": {
        "location": "src/mapping.ts",
        "imports": ["@graphprotocol/graph-ts"],
        "exports": ["event handlers", "block handlers"]
    },
    "schema_types": {
        "generation": "graph codegen",
        "location": "generated/schema.ts",
        "usage": "Import entity classes"
    }
}

```typescript
// Example mapping function
import { Transfer } from '../generated/Contract/Contract'
import { User, Transfer as TransferEntity } from '../generated/schema'

export function handleTransfer(event: Transfer): void {
  let user = User.load(event.params.to.toHex())
  if (user == null) {
    user = new User(event.params.to.toHex())
    user.save()
  }
  
  let transfer = new TransferEntity(event.transaction.hash.toHex())
  transfer.from = event.params.from
  transfer.to = event.params.to
  transfer.value = event.params.value
  transfer.save()
}
```

## Configuration Files

@configuration {
    "subgraph.yaml": {
        "purpose": "Subgraph manifest",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [graphprotocol/graph-tooling](https://github.com/graphprotocol/graph-tooling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
