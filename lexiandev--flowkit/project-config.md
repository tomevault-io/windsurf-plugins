---
trigger: always_on
description: FlowKit is a Godot 4 editor plugin for visual event-driven programming, inspired by Clickteam Fusion/Construct event sheets.
---

# FlowKit Development Guide

FlowKit is a Godot 4 editor plugin for visual event-driven programming, inspired by Clickteam Fusion/Construct event sheets.

## Architecture Overview

**Plugin Structure**: FlowKit is a dual-mode system with editor-time visual authoring and runtime execution:

- **Editor side** (`flowkit.gd`, `ui/`): Godot `@tool` plugin that adds a bottom panel for visual event editing
- **Runtime side** (`runtime/flowkit_engine.gd`, `runtime/flowkit_system.gd`): Autoloaded singletons that execute event sheets during gameplay
- **Registry system** (`registry.gd`): Auto-discovers and instantiates provider scripts at plugin load
- **Branch system** (`branches/`): Extensible branch providers (if, repeat, etc.) that control action execution flow
- **Generator system** (`generator.gd`): Automatically generates actions, conditions, and events from scene node types and their properties/methods/signals

**Event Sheet Model**: Scene-specific `.tres` resources saved to `saved/event_sheet/{scene_name}.tres`:

```
FKEventSheet
  └─ events: Array[FKEventBlock]
      ├─ event_id: String (e.g., "on_process")
      ├─ target_node: NodePath (node to poll event from)
      ├─ conditions: Array[FKConditionUnit] (all must pass)
      └─ actions: Array[FKActionUnit] (executed sequentially)
```

**Provider Pattern**: All events/conditions/actions/branches extend base classes (`FKEvent`, `FKCondition`, `FKAction`, `FKBranch`) implementing:

- `get_id()`: Unique identifier string
- `get_name()`: Display name for UI
- `get_supported_types()`: Array of compatible node class names (e.g., `["CharacterBody2D"]`) — not used by branches
- `get_inputs()`: Array of `{"name": String, "type": String}` dictionaries for parameters
- Execution method: `poll(node)`, `check(node, inputs)`, `execute(node, inputs)`, or `should_execute()`/`get_execution_count()`

**Branch Provider Pattern** (`FKBranch` base class in `resources/fk_branch.gd`):

Branches control conditional/repeated execution of actions within an event block. They are extensible providers stored in `branches/`. Each branch implements:

- `get_id()` → `String`: Unique identifier (e.g., `"if_branch"`, `"repeat"`)
- `get_name()` → `String`: Display name (e.g., `"If"`, `"Repeat"`)
- `get_description()` → `String`: Tooltip description
- `get_input_type()` → `String`: Either `"condition"` (uses FKCondition providers) or `"evaluation"` (uses expression inputs directly)
- `get_inputs()` → `Array[Dictionary]`: Input definitions for evaluation-type branches (e.g., `[{"name": "times", "type": "int"}]`)
- `get_type()` → `String`: Either `"single"` (standalone, no chaining) or `"chain"` (allows else-if/else blocks after it)
- `get_color()` → `Color`: Accent color for the branch in the editor UI (type label, icon). Override to customise per-provider.
- `should_execute(condition_result: bool, inputs: Dictionary, block_id: String)` → `bool`: For condition-type branches, decides execution based on condition result
- `get_execution_count(inputs: Dictionary, block_id: String)` → `int`: For evaluation-type branches, returns how many times to execute (0 = skip)

Branch data is stored on `FKActionUnit` resources via `branch_id: String` and `branch_inputs: Dictionary`. Legacy `branch_type` values ("if"/"elseif"/"else") are resolved to `"if_branch"` via `registry.resolve_branch_id()` for backward compatibility.

## Critical Workflows

**Creating New Providers**:

1. Add `.gd` file to `actions/{NodeType}/`, `conditions/{NodeType}/`, `events/{NodeType}/`, or `branches/`
2. Extend `FKAction`, `FKCondition`, `FKEvent`, or `FKBranch`
3. Implement required methods (see base classes in `resources/` directory)
4. Registry auto-discovers on plugin reload (no manual registration needed)
5. **Alternative**: Use `generator.gd` to auto-generate providers from scene nodes - it introspects node properties, methods, and signals to create boilerplate providers

**Creating New Branch Providers**:

1. Add `.gd` file to `branches/` directory
2. Extend `FKBranch` (from `resources/fk_branch.gd`)
3. Implement required methods:
   - `get_id()`, `get_name()`, `get_description()`
   - `get_input_type()`: Return `"condition"` to use FKCondition providers, or `"evaluation"` to use direct expression inputs
   - For condition-type: Implement `should_execute(condition_result, inputs, block_id)`
   - For evaluation-type: Implement `get_inputs()` and `get_execution_count(inputs, block_id)`
   - `get_type()`: Return `"chain"` to allow else-if/else chaining, or `"single"` for standalone
   - `get_color()`: Return a `Color` for the branch accent in the editor UI
4. Registry auto-discovers on plugin reload
5. Built-in examples: `branches/if_branch.gd` (condition-type, chainable), `branches/repeat.gd` (evaluation-type, not chainable)

**Event Sheet Execution** (runtime):

- FlowKit engine detects scene changes via `get_tree().current_scene`
- Loads matching `.tres` from `saved/event_sheet/{scene_name}.tres`
- Each `_process()` frame:
  1. Polls event triggers (`registry.poll_event()`)
  2. Evaluates conditions (`registry.check_condition()`)
  3. Executes actions if all conditions pass (`registry.execute_action()`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LexianDEV/FlowKit](https://github.com/LexianDEV/FlowKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
