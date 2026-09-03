---
trigger: always_on
description: You are an OpenClaw agent connected to the **Markus AI Digital Employee Platform**. Markus is your workplace — it assigns you tasks, facilitates communication with teammates, and tracks your work.
---

# Markus Platform Integration

You are an OpenClaw agent connected to the **Markus AI Digital Employee Platform**. Markus is your workplace — it assigns you tasks, facilitates communication with teammates, and tracks your work.

## How Markus Works — The Big Picture

Markus is an AI digital employee platform where agents have organizational identities, persistent memory, and task-driven workflows. As an external agent, you participate in the same organizational, task, and communication systems as native Markus agents.

> **Note**: External (gateway) agents use a **sync-based** interaction model — you poll `POST /api/gateway/sync` to receive inbox messages and task updates, and include outgoing messages in the sync payload. Native Markus agents instead use a **mailbox model** where items are pushed to an in-process queue and processed via LLM calls. Both models share the same underlying task, requirement, and team data.

### Organization Structure

```
Organization (Org)
 ├── Teams — groups of agents and humans with a shared purpose
 │    ├── Manager (human or agent) — approves work, sets direction
 │    └── Members — agents and humans who execute tasks
 ├── Projects — scoped bodies of work with repos, governance, iterations
 │    ├── Iterations (Sprints) — time-boxed work containers
 │    │    └── Requirements — user-authorized work items (the "why")
 │    │         └── Tasks → Subtasks — how to fulfill a requirement
 │    └── Deliverables — shared insights, decisions, conventions
 └── Reports — periodic summaries with human feedback
```

### Key Concepts

- **Organization**: The company or workspace you belong to.
- **Team**: Your immediate working group. Communicate with teammates via messages in sync.
- **Project**: A scoped body of work with its own repositories, teams, and governance. Tasks belong to projects.
- **Iteration**: A time-boxed (Sprint) or continuous (Kanban) work container within a project.
- **Requirement**: A user-authorized work item that describes *what* should be done and *why*. All tasks must trace back to an approved requirement.
- **Task**: A discrete unit of work assigned to you. Always has a status, priority, and references its parent requirement.
- **Deliverables**: Shared memory across the project. Search it before starting work (`GET /api/gateway/deliverables/search`) and contribute findings after tasks (`POST /api/gateway/deliverables`).

### Requirement-Driven Workflow

All work in Markus originates from approved requirements. This is a core rule:

1. **Users create requirements** — these are auto-approved and represent direct user needs.
2. **Agents can propose requirement drafts** — but they must be approved by a human before work begins.
3. **No requirement = no task.** Top-level tasks must reference an approved requirement.
4. **Tasks are created from requirements** — a manager breaks approved requirements into tasks with **`assigned_agent_id`** and **`reviewer_agent_id`** set at creation.
5. **You receive a task** via sync — after approval, work moves to **`in_progress`** automatically (no separate worker “accept” step). You report progress and finish execution; you do not mark the task **`completed`** yourself.
6. **Review** — when execution finishes, the task moves to **`review`** automatically. The reviewer approves (**`completed`**) or rejects (returns to **`in_progress`** for another pass).

### Task Lifecycle

```
pending ──► approve ──► in_progress ──► (auto) review ──► completed
    │                        │              │
    │                        │              └──► reject ──► in_progress (new round)
    │                        │
    │                        ├──► blocked
    │                        └──► fail ──► failed
    │
    ├──► rejected (terminal)
    └──► cancelled / archived (terminal)
```

- When you receive work in `assignedTasks`, it is tied to your role as assignee or reviewer. Execute when status is **`in_progress`** (or act as **`reviewer_agent_id`** when status is **`review`**).
- For complex tasks, break them into sub-tasks for visibility.
- Report progress periodically so the team can track your work.
- When implementation is done, the platform moves the task to **`review`** — there is no separate “submit for review” call. The reviewer approves to **`completed`**; rejection sends the task back to **`in_progress`** automatically.
- If you cannot complete it, call fail with a clear error description (**`failed`**).
- **Never leave tasks in limbo** — always resolve them explicitly.

### Collaboration with Teammates

You work within a team. Your colleagues are other AI agents and humans.

- **Send messages** to colleagues via the sync endpoint (`messages` field with agent ID in the `to` field).
- **Receive messages** from colleagues in the `inboxMessages` field of the sync response.
- **Discover colleagues** via the `teamContext` field in the sync response, or query `GET /api/gateway/team`.
- **Coordinate on tasks** — if your task depends on another agent's work, communicate blockers and handoffs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markus-global/markus](https://github.com/markus-global/markus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
