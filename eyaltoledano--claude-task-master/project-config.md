---
trigger: always_on
description: Guidelines for managing task dependencies and relationships
---


# Dependency Management Guidelines

## Dependency Structure Principles

- **Dependency References**:
  - ✅ DO: Represent task dependencies as arrays of task IDs
  - ✅ DO: Use numeric IDs for direct task references
  - ✅ DO: Use string IDs with dot notation (e.g., "1.2") for subtask references
  - ❌ DON'T: Mix reference types without proper conversion

  ```javascript
  // ✅ DO: Use consistent dependency formats
  // For main tasks
  task.dependencies = [1, 2, 3]; // Dependencies on other main tasks
  
  // For subtasks
  subtask.dependencies = [1, "3.2"]; // Dependency on main task 1 and subtask 2 of task 3
  ```

- **Subtask Dependencies**:
  - ✅ DO: Allow numeric subtask IDs to reference other subtasks within the same parent
  - ✅ DO: Convert between formats appropriately when needed
  - ❌ DON'T: Create circular dependencies between subtasks

  ```javascript
  // ✅ DO: Properly normalize subtask dependencies
  // When a subtask refers to another subtask in the same parent
  if (typeof depId === 'number' && depId < 100) {
    // It's likely a reference to another subtask in the same parent task
    const fullSubtaskId = `${parentId}.${depId}`;
    // Now use fullSubtaskId for validation
  }
  ```

## Dependency Validation

- **Existence Checking**:
  - ✅ DO: Validate that referenced tasks exist before adding dependencies
  - ✅ DO: Provide clear error messages for non-existent dependencies
  - ✅ DO: Remove references to non-existent tasks during validation

  ```javascript
  // ✅ DO: Check if the dependency exists before adding
  if (!taskExists(data.tasks, formattedDependencyId)) {
    log('error', `Dependency target ${formattedDependencyId} does not exist in tasks.json`);
    process.exit(1);
  }
  ```

- **Circular Dependency Prevention**:
  - ✅ DO: Check for circular dependencies before adding new relationships
  - ✅ DO: Use graph traversal algorithms (DFS) to detect cycles
  - ✅ DO: Provide clear error messages explaining the circular chain

  ```javascript
  // ✅ DO: Check for circular dependencies before adding
  const dependencyChain = [formattedTaskId];
  if (isCircularDependency(data.tasks, formattedDependencyId, dependencyChain)) {
    log('error', `Cannot add dependency ${formattedDependencyId} to task ${formattedTaskId} as it would create a circular dependency.`);
    process.exit(1);
  }
  ```

- **Self-Dependency Prevention**:
  - ✅ DO: Prevent tasks from depending on themselves
  - ✅ DO: Handle both direct and indirect self-dependencies

  ```javascript
  // ✅ DO: Prevent self-dependencies
  if (String(formattedTaskId) === String(formattedDependencyId)) {
    log('error', `Task ${formattedTaskId} cannot depend on itself.`);
    process.exit(1);
  }
  ```

## Dependency Modification

- **Adding Dependencies**:
  - ✅ DO: Format task and dependency IDs consistently
  - ✅ DO: Check for existing dependencies to prevent duplicates
  - ✅ DO: Sort dependencies for better readability

  ```javascript
  // ✅ DO: Format IDs consistently when adding dependencies
  const formattedTaskId = typeof taskId === 'string' && taskId.includes('.') 
    ? taskId : parseInt(taskId, 10);
  
  const formattedDependencyId = formatTaskId(dependencyId);
  ```

- **Removing Dependencies**:
  - ✅ DO: Check if the dependency exists before removing
  - ✅ DO: Handle different ID formats consistently
  - ✅ DO: Provide feedback about the removal result

  ```javascript
  // ✅ DO: Properly handle dependency removal
  const dependencyIndex = targetTask.dependencies.findIndex(dep => {
    // Convert both to strings for comparison
    let depStr = String(dep);
    
    // Handle relative subtask references
    if (typeof dep === 'number' && dep < 100 && isSubtask) {
      const [parentId] = formattedTaskId.split('.');
      depStr = `${parentId}.${dep}`;
    }
    
    return depStr === normalizedDependencyId;
  });
  
  if (dependencyIndex === -1) {
    log('info', `Task ${formattedTaskId} does not depend on ${formattedDependencyId}, no changes made.`);
    return;
  }
  
  // Remove the dependency
  targetTask.dependencies.splice(dependencyIndex, 1);
  ```

## Dependency Cleanup

- **Duplicate Removal**:
  - ✅ DO: Use Set objects to identify and remove duplicates
  - ✅ DO: Handle both numeric and string ID formats

  ```javascript
  // ✅ DO: Remove duplicate dependencies
  const uniqueDeps = new Set();
  const uniqueDependencies = task.dependencies.filter(depId => {
    // Convert to string for comparison to handle both numeric and string IDs
    const depIdStr = String(depId);
    if (uniqueDeps.has(depIdStr)) {
      log('warn', `Removing duplicate dependency from task ${task.id}: ${depId}`);
      return false;
    }
    uniqueDeps.add(depIdStr);
    return true;
  });
  ```

- **Invalid Reference Cleanup**:
  - ✅ DO: Check for and remove references to non-existent tasks
  - ✅ DO: Check for and remove self-references
  - ✅ DO: Track and report changes made during cleanup

  ```javascript
  // ✅ DO: Filter invalid task dependencies
  task.dependencies = task.dependencies.filter(depId => {
    const numericId = typeof depId === 'string' ? parseInt(depId, 10) : depId;
    if (!validTaskIds.has(numericId)) {
      log('warn', `Removing invalid task dependency from task ${task.id}: ${depId} (task does not exist)`);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
