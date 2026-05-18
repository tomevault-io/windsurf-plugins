---
trigger: always_on
description: Guidelines for integrating new features into the Task Master CLI
---


# Task Master Feature Integration Guidelines

## Feature Placement Decision Process

- **Identify Feature Type**:
  - **Data Manipulation**: Features that create, read, update, or delete tasks belong in [`task-manager.js`](mdc:scripts/modules/task-manager.js)
  - **Dependency Management**: Features that handle task relationships belong in [`dependency-manager.js`](mdc:scripts/modules/dependency-manager.js)
  - **User Interface**: Features that display information to users belong in [`ui.js`](mdc:scripts/modules/ui.js)
  - **AI Integration**: Features that use AI models belong in [`ai-services.js`](mdc:scripts/modules/ai-services.js)
  - **Cross-Cutting**: Features that don't fit one category may need components in multiple modules

- **Command-Line Interface**:
  - All new user-facing commands should be added to [`commands.js`](mdc:scripts/modules/commands.js)
  - Use consistent patterns for option naming and help text
  - Follow the Commander.js model for subcommand structure

## Implementation Pattern

The standard pattern for adding a feature follows this workflow:

1. **Core Logic**: Implement the business logic in the appropriate module
2. **UI Components**: Add any display functions to [`ui.js`](mdc:scripts/modules/ui.js)
3. **Command Integration**: Add the CLI command to [`commands.js`](mdc:scripts/modules/commands.js)
4. **Testing**: Write tests for all components of the feature (following [`tests.mdc`](mdc:.cursor/rules/tests.mdc))
5. **Configuration**: Update any configuration in [`utils.js`](mdc:scripts/modules/utils.js) if needed
6. **Documentation**: Update help text and documentation in [dev_workflow.mdc](mdc:scripts/modules/dev_workflow.mdc)

```javascript
// 1. CORE LOGIC: Add function to appropriate module (example in task-manager.js)
/**
 * Archives completed tasks to archive.json
 * @param {string} tasksPath - Path to the tasks.json file
 * @param {string} archivePath - Path to the archive.json file
 * @returns {number} Number of tasks archived
 */
async function archiveTasks(tasksPath, archivePath = 'tasks/archive.json') {
  // Implementation...
  return archivedCount;
}

// Export from the module
export {
  // ... existing exports ...
  archiveTasks,
};
```

```javascript
// 2. UI COMPONENTS: Add display function to ui.js
/**
 * Display archive operation results
 * @param {string} archivePath - Path to the archive file
 * @param {number} count - Number of tasks archived
 */
function displayArchiveResults(archivePath, count) {
  console.log(boxen(
    chalk.green(`Successfully archived ${count} tasks to ${archivePath}`),
    { padding: 1, borderColor: 'green', borderStyle: 'round' }
  ));
}

// Export from the module
export {
  // ... existing exports ...
  displayArchiveResults,
};
```

```javascript
// 3. COMMAND INTEGRATION: Add to commands.js
import { archiveTasks } from './task-manager.js';
import { displayArchiveResults } from './ui.js';

// In registerCommands function
programInstance
  .command('archive')
  .description('Archive completed tasks to separate file')
  .option('-f, --file <file>', 'Path to the tasks file', 'tasks/tasks.json')
  .option('-o, --output <file>', 'Archive output file', 'tasks/archive.json')
  .action(async (options) => {
    const tasksPath = options.file;
    const archivePath = options.output;

    console.log(chalk.blue(`Archiving completed tasks from ${tasksPath} to ${archivePath}...`));

    const archivedCount = await archiveTasks(tasksPath, archivePath);
    displayArchiveResults(archivePath, archivedCount);
  });
```

## Cross-Module Features

For features requiring components in multiple modules:

- ✅ **DO**: Create a clear unidirectional flow of dependencies
  ```javascript
  // In task-manager.js
  function analyzeTasksDifficulty(tasks) {
    // Implementation...
    return difficultyScores;
  }

  // In ui.js - depends on task-manager.js
  import { analyzeTasksDifficulty } from './task-manager.js';

  function displayDifficultyReport(tasks) {
    const scores = analyzeTasksDifficulty(tasks);
    // Render the scores...
  }
  ```

- ❌ **DON'T**: Create circular dependencies between modules
  ```javascript
  // In task-manager.js - depends on ui.js
  import { displayDifficultyReport } from './ui.js';

  function analyzeTasks() {
    // Implementation...
    displayDifficultyReport(tasks); // WRONG! Don't call UI functions from task-manager
  }

  // In ui.js - depends on task-manager.js
  import { analyzeTasks } from './task-manager.js';
  ```

## Command-Line Interface Standards

- **Naming Conventions**:
  - Use kebab-case for command names (`analyze-complexity`, not `analyzeComplexity`)
  - Use kebab-case for option names (`--output-format`, not `--outputFormat`)
  - Use the same option names across commands when they represent the same concept

- **Command Structure**:
  ```javascript
  programInstance
    .command('command-name')
    .description('Clear, concise description of what the command does')
    .option('-s, --short-option <value>', 'Option description', 'default value')
    .option('--long-option <value>', 'Option description')
    .action(async (options) => {
      // Command implementation
    });
  ```

## Utility Function Guidelines

When adding utilities to [`utils.js`](mdc:scripts/modules/utils.js):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skindhu/AI-TASK-MANAGER](https://github.com/skindhu/AI-TASK-MANAGER) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
