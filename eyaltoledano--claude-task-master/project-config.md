---
trigger: always_on
description: Guidelines for implementing task management operations
---

# Task Management Guidelines

## Tagged Task Lists System

Task Master now uses a **tagged task lists system** for multi-context task management:

- **Data Structure**: Tasks are organized into separate contexts (tags) within `tasks.json`
- **Legacy Format**: `{"tasks": [...]}`
- **Tagged Format**: `{"master": {"tasks": [...]}, "feature-branch": {"tasks": [...]}}`
- **Silent Migration**: Legacy format automatically converts to tagged format on first use
- **Tag Resolution**: Core functions receive legacy format for 100% backward compatibility
- **Default Tag**: "master" is used for all existing and new tasks unless otherwise specified

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

- **Tagged Data Structure**:
  - ✅ DO: Access tasks through tag resolution layer
  - ✅ DO: Use `getTasksForTag(data, tagName)` to retrieve tasks for a specific tag
  - ✅ DO: Use `setTasksForTag(data, tagName, tasks)` to update tasks for a specific tag
  - ❌ DON'T: Directly manipulate the tagged structure in core functions

  ```javascript
  // ✅ DO: Use tag resolution functions
  const tasksData = readJSON(tasksPath);
  const currentTag = getCurrentTag() || 'master';
  const tasks = getTasksForTag(tasksData, currentTag);
  
  // Manipulate tasks as normal...
  
  // Save back to the tagged structure
  setTasksForTag(tasksData, currentTag, tasks);
  writeJSON(tasksPath, tasksData);
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
  - ✅ DO: Assign unique sequential IDs to tasks within each tag context
  - ✅ DO: Calculate the next ID based on existing tasks in the current tag
  - ❌ DON'T: Hardcode or reuse IDs within the same tag

  ```javascript
  // ✅ DO: Calculate the next available ID within the current tag
  const tasksData = readJSON(tasksPath);
  const currentTag = getCurrentTag() || 'master';
  const tasks = getTasksForTag(tasksData, currentTag);
  const highestId = Math.max(...tasks.map(t => t.id));
  const nextTaskId = highestId + 1;
  ```

- **PRD Parsing**:
  - ✅ DO: Extract tasks from PRD documents using AI
  - ✅ DO: Create tasks in the current tag context (defaults to "master")
  - ✅ DO: Provide clear prompts to guide AI task generation
  - ✅ DO: Validate and clean up AI-generated tasks

  ```javascript
  // ✅ DO: Parse into current tag context
  const tasksData = readJSON(tasksPath) || {};
  const currentTag = getCurrentTag() || 'master';
  
  // Parse tasks and add to current tag
  const newTasks = await parseTasksFromPRD(prdContent);
  setTasksForTag(tasksData, currentTag, newTasks);
  writeJSON(tasksPath, tasksData);
  ```

## Task Updates and Modifications

- **Status Management**:
  - ✅ DO: Provide functions for updating task status within current tag context
  - ✅ DO: Handle both individual tasks and subtasks
  - ✅ DO: Consider subtask status when updating parent tasks

  ```javascript
  // ✅ DO: Handle status updates within tagged context
  async function setTaskStatus(tasksPath, taskIdInput, newStatus) {
    const tasksData = readJSON(tasksPath);
    const currentTag = getCurrentTag() || 'master';
    const tasks = getTasksForTag(tasksData, currentTag);
    
    // Check if it's a subtask (e.g., "1.2")
    if (taskIdInput.includes('.')) {
      const [parentId, subtaskId] = taskIdInput.split('.').map(id => parseInt(id, 10));
      
      // Find the parent task and subtask
      const parentTask = tasks.find(t => t.id === parentId);
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
      const task = tasks.find(t => t.id === parseInt(taskIdInput, 10));
      task.status = newStatus;
      
      // If marking as done, also mark subtasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
