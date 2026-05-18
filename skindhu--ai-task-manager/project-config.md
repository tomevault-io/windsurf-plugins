---
trigger: always_on
description: Guidelines for implementing task management operations
---


# Task Management Guidelines

## Task Structure Standards

- **Core Task Properties**:
  - ✅ DO: Include all required properties in each task object
  - ✅ DO: Provide default values for optional properties
  - ❌ DON'T: Add extra properties that aren't in the standard schema

  ```javascript
  // ✅ DO: Follow this structure for task objects
  const task = {
    id: nextId,
    title: "Task title",
    description: "Brief task description",
    status: "pending", // "pending", "in-progress", "done", etc.
    dependencies: [], // Array of task IDs
    priority: "medium", // "high", "medium", "low"
    details: "Detailed implementation instructions",
    testStrategy: "Verification approach",
    subtasks: [] // Array of subtask objects
  };
  ```

- **Subtask Structure**:
  - ✅ DO: Use consistent properties across subtasks
  - ✅ DO: Maintain simple numeric IDs within parent tasks
  - ❌ DON'T: Duplicate parent task properties in subtasks

  ```javascript
  // ✅ DO: Structure subtasks consistently
  const subtask = {
    id: nextSubtaskId, // Simple numeric ID, unique within the parent task
    title: "Subtask title",
    description: "Brief subtask description",
    status: "pending",
    dependencies: [], // Can include numeric IDs (other subtasks) or full task IDs
    details: "Detailed implementation instructions"
  };
  ```

## Task Creation and Parsing

- **ID Management**:
  - ✅ DO: Assign unique sequential IDs to tasks
  - ✅ DO: Calculate the next ID based on existing tasks
  - ❌ DON'T: Hardcode or reuse IDs

  ```javascript
  // ✅ DO: Calculate the next available ID
  const highestId = Math.max(...data.tasks.map(t => t.id));
  const nextTaskId = highestId + 1;
  ```

- **PRD Parsing**:
  - ✅ DO: Extract tasks from PRD documents using AI
  - ✅ DO: Provide clear prompts to guide AI task generation
  - ✅ DO: Validate and clean up AI-generated tasks

  ```javascript
  // ✅ DO: Validate AI responses
  try {
    // Parse the JSON response
    taskData = JSON.parse(jsonContent);
    
    // Check that we have the required fields
    if (!taskData.title || !taskData.description) {
      throw new Error("Missing required fields in the generated task");
    }
  } catch (error) {
    log('error', "Failed to parse AI's response as valid task JSON:", error);
    process.exit(1);
  }
  ```

## Task Updates and Modifications

- **Status Management**:
  - ✅ DO: Provide functions for updating task status
  - ✅ DO: Handle both individual tasks and subtasks
  - ✅ DO: Consider subtask status when updating parent tasks

  ```javascript
  // ✅ DO: Handle status updates for both tasks and subtasks
  async function setTaskStatus(tasksPath, taskIdInput, newStatus) {
    // Check if it's a subtask (e.g., "1.2")
    if (taskIdInput.includes('.')) {
      const [parentId, subtaskId] = taskIdInput.split('.').map(id => parseInt(id, 10));
      
      // Find the parent task and subtask
      const parentTask = data.tasks.find(t => t.id === parentId);
      const subtask = parentTask.subtasks.find(st => st.id === subtaskId);
      
      // Update subtask status
      subtask.status = newStatus;
      
      // Check if all subtasks are done
      if (newStatus === 'done') {
        const allSubtasksDone = parentTask.subtasks.every(st => st.status === 'done');
        if (allSubtasksDone) {
          // Suggest updating parent task
        }
      }
    } else {
      // Handle regular task
      const task = data.tasks.find(t => t.id === parseInt(taskIdInput, 10));
      task.status = newStatus;
      
      // If marking as done, also mark subtasks
      if (newStatus === 'done' && task.subtasks && task.subtasks.length > 0) {
        task.subtasks.forEach(subtask => {
          subtask.status = newStatus;
        });
      }
    }
  }
  ```

- **Task Expansion**:
  - ✅ DO: Use AI to generate detailed subtasks
  - ✅ DO: Consider complexity analysis for subtask counts
  - ✅ DO: Ensure proper IDs for newly created subtasks

  ```javascript
  // ✅ DO: Generate appropriate subtasks based on complexity
  if (taskAnalysis) {
    log('info', `Found complexity analysis for task ${taskId}: Score ${taskAnalysis.complexityScore}/10`);
    
    // Use recommended number of subtasks if available
    if (taskAnalysis.recommendedSubtasks && numSubtasks === CONFIG.defaultSubtasks) {
      numSubtasks = taskAnalysis.recommendedSubtasks;
      log('info', `Using recommended number of subtasks: ${numSubtasks}`);
    }
  }
  ```

## Task File Generation

- **File Formatting**:
  - ✅ DO: Use consistent formatting for task files
  - ✅ DO: Include all task properties in text files
  - ✅ DO: Format dependencies with status indicators

  ```javascript
  // ✅ DO: Use consistent file formatting
  let content = `# Task ID: ${task.id}\n`;
  content += `# Title: ${task.title}\n`;
  content += `# Status: ${task.status || 'pending'}\n`;
  
  // Format dependencies with their status
  if (task.dependencies && task.dependencies.length > 0) {
    content += `# Dependencies: ${formatDependenciesWithStatus(task.dependencies, data.tasks)}\n`;
  } else {
    content += '# Dependencies: None\n';
  }
  ```

- **Subtask Inclusion**:
  - ✅ DO: Include subtasks in parent task files
  - ✅ DO: Use consistent indentation for subtask sections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skindhu/AI-TASK-MANAGER](https://github.com/skindhu/AI-TASK-MANAGER) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
