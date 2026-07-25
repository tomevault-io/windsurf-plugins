---
trigger: always_on
description: Node-RED extension providing WebSocket and REST API integration with Home Assistant. Built with TypeScript for both Node.js runtime and browser-based editor.
---

# Copilot Instructions for node-red-contrib-home-assistant-websocket

## Project Overview

Node-RED extension providing WebSocket and REST API integration with Home Assistant. Built with TypeScript for both Node.js runtime and browser-based editor.

**Tech Stack:**

- **Runtime**: Node.js v18.2+, Node-RED v3.1.1+
- **Language**: TypeScript (dual configs for server/client)
- **Package Manager**: pnpm v10.20.0 (required, never use npm)
- **Build System**: Custom build.js (esbuild, SASS, HTML processing)
- **Testing**: Vitest (481+ unit tests)
- **HA Integration**: WebSocket + REST API (requires HA 2024.3+)

**Critical First-Time Setup:**

```bash
corepack enable && corepack enable pnpm  # Enable pnpm
pnpm install                             # Install dependencies
pnpm build                               # Initial build
```

## Architecture Patterns

### Node Structure (3-Part System)

Each node in `src/nodes/{node}/` consists of:

1. **`index.ts`** - Server-side logic (Node.js runtime)
   - TypeScript with Joi validation
   - Controller pattern extending `BaseController`
   - InputService for message parsing
   - Compiled via `tsconfig.json`

2. **`editor.html`** - Client UI definition
   - Mustache templates for rendering
   - HTML form controls for configuration
   - Localization support via `data-i18n`
   - Bundled into dist during build

3. **`editor.ts`** - Client-side logic (browser runtime)
   - Node-RED editor integration
   - UI interactions and form validation
   - Compiled via `tsconfig.editor.json`
   - Must use browser-compatible APIs only

### Dual TypeScript Configurations

- **`tsconfig.json`**: Server code targeting Node.js
  - Compiles `src/nodes/**/index.ts`
  - Can use Node.js APIs (fs, path, etc.)
- **`tsconfig.editor.json`**: Editor code targeting browsers
  - Compiles `src/**/*.ts` excluding `index.ts`
  - Browser-only APIs (DOM, localStorage, etc.)
  - No Node.js modules allowed

### Directory Structure

- **`src/nodes/`**: Individual node implementations (30+ nodes)
- **`src/common/`**: Shared services and controllers
  - `controllers/`: Base controller classes
  - `services/`: InputService, TypedInputService, JSONataService
  - `status/`: Node status management
  - `events/`: ClientEvents for HA WebSocket
- **`src/homeAssistant/`**: HA communication layer
  - WebSocket client
  - HTTP API client
  - Event subscriptions
- **`src/helpers/`**: Utility functions
- **`src/types/`**: TypeScript type definitions
- **`test/`**: Vitest unit tests with mocks

## Development Workflow

### Essential Commands

```bash
pnpm build     # Production build (~14s) - compiles TS, bundles assets
pnpm dev       # Development server with watch mode (ports 1880/3000)
pnpm test      # Run all unit tests (481+ tests, ~5s)
pnpm test:watch # Watch mode for test development
pnpm test:coverage # Generate coverage reports
pnpm lint      # ESLint + Prettier validation
pnpm lint:fix  # Auto-fix linting issues
```

### Build Process Details

The custom `build.js` script handles:

- **TypeScript compilation**: Two separate builds (server + editor)
- **SASS compilation**: Converts SCSS to CSS with autoprefixer
- **HTML processing**: Bundles editor.html with inlined CSS/JS
- **Asset copying**: Icons, locales, resources to dist/
- **Watch mode**: Auto-rebuilds on file changes during `pnpm dev`

### Testing Strategy

- **Framework**: Vitest with 481+ unit tests
- **Mocks**: Node-RED and HA WebSocket mocks in `test/mocks/`
- **Coverage**: v8 provider, HTML reports in `coverage/`
- **Patterns**: Test files mirror source structure `test/unit/nodes/{node}/*.test.ts`
- **Run tests after edits**: Always verify changes don't break existing functionality

### Node Development Pattern

**Complete Example Structure:**

```typescript
// src/nodes/action/index.ts - Server-side implementation
import Joi from "joi";
import { createControllerDependencies } from "../../common/controllers/helpers";
import { inputErrorHandler } from "../../common/errors/inputErrorHandler";
import InputService, {
  NodeInputs,
  ParsedMessage,
} from "../../common/services/InputService";
import Status from "../../common/status/Status";
import {
  BaseNode,
  BaseNodeProperties,
  OutputProperty,
} from "../../types/nodes";

// 1. Define node properties interface
export interface ActionNodeProperties extends BaseNodeProperties {
  action: string; // Home Assistant action (e.g., "light.turn_on")
  data: string; // Action data payload
  dataType: string; // Type: json, jsonata, msg, flow, global
  areaId?: string[]; // Target area IDs
  deviceId?: string[]; // Target device IDs
  entityId?: string[]; // Target entity IDs
  queue: Queue; // Message queue behavior
  outputProperties: OutputProperty[];
  blockInputOverrides: boolean;
}

export interface ActionNode extends BaseNode {
  config: ActionNodeProperties;
}

// 2. Define input mappings with validation
const inputs: NodeInputs = {
  action: {
    messageProp: "payload.action", // Read from msg.payload.action
    configProp: "action", // Fallback to node config
  },
  data: {
    messageProp: "payload.data",
    configProp: "data",
    default: {}, // Default value if not provided
  },
};


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zachowj/node-red-contrib-home-assistant-websocket](https://github.com/zachowj/node-red-contrib-home-assistant-websocket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
