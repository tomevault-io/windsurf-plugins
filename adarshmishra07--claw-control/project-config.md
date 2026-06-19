---
trigger: always_on
description: Complete AI agent operating system with Kanban task management, multi-agent coordination, human approval gates, and autonomous task discovery via heartbeat polling. Use when setting up multi-agent coordination, task tracking, or configuring an agent team. Includes theme selection (DBZ, One Piece, Marvel, etc.), workflow enforcement (all tasks through board), browser setup, GitHub integration, and memory enhancement (Supermemory, QMD).
---


# Claw Control - Agent Operating System (v2)

Complete setup for AI agent coordination with real-time Kanban dashboard, featuring autonomous task discovery, multi-agent collaboration, and human approval gates.

## What This Skill Does

1. **Deploy Claw Control** - Three paths: one-click, bot-assisted, or fully automated
2. **Theme your team** - Pick a series (DBZ, One Piece, Marvel, etc.)
3. **Enforce workflow** - ALL tasks go through the board, no exceptions
4. **Configure agent behavior** - Update AGENTS.md and SOUL.md
5. **Setup browser** - Required for autonomous actions
6. **Setup GitHub** - Enable autonomous deployments
7. **Enhance memory** - Integrate Supermemory and QMD

---

## 🚀 v2 Features Overview

Claw Control v2 includes powerful new capabilities for multi-agent orchestration:

| Feature | Description |
|---------|-------------|
| **Task Comments** | Collaborate on tasks with POST/GET /api/tasks/:id/comments |
| **Task Context** | Rich context field for passing additional data to agents |
| **Task Deliverables** | deliverable_type + deliverable_content for concrete outputs |
| **Agent Heartbeat Polling** | Autonomous task discovery via PUT /api/agents/:id/heartbeat |
| **Human Approval Gates** | requires_approval, approved_at, approved_by for quality control |
| **Multi-Agent Assignment** | Multiple agents can work on one task with roles (lead/contributor/reviewer) |
| **Task Subtasks** | Break down complex tasks with POST/GET/PUT/DELETE /api/tasks/:id/subtasks |

### Jarvis Multi-Agent Pattern (Best Practices)

The Jarvis pattern enables truly autonomous multi-agent systems:

- **15-min Heartbeat Polling**: Each agent polls every 15 minutes to discover and claim tasks
- **Hierarchical Structure**: Coordinator → Specialists (coordinator delegates, specialists execute)
- **Self-Discovery**: Agents query the board for unassigned tasks and claim them autonomously
- **Collaborative Tasks**: Multiple agents can contribute to a single task
- **Deliverable-First**: Every task MUST have a clear deliverable

---

## ⚠️ CRITICAL: The Golden Rules

**After setup, you MUST follow these rules EVERY TIME:**

### Before Doing ANY Work:
1. **Create a task on Mission Control** - Even for small things
2. **Spawn a sub-agent** - Use `sessions_spawn` to delegate
3. **Never do the work yourself** - Coordinator coordinates, agents execute

### The Workflow (No Exceptions):
```
User Request → Create Task → Spawn Agent → Agent Works → Review → Complete
```

### If You Catch Yourself Working:
**STOP!** Ask: "Did I create a task? Did I spawn an agent?"
If no → Go back and do it properly.

### If You Catch An Agent Breaking Rules:
**PAUSE** and enforce:
- Working without a task? → "Is this on the board?"
- Acting solo? → "Did you delegate/query/verify?"
- Skipping review? → "Let's check before marking complete"

**Your role is COORDINATOR.** Coordinate, review, verify, **ENFORCE**. Never execute.

### ⚠️ CRITICAL: How Subagents Work

When users ask about "agents" or "subagents", explain clearly:

**The agents are NOT just UI labels — they are REAL autonomous executors.**

- We use `sessions_spawn` to create isolated sub-agent sessions
- Each subagent runs independently with its own model & context
- They execute tasks in parallel and report back when complete
- This is actual distributed computing, not visualization

**Example:** When a user asks "Build X", we:
1. Create a task on Mission Control
2. Spawn a subagent via `sessions_spawn` 
3. Subagent executes autonomously (code, research, deployment, etc.)
4. Results report back to the main session

**Never tell users agents are "just UI representations" — they are real autonomous workers.**

### 🔒 Repo Hierarchy (For claw-control maintainers)

| Repo | Purpose | Direct Push? |
|------|---------|--------------|
| `claw-control-trip/` | Internal testing | ✅ Yes |
| `claw-control` (public) | Production OSS | ❌ PR only after internal test |

**Rule:** Test ALL changes in `claw-control-trip/` FIRST, then PR to public `claw-control`.

### 📝 Commit Message Convention

```
[#TASK_ID] Brief description

Example:
[#129] Add workflow enforcement to SKILL.md
```

If you committed without a task: **CREATE ONE RETROACTIVELY** and link it.

### 🚨 Orphan Work Protocol

If work was done without a task on Mission Control:
1. STOP and create the task NOW
2. Mark it with what was done
3. Set status to `completed`
4. Don't let it happen again

### 📢 Feed Protocol (Communication)

All significant updates go to the agent feed via `POST /api/messages`:

```bash
curl -X POST <BACKEND_URL>/api/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: <API_KEY>" \
  -d '{"agent_id": 1, "message": "✅ Task #X completed: Brief summary"}'
```

**When to post:**
- ✅ Task completions
- 🚀 Major milestones  
- 🔍 Audit results
- 📦 Deployment updates
- 🚧 Blockers or questions
- 💡 Discoveries or insights


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adarshmishra07/claw-control](https://github.com/adarshmishra07/claw-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
