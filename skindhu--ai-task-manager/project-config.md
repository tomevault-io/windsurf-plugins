---
trigger: always_on
description: Describes the high-level architecture of the Task Master CLI application.
---


# Application Architecture Overview

- **Modular Structure**: The Task Master CLI is built using a modular architecture, with distinct modules responsible for different aspects of the application. This promotes separation of concerns, maintainability, and testability.

- **Main Modules and Responsibilities**:

  - **[`commands.js`](mdc:scripts/modules/commands.js): Command Handling**
    - **Purpose**: Defines and registers all CLI commands using Commander.js.
    - **Responsibilities**:
      - Parses command-line arguments and options.
      - Invokes appropriate functions from other modules to execute commands.
      - Handles user input and output related to command execution.
      - Implements input validation and error handling for CLI commands.
    - **Key Components**:
      - `programInstance` (Commander.js `Command` instance): Manages command definitions.
      - `registerCommands(programInstance)`: Function to register all application commands.
      - Command action handlers: Functions executed when a specific command is invoked.

  - **[`task-manager.js`](mdc:scripts/modules/task-manager.js): Task Data Management**
    - **Purpose**: Manages task data, including loading, saving, creating, updating, deleting, and querying tasks.
    - **Responsibilities**:
      - Reads and writes task data to `tasks.json` file.
      - Implements functions for task CRUD operations (Create, Read, Update, Delete).
      - Handles task parsing from PRD documents using AI.
      - Manages task expansion and subtask generation.
      - Updates task statuses and properties.
      - Implements task listing and display logic.
      - Performs task complexity analysis using AI.
    - **Key Functions**:
      - `readTasks(tasksPath)` / `writeTasks(tasksPath, tasksData)`: Load and save task data.
      - `parsePRD(prdFilePath, outputPath, numTasks)`: Parses PRD document to create tasks.
      - `expandTask(taskId, numSubtasks, useResearch, prompt, force)`: Expands a task into subtasks.
      - `setTaskStatus(tasksPath, taskIdInput, newStatus)`: Updates task status.
      - `listTasks(tasksPath, statusFilter, withSubtasks)`: Lists tasks with filtering and subtask display options.
      - `analyzeComplexity(tasksPath, reportPath, useResearch, thresholdScore)`: Analyzes task complexity.

  - **[`dependency-manager.js`](mdc:scripts/modules/dependency-manager.js): Dependency Management**
    - **Purpose**: Manages task dependencies, including adding, removing, validating, and fixing dependency relationships.
    - **Responsibilities**:
      - Adds and removes task dependencies.
      - Validates dependency relationships to prevent circular dependencies and invalid references.
      - Fixes invalid dependencies by removing non-existent or self-referential dependencies.
      - Provides functions to check for circular dependencies.
    - **Key Functions**:
      - `addDependency(tasksPath, taskId, dependencyId)`: Adds a dependency between tasks.
      - `removeDependency(tasksPath, taskId, dependencyId)`: Removes a dependency.
      - `validateDependencies(tasksPath)`: Validates task dependencies.
      - `fixDependencies(tasksPath)`: Fixes invalid task dependencies.
      - `isCircularDependency(tasks, taskId, dependencyChain)`: Detects circular dependencies.

  - **[`ui.js`](mdc:scripts/modules/ui.js): User Interface Components**
    - **Purpose**: Handles all user interface elements, including displaying information, formatting output, and providing user feedback.
    - **Responsibilities**:
      - Displays task lists, task details, and command outputs in a formatted way.
      - Uses `chalk` for colored output and `boxen` for boxed messages.
      - Implements table display using `cli-table3`.
      - Shows loading indicators using `ora`.
      - Provides helper functions for status formatting, dependency display, and progress reporting.
      - Suggests next actions to the user after command execution.
    - **Key Functions**:
      - `displayTaskList(tasks, statusFilter, withSubtasks)`: Displays a list of tasks in a table.
      - `displayTaskDetails(task)`: Displays detailed information for a single task.
      - `displayComplexityReport(reportPath)`: Displays the task complexity report.
      - `startLoadingIndicator(message)` / `stopLoadingIndicator(indicator)`: Manages loading indicators.
      - `getStatusWithColor(status)`: Returns status string with color formatting.
      - `formatDependenciesWithStatus(dependencies, allTasks, inTable)`: Formats dependency list with status indicators.

  - **[`ai-services.js`](mdc:scripts/modules/ai-services.js) (Conceptual): AI Integration**
    - **Purpose**:  Abstracts interactions with AI models (like Anthropic Claude and Perplexity AI) for various features. *Note: This module might be implicitly implemented within `task-manager.js` and `utils.js` or could be explicitly created for better organization as the project evolves.*
    - **Responsibilities**:
      - Handles API calls to AI services.
      - Manages prompts and parameters for AI requests.
      - Parses AI responses and extracts relevant information.
      - Implements logic for task complexity analysis, task expansion, and PRD parsing using AI.
    - **Potential Functions**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skindhu/AI-TASK-MANAGER](https://github.com/skindhu/AI-TASK-MANAGER) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
