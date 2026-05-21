---
trigger: always_on
description: Comprehensive development workflow system for task tracking, testing, and version control
---

# Development Workflow System

Rule for managing the complete development lifecycle including task tracking, testing, and version control.

<rule>
name: development_workflow_system
filters:
  # Task management filters
  - type: event
    pattern: "task_start"
  - type: event
    pattern: "task_complete"
  - type: command
    pattern: "task"
  - type: event
    pattern: "user_request"
  - type: event 
    pattern: "implementation_complete"
  
  # Testing filters
  - type: event
    pattern: "implementation_start"
  - type: command
    pattern: "test"
  - type: file_change
    pattern: "src/*"
  - type: file_change
    pattern: "tests/*"
  - type: event
    pattern: "test_failure"
  
  # Git commit filters
  - type: event
    pattern: "build_success"
  - type: event
    pattern: "test_success"
  - type: event
    pattern: "file_save"
  - type: file_change
    pattern: "*"
  
  # README management filters
  - type: command
    pattern: "readme"
  - type: file_change
    pattern: "README.md"

actions:
  #
  # SECTION 1: TASK MANAGEMENT
  #
  
  - type: react
    conditions:
      - pattern: "task create|task_start"
    action: |
      # Create a new development task
      
      I'll create a new task with:
      - Unique task ID (TASK-YYYY-MM-DD-NN format)
      - Description section
      - Relevant specifications section (linked to `.cursor/specs/` managed by the specification_management rule)
      - Acceptance criteria section
      - Metadata (ID, start date, end date, state)
      - Learnings section for capturing implementation insights
      
      The task will be stored in `.cursor/tasks/` with an Open (📝) status
      and added to the task index in `.cursor/TASKS.md`

  - type: react
    conditions:
      - pattern: "task done|task complete|task_complete"
    action: |
      # Mark a task as complete
      
      I'll update the specified task:
      - Change state to Done (✅)
      - Set end date to current date
      - Extract learnings to a separate file if any exist
      - Mark associated specs as completed
      - Update the task index in `.cursor/TASKS.md`
      
      If the task contains valuable learnings, they'll be saved to 
      `.cursor/learnings/` with proper cross-references and managed by the knowledge_management rule

  - type: react
    conditions:
      - pattern: "task list|task_status"
    action: |
      # List all tasks with their status
      
      I'll generate a table showing:
      - Task ID
      - Current state (Open 📝, Active 🔄, or Done ✅)
      - Task description
      - Start date
      - End date (if completed)
      
      This provides a quick overview of all project tasks and their progress

  - type: react
    conditions:
      - pattern: "task start|task_active"
    action: |
      # Mark a task as active
      
      I'll update the specified task:
      - Change state from Open (📝) to Active (🔄)
      - Update the task index in `.cursor/TASKS.md`
      
      This indicates work has started on this task

  - type: react
    event: "user_request"
    conditions:
      - pattern: "implement|create|build|develop"
    action: |
      # Auto-create task when user requests implementation
      
      When you ask me to implement something, I'll automatically:
      - Create a new task with the implementation request as description
      - Look for related specifications to link to the task (using the specification_management rule)
      - Set the task state to Active (🔄)
      - Add acceptance criteria
      - Create a unique task ID
      - Add the task to the task index
      
      This ensures all implementation work is properly tracked
  
  #
  # SECTION 2: TESTING MANAGEMENT
  #
  
  - type: react
    event: "implementation_start"
    action: |
      # Create test files when implementation begins
      
      When implementation starts, I'll:
      
      1. Find the active task and its associated specifications
      2. For each specification, create appropriate test files:
         - Detect the project type (Node.js, Python, Rust, etc.)
         - Create test files in the proper location and format
         - Link tests to the specification requirements
         - Add placeholder tests for each requirement
         - Include references to the specs and task
      
      3. Consider project-specific testing patterns:
         - Use Jest for JavaScript/TypeScript
         - Use pytest for Python
         - Use Cargo test for Rust
         - Use JUnit for Java
      
      This ensures tests are created before implementation, supporting 
      test-driven development.

  - type: react
    conditions:
      - pattern: "test run|test execute"
    action: |
      # Run tests based on project type
      
      I'll execute tests for the project:
      
      1. Detect the project type:
         - Node.js (package.json) → npm test
         - Rust (Cargo.toml) → cargo test
         - Java (pom.xml) → mvn test
         - Python (requirements.txt/setup.py) → pytest or unittest
      
      2. Process test results:
         - If tests pass, update active task to reflect passing tests
         - If tests fail, create detailed failure report
         - Trigger appropriate success/failure events
      

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreRatzenberger/cursor-rules](https://github.com/AndreRatzenberger/cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
