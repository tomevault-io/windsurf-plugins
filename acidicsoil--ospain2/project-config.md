---
trigger: always_on
description: Guidelines for implementing CLI commands using Commander.js
---


# Command-Line Interface Implementation Guidelines

**Note on Interaction Method:**

While this document details the implementation of Task Master's **CLI commands**, the **preferred method for interacting with Task Master in integrated environments (like Cursor) is through the MCP server tools**. 

- **Use MCP Tools First**: Always prefer using the MCP tools (e.g., `get_tasks`, `add_task`) when interacting programmatically or via an integrated tool. They offer better performance, structured data, and richer error handling. See [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc) for a comprehensive list of MCP tools and their corresponding CLI commands.
- **CLI as Fallback/User Interface**: The `task-master` CLI commands described here are primarily intended for:
    - Direct user interaction in the terminal.
    - A fallback mechanism if the MCP server is unavailable or a specific functionality is not exposed via an MCP tool.
- **Implementation Context**: This document (`commands.mdc`) focuses on the standards for *implementing* the CLI commands using Commander.js within the [`commands.js`](mdc:scripts/modules/commands.js) module.

## Command Structure Standards

- **Basic Command Template**:
  ```javascript
  // ✅ DO: Follow this structure for all commands
  programInstance
    .command('command-name')
    .description('Clear, concise description of what the command does')
    .option('-o, --option <value>', 'Option description', 'default value')
    .option('--long-option <value>', 'Option description')
    .action(async (options) => {
      // Command implementation
    });
  ```

- **Command Handler Organization**:
  - ✅ DO: Keep action handlers concise and focused
  - ✅ DO: Extract core functionality to appropriate modules
  - ✅ DO: Have the action handler import and call the relevant functions from core modules, like `task-manager.js` or `init.js`, passing the parsed `options`.
  - ✅ DO: Perform basic parameter validation, such as checking for required options, within the action handler or at the start of the called core function.
  - ❌ DON'T: Implement business logic in command handlers

## Best Practices for Removal/Delete Commands

When implementing commands that delete or remove data (like `remove-task` or `remove-subtask`), follow these specific guidelines:

- **Confirmation Prompts**:
  - ✅ **DO**: Include a confirmation prompt by default for destructive operations
  - ✅ **DO**: Provide a `--yes` or `-y` flag to skip confirmation, useful for scripting or automation
  - ✅ **DO**: Show what will be deleted in the confirmation message
  - ❌ **DON'T**: Perform destructive operations without user confirmation unless explicitly overridden

  ```javascript
  // ✅ DO: Include confirmation for destructive operations
  programInstance
    .command('remove-task')
    .description('Remove a task or subtask permanently')
    .option('-i, --id <id>', 'ID of the task to remove')
    .option('-y, --yes', 'Skip confirmation prompt', false)
    .action(async (options) => {
      // Validation code...
      
      if (!options.yes) {
        const confirm = await inquirer.prompt([{
          type: 'confirm',
          name: 'proceed',
          message: `Are you sure you want to permanently delete task ${taskId}? This cannot be undone.`,
          default: false
        }]);
        
        if (!confirm.proceed) {
          console.log(chalk.yellow('Operation cancelled.'));
          return;
        }
      }
      
      // Proceed with removal...
    });
  ```

- **File Path Handling**:
  - ✅ **DO**: Use `path.join()` to construct file paths
  - ✅ **DO**: Follow established naming conventions for tasks, like `task_001.txt`
  - ✅ **DO**: Check if files exist before attempting to delete them
  - ✅ **DO**: Handle file deletion errors gracefully
  - ❌ **DON'T**: Construct paths with string concatenation

  ```javascript
  // ✅ DO: Properly construct file paths
  const taskFilePath = path.join(
    path.dirname(tasksPath),
    `task_${taskId.toString().padStart(3, '0')}.txt`
  );
  
  // ✅ DO: Check existence before deletion
  if (fs.existsSync(taskFilePath)) {
    try {
      fs.unlinkSync(taskFilePath);
      console.log(chalk.green(`Task file deleted: ${taskFilePath}`));
    } catch (error) {
      console.warn(chalk.yellow(`Could not delete task file: ${error.message}`));
    }
  }
  ```

- **Clean Up References**:
  - ✅ **DO**: Clean up references to the deleted item in other parts of the data
  - ✅ **DO**: Handle both direct and indirect references
  - ✅ **DO**: Explain what related data is being updated
  - ❌ **DON'T**: Leave dangling references

  ```javascript
  // ✅ DO: Clean up references when deleting items
  console.log(chalk.blue('Cleaning up task dependencies...'));
  let referencesRemoved = 0;
  
  // Update dependencies in other tasks
  data.tasks.forEach(task => {
    if (task.dependencies && task.dependencies.includes(taskId)) {
      task.dependencies = task.dependencies.filter(depId => depId !== taskId);
      referencesRemoved++;
    }
  });
  
  if (referencesRemoved > 0) {
    console.log(chalk.green(`Removed ${referencesRemoved} references to task ${taskId} from other tasks`));
  }
  ```

- **Task File Regeneration**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AcidicSoil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
