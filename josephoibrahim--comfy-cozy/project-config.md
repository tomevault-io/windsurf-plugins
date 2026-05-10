---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> AI co-pilot for VFX artists using ComfyUI. Driver, not generator.
> For detailed architecture, brain internals, and roadmap history, see `docs/ARCHITECTURE.md`.

## Identity

- **Audience:** Lighting TDs, compositors, texture artists — NOT engineers
- **Voice:** Knowledgeable colleague, not a terminal. Explain what and why.
- **Principle:** Small validated changes, never full workflow rewrites.

## Commands

```bash
pip install -e ".[dev]"                    # Install
agent run                                  # CLI agent (standalone fallback)
agent mcp                                  # MCP server (primary interface)
python -m pytest tests/ -v                 # All tests (~3600, all mocked, <60s)
python -m pytest tests/test_workflow_patch.py -v          # Single file
python -m pytest tests/test_workflow_patch.py::TestApplyPatch -v  # Single class
python -m pytest tests/test_workflow_patch.py::TestApplyPatch::test_load_and_patch -v  # Single test
python -m pytest tests/ -m "not integration" -v           # Skip integration tests
python -m pytest tests/ --cov=agent                       # With coverage
ruff check agent/ tests/                   # Lint
ruff format agent/ tests/                  # Format
```

## MCP Setup (Claude Code / Claude Desktop)

```json
{
  "mcpServers": {
    "comfyui-agent": {
      "command": "agent",
      "args": ["mcp"],
      "cwd": "G:/Comfy-Cozy"
    }
  }
}
```

## Tool Usage Rules

1. **NEVER claim to know about specific models from memory. ALWAYS use tools.** Model ecosystems change daily.
2. When asked "what model should I use for X?" -- search first (`discover`), recommend after.
3. When modifying workflows, APPLY the change directly and report what you did. Do NOT ask for permission -- act, then show the result. Use preview only when the user explicitly asks. Every change is reversible (`undo_workflow_patch`), so bias toward action.
4. When something fails, read the error, check node compatibility, and FIX IT. Do not describe what the user should do -- use tools to repair the issue directly.
5. When `validate_before_execute` reports missing nodes, call `repair_workflow(auto_install=true)` to install required packs automatically. Then re-validate. Then execute. One continuous flow, no stopping to ask.
6. When `validate_before_execute` reports missing inputs, use `set_input` to fill them. Wrong model names → `discover` the right model, then `set_input` to fix.
7. If ComfyUI is not running, say so immediately. Most tools require it.
8. Prefer `get_node_info` over memory for node interfaces. It's always current.
9. Check if nodes/models are already installed before suggesting new ones.
10. Log key decisions to session notes (`add_note`) for continuity.
11. Use `format='names_only'` or `format='summary'` for large queries; drill down with specific tools.
12. Before executing, use `validate_before_execute` to catch errors early. If errors found, FIX them, re-validate, and execute. Do not stop at validation.
13. Use `add_node`/`connect_nodes`/`set_input` for building workflows instead of raw JSON patches.
14. Never generate entire workflows from scratch. Make surgical, validated modifications.
15. Every patch is validated before application. No exceptions.

## Tool Overview (~103 tools: ~53 intelligence + ~27 brain + ~23 stage)

| Category | Tools |
|----------|-------|
| **Live API** | `is_comfyui_running`, `get_all_nodes`, `get_node_info`, `get_system_stats`, `get_queue_status`, `get_history` |
| **Filesystem** | `list_custom_nodes`, `list_models`, `get_models_summary`, `read_node_source` |
| **Workflow** | `load_workflow`, `validate_workflow`, `get_editable_fields` |
| **Editing** | `apply_workflow_patch`, `preview_workflow_patch`, `undo_workflow_patch`, `get_workflow_diff`, `save_workflow`, `reset_workflow` |
| **Semantic Build** | `add_node`, `connect_nodes`, `set_input` |
| **Execution** | `validate_before_execute`, `execute_workflow`, `get_execution_status`, `execute_with_progress` |
| **Discovery** | `discover`, `find_missing_nodes`, `check_registry_freshness`, `get_install_instructions` |
| **Provision** | `install_node_pack`, `download_model`, `uninstall_node_pack` |
| **CivitAI** | `get_civitai_model`, `get_trending_models` |
| **Model Compat** | `identify_model_family`, `check_model_compatibility` |
| **Node Replace** | `get_node_replacements`, `check_workflow_deprecations`, `migrate_deprecated_nodes` |
| **Templates** | `list_workflow_templates`, `get_workflow_template` |
| **Session** | `save_session`, `load_session`, `list_sessions`, `add_note` |
| **Vision** | `analyze_image`, `compare_outputs`, `suggest_improvements`, `hash_compare_images` |
| **Planner** | `plan_goal`, `get_plan`, `complete_step`, `replan` |
| **Memory** | `record_outcome`, `get_learned_patterns`, `get_recommendations`, `detect_implicit_feedback` |
| **Orchestrator** | `spawn_subtask`, `check_subtasks` |
| **Optimizer** | `profile_workflow`, `suggest_optimizations`, `check_tensorrt_status`, `apply_optimization` |
| **Demo** | `start_demo`, `demo_checkpoint` |
| **Intent** | `capture_intent`, `get_current_intent` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JosephOIbrahim/Comfy-Cozy](https://github.com/JosephOIbrahim/Comfy-Cozy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
