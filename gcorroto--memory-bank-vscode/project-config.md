---
trigger: always_on
description: - **Project ID**: `memory_bank_vscode_extension`
---

# Memory Bank MCP - Auto-Index Mode

## Project Configuration

- **Project ID**: `memory_bank_vscode_extension`
- **Mode**: Auto-Index (continuous RAG synchronization)

---

## Memory Bank Instructions

This project uses Memory Bank MCP as a **RAG system** (Retrieval-Augmented Generation). It keeps your knowledge accurate and **prevents hallucinations**.

### ⚠️ CRITICAL RULES - MUST FOLLOW

#### Rule 0: COORDINATE WITH OTHER AGENTS

**BEFORE starting any task, you MUST check the Agent Board.** This prevents multiple agents from modifying the same files simultaneously or duplicating work.

1. **Check Board**: Use `memorybank_manage_agents` with `action: "get_board"` to see active agents/locks.
2. **Register**: Identity yourself (e.g., `role-ide-model`). Call `action: "register"` with your `agentId`. The system will assign a **Session ID** for tracking context automatically.
3. **Claim Task**: `action: "claim_resource"` for the file/feature you are working on.
4. **Work**: Perform your task (Route -> Search -> Implement -> Index).
5. **Release**: `action: "release_resource"` when done.

#### Rule 0.5: ROUTE BEFORE IMPLEMENTING (MANDATORY)

**BEFORE writing ANY code, you MUST call `memorybank_route_task` to determine what belongs to this project vs what should be delegated.**

```json
// memorybank_route_task - MANDATORY before ANY code changes
{
  "projectId": "memory_bank_vscode_extension",
  "taskDescription": "[describe what you're about to implement]"
}
```

The orchestrator will analyze the task against ALL projects' responsibilities and return:
- **myResponsibilities**: What YOU should implement in this project
- **delegations**: What should be delegated to other projects
- **suggestedImports**: What to import from other projects
- **architectureNotes**: Design guidance

**Why this matters:**
- Prevents creating code that belongs to other projects (e.g., DTOs in API when lib-dtos exists)
- Ensures proper separation of concerns across the ecosystem
- Automatically delegates work to the right project

**NO EXCEPTIONS.** Even for "simple" tasks - the orchestrator knows the ecosystem better than you.

#### Rule 1: ALWAYS SEARCH BEFORE IMPLEMENTING

**NEVER write code without first consulting the Memory Bank.**

```json
// memorybank_search - MANDATORY before ANY implementation
{
  "projectId": "memory_bank_vscode_extension",
  "query": "how does [feature/component] work"
}
```

**When to search:**
- Before implementing anything → Search for similar patterns
- Before modifying code → Search for usages and dependencies  
- Before answering questions → Search for accurate info
- Before suggesting architecture → Search for existing patterns

#### Rule 2: ALWAYS REINDEX AFTER MODIFYING

**IMMEDIATELY after modifying ANY file, reindex it.**

```json
// memorybank_index_code - MANDATORY after ANY file change
{
  "projectId": "memory_bank_vscode_extension",
  "path": "path/to/modified/file.ts"
}
```

**No exceptions.** Keeps the RAG updated and accurate.

#### Rule 3: RESPECT PROJECT BOUNDARIES

**You own `memory_bank_vscode_extension`. Do NOT modify other projects.**
- **Discover**: `memorybank_discover_projects` to find other agents.
- **Delegate**: `memorybank_delegate_task` to hand off work.

#### Rule 4: DOCUMENT EVERYTHING CONTINUOUSLY

**After EVERY significant action, update the Memory Bank:**

1. **Track progress** after completing ANY task:
   ```json
   { "projectId": "memory_bank_vscode_extension", "progress": { "completed": ["Task done"], "inProgress": ["Next task"] } }
   ```

2. **Record decisions** when making architectural/technical choices:
   ```json
   { "projectId": "memory_bank_vscode_extension", "decision": { "title": "...", "description": "...", "rationale": "..." } }
   ```

3. **Update context** to leave notes for next session:
   ```json
   { "projectId": "memory_bank_vscode_extension", "recentChanges": ["..."], "nextSteps": ["..."] }
   ```

**The goal: Next session (or another agent) can pick up exactly where you left off.**

---

### Available Tools

#### Core Memory Bank (Semantic RAG)
| Tool | When to Use |
|------|-------------|
| `memorybank_route_task` | **FIRST - Before ANY code changes** |
| `memorybank_search` | **SECOND - Before implementation** |
| `memorybank_index_code` | **AFTER any modification** |
| `memorybank_read_file` | When need full file context |
| `memorybank_write_file` | Write with auto-reindex |

#### Multi-Project
| Tool | Description |
|------|-------------|
| `memorybank_manage_agents` | Coordination, locking & task management |
| `memorybank_discover_projects` | Find other projects |
| `memorybank_delegate_task` | Handoff work to other projects |

#### Agent Board Actions (`memorybank_manage_agents`)
| Action | Description |
|--------|-------------|
| `register` | Register agent at session start |
| `get_board` | View agents, tasks, locks |
| `claim_task` | Claim a pending task |
| `complete_task` | Mark task as completed |
| `claim_resource` | Lock a file/resource |
| `release_resource` | Unlock a file/resource |
| `update_status` | Update agent status |

#### Project Knowledge Layer
| Tool | Description |
|------|-------------|
| `memorybank_generate_project_docs` | Generate AI docs (replaces templates) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gcorroto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
