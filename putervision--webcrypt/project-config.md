---
trigger: always_on
description: This project tracks workflow state, tasks, design decisions, and blockers using `state-graph-mcp`.
---

## State Graph (state-graph-mcp)

This project tracks workflow state, tasks, design decisions, and blockers using `state-graph-mcp`.

### 1. Priority Order

Before doing any coding or investigation:

1. `get_project_summary` — Always run this first to understand current project state, active branches, and overall progress.
2. `find_blockers` — Identify any active blockers preventing progress.
3. `list_nodes` — Find pending tasks, past decisions, or milestones.
4. `trace_dependencies` — Trace what depends on or blocks a task.

### 2. When to Write to the Graph

You MUST update the graph as you work:

- **Starting a new task**: Create a node with `add_node(type: "task", title: "...")`.
- **Making a design or implementation decision**: Document it with `add_node(type: "decision", title: "...", metadata: { "rationale": "..." })`.
- **Encountering a blocker**: Record the blocker with `add_node(type: "blocker", ...)` and connect it to the blocked task using `add_edge(type: "blocks", source_id: blocker_id, target_id: task_id)`.
- **Completing a task**: Update the status to done using `update_node(id: task_id, status: "done")`.
- **Creating/generating a new file**: Create an artifact node with `add_node(type: "artifact", ...)` and connect it using `add_edge(type: "produces")`.

### 3. Workflow Pattern

1. **Start of session**: Run `get_project_summary` and `find_blockers` to align on current status.
2. **Task decomposition**: Decompose user requests into tasks and add them to the graph.
3. **Execution**: Mark tasks as "in_progress", document design decisions as they occur, and log blockers if you hit any obstacles.
4. **Resolution**: Mark tasks as "done", document completed artifacts, and resolve blockers.

### 4. Codebase Seeding on Initialization

If the project was just initialized or is missing high-level structure (Plans, Milestones, Decisions):

1. **Inspect the Codebase**: Read the README and core files to understand the roadmap and architecture.
2. **Scaffold the Roadmap**: Create a `plan` node (e.g., "Project Roadmap") and add `milestone` nodes representing key target phases, connecting them using `part_of` edges.
3. **Scaffold Architecture**: Create `decision` nodes representing core technical choices (e.g., choice of databases, frameworks) and link them to the milestones/tasks using `decided_in` edges.

<!-- vision-memory-mcp:start -->

## Visual Memory (vision-memory-mcp)

This project utilizes `vision-memory-mcp` to cache visual states, record layout transitions, provide element grounding, and avoid repetitive LLM vision calls.

### 1. Mandatory Workflow & Priority

1. **Orient**: Call `get_session_context` to align your visual state context at the start of work.
2. **Search**: Call `recall_memory` (text/image search) before recreating duplicate UI state paths.
3. **Ingest/Verify**: ALWAYS call `analyze_screenshot` before querying any front-end vision models.
   - **Cache Hit (`is_known: true`)**: Do NOT use vision models; read the returned `description` as context and use `grounded_elements` (selectors, coordinates) for action target selection.
   - **Cache Miss (`is_known: false`)**: Query your vision model, then run `analyze_screenshot` with both the image and description to seed the cache.
4. **Action Target Execution**: Use `predict_next_action` to retrieve `grounded_target` handles (`target_selector`, `target_coords`) for deterministic UI clicks and typing.
5. **Transitions**: Call `record_outcome` after every click/type/scroll action to construct navigation paths.
6. **Privacy & Cleanup**: Call `forget_state` to purge sensitive or secret states from storage.

### 2. Tool Reference Summary (15 Core MCP Tools)

- `analyze_screenshot`: Ingest screenshot(s) (single or batch via `items`), lookup cache, return layout description and grounded elements.
- `recall_memory`: Search visual memory by description query or base64 image query (read-only).
- `record_outcome`: Save UI action execution outcomes, transitions, or log visual blockers (`action_type: 'blocker'`).
- `get_navigation_paths`: Find path between states using BFS navigation graph.
- `predict_next_action`: Predict best next UI action and target coordinates based on transition success rates and AX tree grounding.
- `compare_states`: Compare visual states structurally (`has_layout_change`) or compare video recordings (`video_a_id`/`video_b_id`).
- `get_session_context`: Fetch aggregated visual context, recent/frequent states, transitions, cache hit ratios, token savings metrics, and server version info.
- `manage_snapshot`: Unified snapshot management (`save`, `diff`, `export`, `restore`) for visual checkpoints and regression detection.
- `manage_visual_spec`: Visual SDD design contract baseline registration (`set`), live verification (`verify`), and listing (`list`).
- `manage_video`: Unified video memory operations for ingestion (`ingest`), semantic search (`search`), and keyframe timelines (`timeline`).
- `create_evidence_pack`: Create cryptographic, multi-modal evidence pack linking video keyframes, state graph tasks, and visual proof.
- `export_trajectories`: Export multimodal visual transitions and joint workflow trajectories (`json`, `llava`, `qwen2_vl`, `joint`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [putervision/WebCrypt](https://github.com/putervision/WebCrypt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
