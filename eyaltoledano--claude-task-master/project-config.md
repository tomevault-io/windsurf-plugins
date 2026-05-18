---
trigger: always_on
description: Guidelines for implementing and interacting with the Task Master MCP Server
---

# Task Master MCP Server Guidelines

This document outlines the architecture and implementation patterns for the Task Master Model Context Protocol (MCP) server, designed for integration with tools like Cursor.

## Architecture Overview (See also: [`architecture.mdc`](mdc:.cursor/rules/architecture.mdc))

The MCP server acts as a bridge between external tools (like Cursor) and the core Task Master CLI logic. It leverages FastMCP for the server framework.

- **Flow**: `External Tool (Cursor)` <-> `FastMCP Server` <-> `MCP Tools` (`mcp-server/src/tools/*.js`) <-> `Core Logic Wrappers` (`mcp-server/src/core/direct-functions/*.js`, exported via `task-master-core.js`) <-> `Core Modules` (`scripts/modules/*.js`)
- **Goal**: Provide a performant and reliable way for external tools to interact with Task Master functionality without directly invoking the CLI for every operation.

## Direct Function Implementation Best Practices

When implementing a new direct function in `mcp-server/src/core/direct-functions/`, follow these critical guidelines:

1. **Verify Function Dependencies**:
   - ✅ **DO**: Check that all helper functions your direct function needs are properly exported from their source modules
   - ✅ **DO**: Import these dependencies explicitly at the top of your file
   - ❌ **DON'T**: Assume helper functions like `findTaskById` or `taskExists` are automatically available
   - **Example**:
     ```javascript
     // At top of direct-function file
     import { removeTask, findTaskById, taskExists } from '../../../../scripts/modules/task-manager.js';
     ```

2. **Parameter Verification and Completeness**:
   - ✅ **DO**: Verify the signature of core functions you're calling and ensure all required parameters are provided
   - ✅ **DO**: Pass explicit values for required parameters rather than relying on defaults
   - ✅ **DO**: Double-check parameter order against function definition
   - ❌ **DON'T**: Omit parameters assuming they have default values
   - **Example**:
     ```javascript
     // Correct parameter handling in direct function
     async function generateTaskFilesDirect(args, log) {
       const tasksPath = findTasksJsonPath(args, log);
       const outputDir = args.output || path.dirname(tasksPath);
       
       try {
         // Pass all required parameters
         const result = await generateTaskFiles(tasksPath, outputDir);
         return { success: true, data: result, fromCache: false };
       } catch (error) {
         // Error handling...
       }
     }
     ```

3. **Consistent File Path Handling**:
   - ✅ **DO**: Use `path.join()` instead of string concatenation for file paths
   - ✅ **DO**: Follow established file naming conventions (`task_001.txt` not `1.md`)
   - ✅ **DO**: Use `path.dirname()` and other path utilities for manipulating paths
   - ✅ **DO**: When paths relate to task files, follow the standard format: `task_${id.toString().padStart(3, '0')}.txt`
   - ❌ **DON'T**: Create custom file path handling logic that diverges from established patterns
   - **Example**:
     ```javascript
     // Correct file path handling
     const taskFilePath = path.join(
       path.dirname(tasksPath),
       `task_${taskId.toString().padStart(3, '0')}.txt`
     );
     ```

4. **Comprehensive Error Handling**:
   - ✅ **DO**: Wrap core function calls *and AI calls* in try/catch blocks
   - ✅ **DO**: Log errors with appropriate severity and context
   - ✅ **DO**: Return standardized error objects with code and message (`{ success: false, error: { code: '...', message: '...' } }`)
   - ✅ **DO**: Handle file system errors, AI client errors, AI processing errors, and core function errors distinctly with appropriate codes.
   - **Example**:
     ```javascript
     try {
       // Core function call or AI logic
     } catch (error) {
       log.error(`Failed to execute direct function logic: ${error.message}`);
       return {
         success: false,
         error: {
           code: error.code || 'DIRECT_FUNCTION_ERROR', // Use specific codes like AI_CLIENT_ERROR, etc.
           message: error.message,
           details: error.stack // Optional: Include stack in debug mode
         },
         fromCache: false // Ensure this is included if applicable
       };
     }
     ```

5. **Handling Logging Context (`mcpLog`)**:
   - **Requirement**: Core functions (like those in `task-manager.js`) may accept an `options` object containing an optional `mcpLog` property. If provided, the core function expects this object to have methods like `mcpLog.info(...)`, `mcpLog.error(...)`.
   - **Solution: The Logger Wrapper Pattern**: When calling a core function from a direct function, pass the `log` object provided by FastMCP *wrapped* in the standard `logWrapper` object. This ensures the core function receives a logger with the expected method structure.
     ```javascript
     // Standard logWrapper pattern within a Direct Function
     const logWrapper = {
       info: (message, ...args) => log.info(message, ...args),
       warn: (message, ...args) => log.warn(message, ...args),
       error: (message, ...args) => log.error(message, ...args),
       debug: (message, ...args) => log.debug && log.debug(message, ...args),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
