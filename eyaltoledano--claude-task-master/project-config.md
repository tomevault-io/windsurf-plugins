---
trigger: always_on
description: Guidelines for integrating new features into the Task Master CLI
---

# Task Master Feature Integration Guidelines

## Feature Placement Decision Process

- **Identify Feature Type** (See [`architecture.mdc`](mdc:.cursor/rules/architecture.mdc) for module details):
  - **Data Manipulation**: Features that create, read, update, or delete tasks belong in [`task-manager.js`](mdc:scripts/modules/task-manager.js). Follow guidelines in [`tasks.mdc`](mdc:.cursor/rules/tasks.mdc).
  - **Dependency Management**: Features that handle task relationships belong in [`dependency-manager.js`](mdc:scripts/modules/dependency-manager.js). Follow guidelines in [`dependencies.mdc`](mdc:.cursor/rules/dependencies.mdc).
  - **User Interface**: Features that display information to users belong in [`ui.js`](mdc:scripts/modules/ui.js). Follow guidelines in [`ui.mdc`](mdc:.cursor/rules/ui.mdc).
  - **AI Integration**: Features that use AI models belong in [`ai-services.js`](mdc:scripts/modules/ai-services.js).
  - **Cross-Cutting**: Features that don't fit one category may need components in multiple modules

- **Command-Line Interface** (See [`commands.mdc`](mdc:.cursor/rules/commands.mdc)):
  - All new user-facing commands should be added to [`commands.js`](mdc:scripts/modules/commands.js)
  - Use consistent patterns for option naming and help text
  - Follow the Commander.js model for subcommand structure

## Implementation Pattern

The standard pattern for adding a feature follows this workflow:

1. **Core Logic**: Implement the business logic in the appropriate module (e.g., [`task-manager.js`](mdc:scripts/modules/task-manager.js)).
2. **Context Gathering (If Applicable)**: 
   - For AI-powered commands that benefit from project context, use the standardized context gathering patterns from [`context_gathering.mdc`](mdc:.cursor/rules/context_gathering.mdc).
   - Import `ContextGatherer` and `FuzzyTaskSearch` utilities for reusable context extraction.
   - Support multiple context types: tasks, files, custom text, project tree.
   - Implement detailed token breakdown display for transparency.
3. **AI Integration (If Applicable)**: 
   - Import necessary service functions (e.g., `generateTextService`, `streamTextService`) from [`ai-services-unified.js`](mdc:scripts/modules/ai-services-unified.js).
   - Prepare parameters (`role`, `session`, `systemPrompt`, `prompt`).
   - Call the service function.
   - Handle the response (direct text or stream object).
   - **Important**: Prefer `generateTextService` for calls sending large context (like stringified JSON) where incremental display is not needed. See [`ai_services.mdc`](mdc:.cursor/rules/ai_services.mdc) for detailed usage patterns and cautions.
4. **UI Components**: Add any display functions to [`ui.js`](mdc:scripts/modules/ui.js) following [`ui.mdc`](mdc:.cursor/rules/ui.mdc). Consider enhanced formatting with syntax highlighting for code blocks.
5. **Command Integration**: Add the CLI command to [`commands.js`](mdc:scripts/modules/commands.js) following [`commands.mdc`](mdc:.cursor/rules/commands.mdc).
6. **Testing**: Write tests for all components of the feature (following [`tests.mdc`](mdc:.cursor/rules/tests.mdc))
7. **Configuration**: Update configuration settings or add new ones in [`config-manager.js`](mdc:scripts/modules/config-manager.js) and ensure getters/setters are appropriate. Update documentation in [`utilities.mdc`](mdc:.cursor/rules/utilities.mdc) and [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc). Update the `.taskmasterconfig` structure if needed.
8. **Documentation**: Update help text and documentation in [`dev_workflow.mdc`](mdc:.cursor/rules/dev_workflow.mdc) and [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc).

## Critical Checklist for New Features

- **Comprehensive Function Exports**:
  - ✅ **DO**: Export **all core functions, helper functions (like `generateSubtaskPrompt`), and utility methods** needed by your new function or command from their respective modules.
  - ✅ **DO**: **Explicitly review the module's `export { ... }` block** at the bottom of the file to ensure every required dependency (even seemingly minor helpers like `findTaskById`, `taskExists`, specific prompt generators, AI call handlers, etc.) is included.
  - ❌ **DON'T**: Assume internal functions are already exported - **always verify**. A missing export will cause runtime errors (e.g., `ReferenceError: generateSubtaskPrompt is not defined`).
  - **Example**: If implementing a feature that checks task existence, ensure the helper function is in exports:
  ```javascript
  // At the bottom of your module file:
  export {
    // ... existing exports ...
    yourNewFunction,
    taskExists,  // Helper function used by yourNewFunction
    findTaskById, // Helper function used by yourNewFunction
    generateSubtaskPrompt, // Helper needed by expand/add features
    getSubtasksFromAI,     // Helper needed by expand/add features
  };
  ```

- **Parameter Completeness and Matching**:
  - ✅ **DO**: Pass all required parameters to functions you call within your implementation
  - ✅ **DO**: Check function signatures before implementing calls to them
  - ✅ **DO**: Verify that direct function parameters match their core function counterparts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
