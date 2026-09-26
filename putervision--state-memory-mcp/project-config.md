---
trigger: always_on
description: This project tracks workflow state, tasks, design decisions, and blockers using `state-memory-mcp` with project slug `"state-memory-mcp"`.
---

## State Memory (state-memory-mcp)

This project tracks workflow state, tasks, design decisions, and blockers using `state-memory-mcp` with project slug `"state-memory-mcp"`.

### 1. Priority Order
Before doing any coding or investigation:
1. `manage_sessions(action: "start")` — Start a tracking session for full change attribution.
2. `get_analytics(action: "summary")` — Run to understand current project state, active branches, and overall progress.
3. `manage_tasks(action: "next")` — Query prioritized runnable tasks.
4. `manage_tasks(action: "find_blockers")` — Identify any active blockers preventing progress.
5. `manage_nodes(action: "list")` — Find pending tasks, past decisions, or milestones.
6. `query_graph(action: "trace")` — Trace what depends on or blocks a task.

### 2. When to Write to the Graph
You MUST update the graph as you work:
- **Starting a session**: Always call `manage_sessions(action: "start", agent_id: "my-agent")` to track all mutations under a unique session.
- **Starting a new task**: Create a node with `manage_nodes(action: "create", type: "task", title: "...", session_id: session_id)`.
- **Making a design or implementation decision**: Document it with `manage_nodes(action: "create", type: "decision", title: "...", metadata: { "rationale": "..." }, session_id: session_id)`.
- **Connecting UI tasks to visual states**: Use `manage_edges(action: "link_visual", target_id: task_id, visual_state_id: vs_id, relationship: "renders_state")`.
- **Encountering a blocker**: Record the blocker with `manage_nodes(action: "create", type: "blocker", ..., session_id: session_id)` and connect it using `manage_edges(action: "add", type: "blocks", source_id: blocker_id, target_id: task_id, session_id: session_id)`.
- **Adding observation notes**: Atomically log notes using `manage_nodes(action: "add_note", text: "...", attach_to: node_id)`.
- **Batch updates**: Bulk update tasks/nodes using `manage_nodes(action: "batch_update", ids: ["..."], status: "done")`.
- **Observability & Trajectories**: Check `manage_data(action: "export_synergy_metrics")` for combined token savings & health, and `manage_data(action: "export_joint_trajectories")` for interleaved fine-tuning datasets.
- **Completing a task**: Update status to done using `manage_tasks(action: "complete", task_id: task_id)`.
- **Creating/generating a new file**: Create an artifact node with `manage_nodes(action: "create", type: "artifact", ..., session_id: session_id)` and connect it using `manage_edges(action: "add", type: "produces", ..., session_id: session_id)`.

### 3. Workflow Pattern
1. **Start of session**: Call `manage_sessions(action: "start")` to align and track work, then run `get_analytics(action: "summary")`, `manage_tasks(action: "next")`, and `manage_tasks(action: "find_blockers")`.
2. **Task decomposition**: Decompose user requests into tasks and add them to the graph.
3. **Execution**: Mark tasks as "in_progress", document design decisions as they occur, and log blockers if you hit any obstacles.
4. **Validation & Resolution**: Run `run_diagnostics(action: "validate")` to ensure no cycles/orphans/contradictions, mark tasks as "done", document completed artifacts, and resolve blockers. Call `manage_sessions(action: "end")` to finalize.

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


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [putervision/state-memory-mcp](https://github.com/putervision/state-memory-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
