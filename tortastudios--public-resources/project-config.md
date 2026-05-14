---
trigger: always_on
description: provides: ["file_structure", "tag_patterns", "discovery_protocols"]
---

---
description: Base initialization protocol for all Taskmaster agents
globs: 
  - ".taskmaster/**/*"
  - "**/tasks/**/*"
  - ".taskmasterconfig"
alwaysApply: false
ruleType: auto-attached
---

# Taskmaster Universal Base Protocol

This rule defines the initialization process for Taskmaster agents in any collaborative development environment.

## ⚠️ CRITICAL: Universal Shared File Structure

**All Taskmaster agents must understand the SHARED file structure with TAG SEPARATION:**

### **Universal Monorepo Collaboration Model**
```
SHARED TASKMASTER STRUCTURE (Committed to Git):

<PROJECT_ROOT> (any project root directory)
├── .taskmaster/                    ← SHARED Taskmaster setup
│   ├── config.json                ← Shared AI models, global settings
│   ├── tasks/tasks.json           ← SHARED task file with TAG CONTEXTS
│   │   └── Structure:
│   │       {
│   │         "tags": {
│   │           "{developer}-{feature}": { "tasks": [...], "metadata": {...} },
│   │           "{team}-{area}": { "tasks": [...], "metadata": {...} },
│   │           "shared-{infrastructure}": { "tasks": [...], "metadata": {...} }
│   │         },
│   │         "currentTag": "{context-dependent}"
│   │       }
│   ├── docs/                     ← Shared project requirements (PRDs)
│   └── templates/                ← Shared templates and examples
├── {project_structure}/           ← Any project structure (apps/, src/, lib/, etc.)
│   ├── {project_type}/           ← Web, mobile, API, desktop, etc.
│   └── {project_name}/           ← Actual project implementations
│       ├── src/                  ← Generated/modified code goes here
│       └── {package_file}        ← Project-specific dependencies
├── {workspace_indicator}         ← pnpm-workspace.yaml, lerna.json, nx.json, etc.
└── .git/                         ← Version control

UNIVERSAL PATTERNS:
✅ Single shared tasks.json - no file conflicts
✅ Tag-based contexts - personal vs shared work separation  
✅ MCP tools work for all team members
✅ Background agents can access committed structure
✅ Shared PRDs, templates, and configurations
✅ Flexible project structure support
```

### **Universal Tag Context Discovery Protocol**
**Run this BEFORE any Taskmaster operation:**
```python
def detect_universal_setup():
    """MANDATORY: Detect Taskmaster setup in any project structure"""
    root = detect_project_root()  # Works with any VCS root
    
    # Check for shared Taskmaster structure
    taskmaster_dir = os.path.join(root, '.taskmaster')
    if exists(os.path.join(taskmaster_dir, 'config.json')):
        tasks_file = os.path.join(taskmaster_dir, 'tasks', 'tasks.json')
        if exists(tasks_file):
            return 'collaborative', root, tasks_file
    
    # Check for legacy format  
    if exists(os.path.join(root, '.taskmasterconfig')):
        return 'legacy', root, None
        
    return None, root, None

def determine_universal_tag_context():
    """Determine appropriate tag context based on available information"""
    context_strategies = [
        'check_existing_current_tag',
        'infer_from_work_context', 
        'analyze_directory_patterns',
        'examine_conversation_context',
        'prompt_user_for_clarification'
    ]
    
    for strategy in context_strategies:
        result = execute_strategy(strategy)
        if result:
            return result
    
    return None  # Requires user clarification

def execute_strategy(strategy):
    """Execute specific context discovery strategy"""
    if strategy == 'check_existing_current_tag':
        return check_tasks_json_current_tag()
    elif strategy == 'infer_from_work_context':
        return infer_from_current_work()
    elif strategy == 'analyze_directory_patterns':
        return analyze_directory_structure()
    elif strategy == 'examine_conversation_context':
        return extract_context_from_conversation()
    elif strategy == 'prompt_user_for_clarification':
        return prompt_for_tag_context()

def determine_universal_role():
    """Determine agent role based on context and task assignment"""
    if has_specific_task_assignment():
        return 'task_agent'
    elif has_scope_specification():
        return 'scoped_orchestrator'
    else:
        return 'full_orchestrator'
```

## Agent Types (Universal)

1. **Orchestrator Agent** (Human-initiated)
   - Uses: fetch_rules(["taskmaster-orchestrator"]) for coordination protocols
   - Monitors shared tasks.json across all tags
   - Spawns task agents with proper tag context
   - Manages cross-tag dependencies
   - Coordinates workflows in any project structure

2. **Task Agent** (Orchestrator-spawned)
   - Uses: fetch_rules(["taskmaster-agent"]) for execution protocols
   - Works within specific tag context
   - Updates shared tasks.json with tag isolation
   - Reports back via Taskmaster and Linear
   - Respects tag boundaries

## Universal Tag-Aware Initialization

```python
def determine_role_and_context():
    # CRITICAL: Always start with discovery
    config_status, root_path, tasks_file = detect_universal_setup()
    
    if config_status is None:
        raise Exception("Taskmaster not initialized - user must run 'task-master init' first")
    
    # Get tag context for this session

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tortastudios/public-resources](https://github.com/tortastudios/public-resources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
