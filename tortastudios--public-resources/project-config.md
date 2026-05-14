---
trigger: always_on
description: Request this rule when you need to convert Taskmaster tasks into Linear issues, maintaining proper relationships, preventing duplicates, and ensuring consistent formatting and structure.
---

---
description: Guidelines for creating Linear issues from Taskmaster tasks with proper structure and relationships
globs: .taskmaster/tasks/*.txt
alwaysApply: false
ruleType: agent-requested
taskDescription: Request this rule when you need to convert Taskmaster tasks into Linear issues, maintaining proper relationships, preventing duplicates, and ensuring consistent formatting and structure.
---
# Universal Taskmaster → Linear Integration Protocol

This rule defines the standardized process for converting Taskmaster tasks into Linear issues across any development project, ensuring proper relationships, preventing duplicates, and maintaining consistent structure within tag-aware workflows.

## Core Integration Principles

- **Tag-Aware Conversion**: Respect tag contexts and project boundaries when creating Linear issues
- **Project Discovery**: Dynamically identify project names and structure for proper issue organization
- **Duplicate Prevention**: Check existing Linear issues before creation to avoid conflicts
- **Relationship Preservation**: Maintain parent-child subtask relationships in Linear
- **Consistent Structure**: Standardized issue creation with complete information across all projects
- **Status Synchronization**: Bidirectional sync between Taskmaster and Linear status updates
- **Cross-Project Coordination**: Handle dependencies across different projects and tags

## Mandatory Prerequisites

### 1. Tag Context and Project Discovery
```yaml
before_linear_operations:
  # STEP 1: Ensure proper tag context (from @taskmaster-base.mcp)
  1. verify_tag_context:
     - confirm_current_tag_is_appropriate()
     - discover_project_name_from_tag_and_structure()
     - validate_project_exists_in_monorepo()
  
  # STEP 2: Load Linear workspace context
  2. linear_workspace_discovery:
     - identify_linear_workspace_for_project()
     - detect_team_assignments()
     - map_project_to_linear_labels()
  
  # STEP 3: Validate permissions and access
  3. permission_validation:
     - verify_linear_api_access()
     - confirm_project_team_membership()
     - validate_issue_creation_permissions()
```

### 2. Pre-Conversion Analysis
```yaml
conversion_analysis:
  task_assessment:
    # Analyze task structure and requirements
    - determine_if_conversion_needed(task_complexity, scope)
    - identify_parent_child_relationships()
    - extract_cross_project_dependencies()
    - assess_linear_issue_type_requirements()
  
  duplicate_prevention:
    # Comprehensive duplicate checking
    - search_existing_linear_issues_by_title()
    - check_taskmaster_id_references_in_linear()
    - identify_potential_duplicate_patterns()
    - validate_unique_conversion_opportunity()
  
  project_context_mapping:
    # Map discovered project to Linear organization
    - map_discovered_project_to_linear_team()
    - determine_appropriate_linear_labels()
    - identify_milestone_or_project_associations()
    - assess_priority_level_mapping()
```

## Universal Conversion Strategies

### Strategy 1: Single Task Conversion (Complexity 1-3)
```yaml
simple_task_conversion:
  when: task.complexity <= 3 AND no_subtasks AND single_project_scope
  
  linear_issue_creation:
    title: "{Discovered Project}: {Task Title}"
    description: |
      **Project**: {dynamically_discovered_project_name}
      **Tag Context**: {current_tag}
      **Taskmaster ID**: {task_id}
      **Complexity**: {score}/10
      
      ## Description
      {task.description}
      
      ## Implementation Details
      {task.details}
      
      ## Test Strategy
      {task.testStrategy}
      
      ## Success Criteria
      - [ ] Implementation complete in {discovered_project_path}
      - [ ] Tests pass using appropriate testing framework
      - [ ] Task marked as 'done' in Taskmaster
    
    labels: ["{discovered_project}", "taskmaster-sync", "complexity-{score}"]
    team: "{mapped_team_from_project_discovery}"
    priority: "{mapped_priority_from_taskmaster}"
    
  post_creation:
    - update_taskmaster_with_linear_issue_id()
    - add_linear_url_to_task_details()
    - set_bidirectional_sync_metadata()
```

### Strategy 2: Parent-Child Task Conversion (Complexity 4-7)
```yaml
hierarchical_task_conversion:
  when: task.has_subtasks OR task.complexity >= 4
  
  parent_issue_creation:
    title: "{Discovered Project}: {Parent Task Title}"
    description: |
      **Project**: {dynamically_discovered_project_name}  
      **Tag Context**: {current_tag}
      **Taskmaster Parent ID**: {parent_task_id}
      **Complexity**: {score}/10
      **Subtask Count**: {subtask_count}
      
      ## Epic Description
      {parent_task.description}
      
      ## Implementation Scope
      {parent_task.details}
      
      ## Subtasks Overview
      {list_of_subtask_titles_and_ids}
      
      ## Success Criteria
      - [ ] All subtasks completed
      - [ ] Integration testing complete
      - [ ] Epic marked as 'done' in Taskmaster
    
    type: "Epic" # or "Feature" based on Linear workspace configuration
    labels: ["{discovered_project}", "taskmaster-epic", "complexity-{score}"]
  
  subtask_issue_creation:
    for_each_subtask:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tortastudios/public-resources](https://github.com/tortastudios/public-resources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
