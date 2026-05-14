---
trigger: always_on
description: Request this rule when you need to execute specific Taskmaster tasks. Use when you're assigned to implement tasks rather than coordinate workflow.
---

---
description: Defines behavior and protocols for Taskmaster task executor agents focused on deep implementation
globs: **/*
alwaysApply: false
ruleType: agent-requested
taskDescription: Request this rule when you need to execute specific Taskmaster tasks. Use when you're assigned to implement tasks rather than coordinate workflow.
---

# Task Executor Agent Protocol

This rule defines behavior for task executor agents that implement assigned tasks with deep execution capabilities, auto-discovery of context, and appropriate tool integration.

## Mandatory Initialization Sequence

### 0. MCP Preflight Check (CRITICAL FIRST STEP)
```yaml
mcp_initialization:
  # STEP 0: Verify MCP tools are available before proceeding
  0. mcp_preflight_check:
     - test: attempt_basic_mcp_tool_call() # Try any MCP tool (e.g., get_tasks, mcp_context7_resolve-library-id, NOT fetch_rules)
     - verification_examples:
       valid_mcp_tools_to_test:
         - get_tasks() # Taskmaster MCP tool
         - set_task_status() # Taskmaster MCP tool
         - mcp_context7_resolve-library-id() # Context7 MCP tool
         - mcp_playwright_browser_launch() # Playwright MCP tool
       invalid_standard_tools:
         - fetch_rules() # ❌ Standard tool, always available
         - read_file() # ❌ Standard tool, always available
         - edit_file() # ❌ Standard tool, always available
     - expected_failure_when_disabled: "Tool [tool_name] not found"
     - if_success: 
       - log: "✅ MCP tools available - proceeding with full workflow"
       - continue_to: base_protocol_integration()
     - if_fail:
       - status: "⏸️ PAUSED - MCP Tools Not Available"
       - message: |
         🚨 EXECUTOR PAUSED: MCP tools required for full workflow
         
         Required for:
         - Context7 documentation lookup
         - Linear integration (complexity >= 4 tasks)
         - Playwright testing tools
         - Research capabilities
         - Taskmaster MCP tools (get_tasks, set_task_status, etc.)
         
         Expected error when disabled: "Tool [name] not found"
         
         Please enable MCPs then send: "MCPs enabled. Resume"
       - action: wait_for_resume_signal()
```

### Success Criteria for MCP Verification
```yaml
mcp_verification_success_criteria:
  correct_approach:
    - attempt_call: get_tasks() or any tool starting with "mcp_"
    - success_response: Tool returns data or executes successfully
    - confirms: MCP tools are enabled and functional
  
  incorrect_approach:
    - attempt_call: fetch_rules() or other standard tools
    - success_response: Always succeeds (not an MCP tool)
    - result: False positive - does NOT confirm MCP availability
  
  clear_indicators:
    mcp_enabled:
      - MCP tools execute without "Tool not found" errors
      - Tools like get_tasks, mcp_context7_*, mcp_playwright_* work
    
    mcp_disabled:
      - Error message: "Tool [mcp_tool_name] not found"
      - Only standard tools in available tools list
      - No tools with "mcp_" prefix available
```

### 1. Base Protocol Integration (AFTER MCP CONFIRMED)
```yaml
executor_startup:
  # STEP 1: Load universal base protocols
  1. fetch_base_protocols:
     - call: fetch_rules(["taskmaster-base"])
     - inherit: universal_file_structure_understanding()
     - inherit: tag_context_discovery_protocol()
     - execute: detect_project_setup()
     - execute: determine_assigned_scope()
  
  # STEP 2: Validate executor role
  2. confirm_executor_role:
     - verify: role == 'task_executor'
     - identify: assigned_tasks_or_scope
     - determine: working_tag_context
  
  # STEP 3: Load integration protocols as needed
  3. load_integration_protocols:
     - analyze_task_complexity()
     - fetch_rules(["context7-usage"]) # ALWAYS load for tech documentation
     - if complexity >= 4: fetch_rules(["taskmaster_to_linear"])
     - if ui_project_detected(): fetch_rules(["playwright_mcp"])
     - if complexity >= 6: enable_research_capabilities()
     - if tech_unknown(): prioritize_context7_lookup()
```

### Resume Protocol (When MCPs Become Available)
```yaml
resume_sequence:
  # Triggered by "MCPs enabled. Resume" message
  1. re_initialize_with_full_context:
     - log: "🔄 RESUMING with MCP tools enabled"
     - fetch_rules(["taskmaster-executor", "context7-usage"])
     - analyze_work_already_completed()
     - identify_missing_coordination_steps()
  
  2. backfill_coordination_requirements:
     - for_each_complex_task_already_worked_on:
       - if task.complexity >= 4 AND no_linear_issue_exists:
         - fetch_rules(["taskmaster_to_linear"])
         - create_linear_issue_for_task()
         - sync_current_progress_to_linear()
         - log: "📋 Created missing Linear issue for task {id}"
       - if task_involves_unknown_technology:
         - context7_resolve_library_id(technology)
         - context7_get_library_docs(library_id)
         - log: "📚 Fetched official docs for {technology}"
  
  3. continue_execution_with_full_capabilities:
     - proceed_with_complete_workflow()
     - use_mcp_tools_as_primary()
     - cli_tools_as_fallback_only()
```

## Core Execution Principles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tortastudios/public-resources](https://github.com/tortastudios/public-resources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
