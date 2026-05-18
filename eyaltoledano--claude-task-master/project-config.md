---
trigger: always_on
description: Guidelines for integrating AI usage telemetry across Task Master.
---


# AI Usage Telemetry Integration

This document outlines the standard pattern for capturing, propagating, and handling AI usage telemetry data (cost, tokens, model, etc.) across the Task Master stack. This ensures consistent telemetry for both CLI and MCP interactions.

## Overview

Telemetry data is generated within the unified AI service layer ([`ai-services-unified.js`](mdc:scripts/modules/ai-services-unified.js)) and then passed upwards through the calling functions.

- **Data Source**: [`ai-services-unified.js`](mdc:scripts/modules/ai-services-unified.js) (specifically its `generateTextService`, `generateObjectService`, etc.) returns an object like `{ mainResult: AI_CALL_OUTPUT, telemetryData: TELEMETRY_OBJECT }`.
- **`telemetryData` Object Structure**:
  ```json
  {
    "timestamp": "ISO_STRING_DATE",
    "userId": "USER_ID_FROM_CONFIG",
    "commandName": "invoking_command_or_tool_name",
    "modelUsed": "ai_model_id",
    "providerName": "ai_provider_name",
    "inputTokens": NUMBER,
    "outputTokens": NUMBER,
    "totalTokens": NUMBER,
    "totalCost": NUMBER, // e.g., 0.012414
    "currency": "USD" // e.g., "USD"
  }
  ```

## Integration Pattern by Layer

The key principle is that each layer receives telemetry data from the layer below it (if applicable) and passes it to the layer above it, or handles it for display in the case of the CLI.

### 1. Core Logic Functions (e.g., in `scripts/modules/task-manager/`)

Functions in this layer that invoke AI services are responsible for handling the `telemetryData` they receive from [`ai-services-unified.js`](mdc:scripts/modules/ai-services-unified.js).

- **Actions**:
    1.  Call the appropriate AI service function (e.g., `generateObjectService`).
        -   Pass `commandName` (e.g., `add-task`, `expand-task`) and `outputType` (e.g., `cli` or `mcp`) in the `params` object to the AI service. The `outputType` can be derived from context (e.g., presence of `mcpLog`).
    2.  The AI service returns an object, e.g., `aiServiceResponse = { mainResult: {/*AI output*/}, telemetryData: {/*telemetry data*/} }`.
    3.  Extract `aiServiceResponse.mainResult` for the core processing.
    4.  **Must return an object that includes `aiServiceResponse.telemetryData`**.
        Example: `return { operationSpecificData: /*...*/, telemetryData: aiServiceResponse.telemetryData };`

- **CLI Output Handling (If Applicable)**:
    -   If the core function also handles CLI output (e.g., it has an `outputFormat` parameter that can be `'text'` or `'cli'`):
        1.  Check if `outputFormat === 'text'` (or `'cli'`).
        2.  If so, and if `aiServiceResponse.telemetryData` is available, call `displayAiUsageSummary(aiServiceResponse.telemetryData, 'cli')` from [`scripts/modules/ui.js`](mdc:scripts/modules/ui.js).
        - This ensures telemetry is displayed directly to CLI users after the main command output.

- **Example Snippet (Core Logic in `scripts/modules/task-manager/someAiAction.js`)**:
  ```javascript
  import { generateObjectService } from '../ai-services-unified.js';
  import { displayAiUsageSummary } from '../ui.js';

  async function performAiRelatedAction(params, context, outputFormat = 'text') {
    const { commandNameFromContext, /* other context vars */ } = context;
    let aiServiceResponse = null;

    try {
      aiServiceResponse = await generateObjectService({
        // ... other parameters for AI service ...
        commandName: commandNameFromContext || 'default-action-name',
        outputType: context.mcpLog ? 'mcp' : 'cli' // Derive outputType
      });

      const usefulAiOutput = aiServiceResponse.mainResult.object;
      // ... do work with usefulAiOutput ...

      if (outputFormat === 'text' && aiServiceResponse.telemetryData) {
        displayAiUsageSummary(aiServiceResponse.telemetryData, 'cli');
      }

      return {
        actionData: /* results of processing */,
        telemetryData: aiServiceResponse.telemetryData
      };
    } catch (error) {
      // ... handle error ...
      throw error;
    }
  }
  ```

### 2. Direct Function Wrappers (in `mcp-server/src/core/direct-functions/`)

These functions adapt core logic for the MCP server, ensuring structured responses.

- **Actions**:
    1.  Call the corresponding core logic function.
        -   Pass necessary context (e.g., `session`, `mcpLog`, `projectRoot`).
        -   Provide the `commandName` (typically derived from the MCP tool name) and `outputType: 'mcp'` in the context object passed to the core function.
        -   If the core function supports an `outputFormat` parameter, pass `'json'` to suppress CLI-specific UI.
    2.  The core logic function returns an object (e.g., `coreResult = { actionData: ..., telemetryData: ... }`).
    3.  Include `coreResult.telemetryData` as a field within the `data` object of the successful response returned by the direct function.

- **Example Snippet (Direct Function `someAiActionDirect.js`)**:
  ```javascript
  import { performAiRelatedAction } from '../../../../scripts/modules/task-manager/someAiAction.js'; // Core function
  import { createLogWrapper } from '../../tools/utils.js'; // MCP Log wrapper

  export async function someAiActionDirect(args, log, context = {}) {
    const { session } = context;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
