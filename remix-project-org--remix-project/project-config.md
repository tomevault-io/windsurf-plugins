---
trigger: always_on
description: > **Purpose**: This document provides context for Claude Code (AI assistant) when working on the Remix Project. Read this at the start of conversations to understand the codebase structure, conventions, and common patterns.
---

# Claude Code Context for Remix Project

> **Purpose**: This document provides context for Claude Code (AI assistant) when working on the Remix Project. Read this at the start of conversations to understand the codebase structure, conventions, and common patterns.

## Project Identity

**Remix Project** is a comprehensive smart contract development toolset for Ethereum, including Remix IDE (web and desktop), plugins, and libraries for Solidity development, testing, debugging, and deployment.

- **Repository**: https://github.com/remix-project-org/remix-project
- **Architecture**: Nx monorepo with Yarn workspaces
- **Main Branch**: `master`
- **Default Project**: `remix-ide`
- **Tech Stack**: React 18, TypeScript, Nx 15.7.1, Webpack 5, Node 20+

## Critical Context

### Monorepo Structure

```
remix-project/
├── apps/          # 16+ deployable applications
│   ├── remix-ide/              # Main web IDE (default project)
│   ├── remixdesktop/           # Electron desktop app
│   ├── remix-ide-e2e/          # Nightwatch E2E tests
│   ├── circuit-compiler/       # Circuit compilation
│   ├── contract-verification/  # Contract verification tools
│   ├── noir-compiler/          # Noir language support
│   ├── solidity-compiler/      # Solidity compiler wrapper
│   └── [others...]
└── libs/          # 19+ shared libraries
    ├── remix-analyzer/         # Static analysis & security checks
    ├── remix-debug/            # Transaction debugger
    ├── remix-solidity/         # Compiler management
    ├── remix-tests/            # Solidity unit testing
    ├── remix-ai-core/          # AI features & MCP server ⭐
    ├── remix-core-plugin/      # Plugin base classes
    ├── remix-ui/               # React component library (many sub-packages)
    ├── remixd/                 # Local filesystem daemon
    └── [others...]
```

### Key Libraries to Know

**remix-ai-core** (Important for AI features):
- Location: `libs/remix-ai-core/src/`
- Structure:
  - `agents/`: Code explanation, security, completion, workspace agents
  - `remix-mcp-server/`: MCP (Model Context Protocol) server implementation
    - `handlers/`: Tool handlers (compilation, debugging, deployment, file management)
    - `providers/`: Resource providers (compilation, project, deployment, tutorials)
    - `middleware/`: Security, validation
  - `inferencers/`: Local (Ollama) and remote AI model integration
  - `prompts/`: System prompts and prompt builders

**remix-core-plugin**:
- Base classes for plugin development
- Plugin architecture based on `@remixproject/engine`
- Event-driven communication

**remix-ui**:
- Modular React components in separate packages
- Each concern has its own sub-package (e.g., `remix-ui/terminal`, `remix-ui/editor`)
- Uses Bootstrap 5 and React hooks

**remix-ide-e2e**:
- Nightwatch-based E2E tests
- Tests organized by feature with groups: `<testname>_group<number>.test.js`
- Group tags allow parallel execution: `#group1`, `#group2`, etc.
- Special requirements:
  - `ballot` tests need Ganache running locally
  - `remixd` tests need remixd daemon running
  - `gist` tests need GitHub token in `.env`

## Development Commands

```bash
# Initial setup
yarn install
yarn run build:libs    # Always build libs first
yarn build             # Build entire project
yarn serve             # Dev server (http://127.0.0.1:8080)
yarn serve:hot         # With hot module reload

# Library-specific
nx build <library-name>
nx test <library-name>
nx lint <library-name>

# E2E testing
yarn build:e2e                                    # Build tests first
yarn test:e2e --test=<testname> --group=group1   # Run specific group
yarn test:e2e --test=<testname>                  # Run all groups
yarn run select_test                             # Interactive selector

# Production
yarn run build:production
yarn run serve:production

# Utilities
nx dep-graph          # View dependency graph
yarn format           # Format code
```

## Important Patterns & Conventions

### File Organization
- TypeScript path aliases defined in `tsconfig.base.json`
- Use `@remix-project/<library-name>` imports, not relative paths across libraries
- Each library has: `src/`, `README.md`, `package.json`, `tsconfig.json`

### Testing Patterns
- Unit tests: Jest, located alongside source files
- E2E tests: Nightwatch, in `apps/remix-ide-e2e/src/tests/`
- Group tags in E2E: `'Test description #group1': function (browser) { ... }`
- Must add `'@disabled': true` to test file metadata when using groups

### Plugin Architecture
- Plugins extend base classes from `remix-core-plugin`
- Communication via event system
- API contracts defined in `remix-api`
- Uses `@remixproject/engine` framework

### UI Component Pattern
```typescript
import React from 'react'

interface ComponentProps {
  // props
}

export const Component: React.FC<ComponentProps> = (props) => {
  // React hooks for state
  // Bootstrap 5 for styling
  return <div>...</div>
}
```

### Internationalization
- Uses react-intl with FormattedMessage
- Translations managed via CrowdIn (NOT GitHub PRs)
- Locale files in `apps/remix-ide/src/app/tabs/locales/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remix-project-org/remix-project](https://github.com/remix-project-org/remix-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
