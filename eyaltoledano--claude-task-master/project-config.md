---
trigger: always_on
description: enableSilentMode();
---

# Utility Function Guidelines

## General Principles

- **Function Scope**:
  - ✅ DO: Create utility functions that serve multiple modules
  - ✅ DO: Keep functions single-purpose and focused
  - ❌ DON'T: Include business logic in utility functions
  - ❌ DON'T: Create utilities with side effects

  ```javascript
  // ✅ DO: Create focused, reusable utilities
  /**
   * Truncates text to a specified length
   * @param {string} text - The text to truncate
   * @param {number} maxLength - The maximum length
   * @returns {string} The truncated text
   */
  function truncate(text, maxLength) {
    if (!text || text.length <= maxLength) {
      return text;
    }
    return text.slice(0, maxLength - 3) + '...';
  }
  ```

  ```javascript
  // ❌ DON'T: Add side effects to utilities
  function truncate(text, maxLength) {
    if (!text || text.length <= maxLength) {
      return text;
    }
    
    // Side effect - modifying global state or logging
    console.log(`Truncating text from ${text.length} to ${maxLength} chars`);
    
    return text.slice(0, maxLength - 3) + '...';
  }
  ```

- **Location**:
    - **Core CLI Utilities**: Place utilities used primarily by the core `task-master` CLI logic and command modules (`scripts/modules/*`) into [`scripts/modules/utils.js`](mdc:scripts/modules/utils.js).
    - **MCP Server Utilities**: Place utilities specifically designed to support the MCP server implementation into the appropriate subdirectories within `mcp-server/src/`.
        - Path/Core Logic Helpers: [`mcp-server/src/core/utils/`](mdc:mcp-server/src/core/utils) (e.g., `path-utils.js`).
        - Tool Execution/Response Helpers: [`mcp-server/src/tools/utils.js`](mdc:mcp-server/src/tools/utils.js).

## Documentation Standards

- **JSDoc Format**:
  - ✅ DO: Document all parameters and return values
  - ✅ DO: Include descriptions for complex logic
  - ✅ DO: Add examples for non-obvious usage
  - ❌ DON'T: Skip documentation for "simple" functions

  ```javascript
  // ✅ DO: Provide complete JSDoc documentation
  /**
   * Reads and parses a JSON file
   * @param {string} filepath - Path to the JSON file
   * @returns {Object|null} Parsed JSON data or null if error occurs
   */
  function readJSON(filepath) {
    try {
      const rawData = fs.readFileSync(filepath, 'utf8');
      return JSON.parse(rawData);
    } catch (error) {
      log('error', `Error reading JSON file ${filepath}:`, error.message);
      if (CONFIG.debug) {
        console.error(error);
      }
      return null;
    }
  }
  ```

## Configuration Management (via `config-manager.js`)

Taskmaster configuration (excluding API keys) is primarily managed through the `.taskmasterconfig` file located in the project root and accessed via getters in [`scripts/modules/config-manager.js`](mdc:scripts/modules/config-manager.js).

- **`.taskmasterconfig` File**:
  - ✅ DO: Use this JSON file to store settings like AI model selections (main, research, fallback), parameters (temperature, maxTokens), logging level, default priority/subtasks, etc.
  - ✅ DO: Manage this file using the `task-master models --setup` CLI command or the `models` MCP tool.
  - ✅ DO: Rely on [`config-manager.js`](mdc:scripts/modules/config-manager.js) to load this file (using the correct project root passed from MCP or found via CLI utils), merge with defaults, and provide validated settings.
  - ❌ DON'T: Store API keys in this file.
  - ❌ DON'T: Manually edit this file unless necessary.

- **Configuration Getters (`config-manager.js`)**:
  - ✅ DO: Import and use specific getters from `config-manager.js` (e.g., `getMainProvider()`, `getLogLevel()`, `getMainMaxTokens()`) to access configuration values *needed for application logic* (like `getDefaultSubtasks`).
  - ✅ DO: Pass the `explicitRoot` parameter to getters if calling from MCP direct functions to ensure the correct project's config is loaded.
  - ❌ DON'T: Call AI-specific getters (like `getMainModelId`, `getMainMaxTokens`) from core logic functions (`scripts/modules/task-manager/*`). Instead, pass the `role` to the unified AI service.
  - ❌ DON'T: Access configuration values directly from environment variables (except API keys).

- **API Key Handling (`utils.js` & `ai-services-unified.js`)**:
  - ✅ DO: Store API keys **only** in `.env` (for CLI, loaded by `dotenv` in `scripts/dev.js`) or `.cursor/mcp.json` (for MCP, accessed via `session.env`).
  - ✅ DO: Use `isApiKeySet(providerName, session)` from `config-manager.js` to check if a provider's key is available *before* potentially attempting an AI call if needed, but note the unified service performs its own internal check.
  - ✅ DO: Understand that the unified service layer (`ai-services-unified.js`) internally resolves API keys using `resolveEnvVariable(key, session)` from `utils.js`.

- **Error Handling**:
  - ✅ DO: Handle potential `ConfigurationError` if the `.taskmasterconfig` file is missing or invalid when accessed via `getConfig` (e.g., in `commands.js` or direct functions).

## Logging Utilities (in `scripts/modules/utils.js`)

- **Log Levels**:
  - ✅ DO: Support multiple log levels (debug, info, warn, error)
  - ✅ DO: Use appropriate icons for different log levels
  - ✅ DO: Respect the configured log level
  - ❌ DON'T: Add direct console.log calls outside the logging utility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
