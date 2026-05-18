---
trigger: always_on
description: This document outlines the core principles, standards, and architecture for the Figma MCP Write Server project, tailored for development with Gemini.
---

# Gemini Code Project Context

This document outlines the core principles, standards, and architecture for the Figma MCP Write Server project, tailored for development with Gemini.

## 1. Commit Message Format

All commit messages must follow this format for consistency and automated changelog generation.

```
type: brief description (vX.X.X)

- Bullet point describing change 1
- Bullet point describing change 2

Designed with ❤️ by oO. Coded with ✨ by Gemini
Co-authored-by: Gemini <gemini@google.com>
```

- **Types**: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

## 2. Core Development Principles

### Error Handling & Responses
- **Server-Side:** All handlers must throw exceptions on failure. The core server will catch and format the error.
- **Plugin-Side:** All plugin operations should be wrapped in a `try...catch` block to prevent the entire plugin from crashing.
- **Tool Responses:** All successful tool calls should return a YAML-formatted string for clear, structured output.
- **JSON-RPC:** When handling errors for JSON-RPC, always use `error.toString()` to provide the most informative message.
- **Debug Logging**: Use `debugLog()` from `src/utils/debug-log.ts` for troubleshooting - it safely ignores errors to avoid breaking JSON-RPC communication.

### Code Standards & Quality
- **Existing Patterns First:** Before creating new patterns, always analyze and follow existing ones. The `UnifiedHandler` is the primary pattern for all server-side tool handlers.
- **Figma API Validation:** Before implementing a new Figma API call, verify its existence and syntax in the official [Figma Plugin API documentation](https://www.figma.com/plugin-docs/).
- **Testing:** All new features or bug fixes must be accompanied by corresponding integration tests. Run `npm test` before submitting any changes.
- **Bug-Driven Testing**: Every bug found MUST have a corresponding test added to prevent regression.
- **Version Management**: Never hardcode version numbers - use dynamic injection from package.json
  - Server: Uses package.json import for VERSION constant
  - Plugin: Uses `PLUGIN_VERSION` global constant injected during build
  - UI: Uses `{{VERSION}}` template placeholder replaced during build

### Figma API Critical Rules
- **Style IDs**: Always clean trailing commas with `.replace(/,$/, '')`
- **Style Operations**: Search local collections, don't use direct API calls
- **Text Styles**: Clean style IDs before `setTextStyleIdAsync()`
- **Property Modification**: For object/array properties (effects, fills, strokes), ALWAYS use the clone-modify-assign pattern.
  - Clone entire property array/object
  - Modify the cloned data
  - Assign entire new array/object back to property
  - Use `FigmaPropertyManager` utility in `figma-plugin/src/utils/figma-property-utils.ts`
  - Reference: https://www.figma.com/plugin-docs/editing-properties/

## 3. API & Tool Design

### MCP Tool Design
- **Flat Parameters:** Tool input schemas should prefer flat parameters. Do not nest objects in the user-facing tool schema unless absolutely necessary.
- **Bulk Operations:** All tools that operate on nodes, styles, or other Figma objects should support bulk operations by accepting arrays for relevant parameters.
- **Consistency:** Follow the patterns established in the `figma_nodes` and `figma_plugin_status` tools.

## 4. Implementation Standards

### Server-Side Handler Pattern (`UnifiedHandler`)
All server-side handlers in `/src/handlers/` must use the `UnifiedHandler` utility. This ensures consistent parameter parsing, bulk operation handling, and error reporting.

```typescript
// Example from /src/handlers/node-handler.ts
export class NodeHandler implements ToolHandler {
  private unifiedHandler: UnifiedHandler;

  constructor(sendToPluginFn: (request: any) => Promise<any>) {
    this.unifiedHandler = new UnifiedHandler(sendToPluginFn);
  }

  async handle(toolName: string, args: any): Promise<any> {
    const config: UnifiedHandlerConfig = {
      toolName: 'figma_nodes',
      bulkParams: [/* array of bulk-capable params */],
      paramConfigs: { /* parameter types and validation */ },
      pluginMessageType: 'MANAGE_NODES', // Single message type for the plugin
      schema: UnifiedNodeOperationsSchema,
    };

    return this.unifiedHandler.handle(args, config);
  }
}
```

### Plugin-Side Operation Pattern
All plugin-side logic in `/figma-plugin/src/operations/` should be consolidated into a single file per tool. This file should export a function that takes a payload and uses a `switch` statement to call the appropriate Figma API function.

```typescript
// Example for a hypothetical /figma-plugin/src/operations/manage-styles.ts
export async function manageStyles(payload: any): Promise<any> {
  switch (payload.operation) {
    case 'create':
      // ... logic to create a style
      break;
    case 'delete':
      // ... logic to delete a style
      break;
    // ... other cases
  }
}
```

### Figma Property Management
```typescript
// CORRECT: Use FigmaPropertyManager for array/object properties
import { modifyEffects } from '../utils/figma-property-utils.js';

modifyEffects(target, manager => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oO/figma-mcp-write-server](https://github.com/oO/figma-mcp-write-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
