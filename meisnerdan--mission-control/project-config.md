---
trigger: always_on
description: - Read `mission-control/data/ai-context.md` FIRST for current state snapshot
---

# Solo Entrepreneur Workspace — Agent Operations Manual

## Quick Start for AI Agents
- Read `mission-control/data/ai-context.md` FIRST for current state snapshot
- For full data, read the JSON files in `mission-control/data/`
- This workspace is designed for multi-agent operation via Claude Code and Claude Cowork
- **Communication**: All agent communication happens through JSON files — see Agent Communication Protocol below

## Workspace Map
```
mission-control/              — Task management + Agent orchestration app (Next.js 15)
mission-control/data/         — JSON data files (THE source of truth for all data)
mission-control/scripts/      — Utility scripts (context generation, daemon)
mission-control/scripts/daemon/ — Autonomous agent daemon (background process)
projects/                     — Individual project codebases (each has its own CLAUDE.md)
research/                     — Research notes (markdown)
docs/                         — Business plans, strategies, analysis
templates/                    — Project templates
scripts/                      — Team execution scripts (run-team.sh, run-task-team.sh)
.claude/commands/             — Claude Code slash commands (auto-generated from agent registry)
.claude-plugin/               — Cowork plugin manifest
skills/                       — Auto-generated skill files from skills-library.json
commands/                     — Plugin commands (Cowork + Claude Code)
```

## Data Schema Reference

### tasks.json — `{ "tasks": Task[] }`
| Field | Type | Description |
|-------|------|-------------|
| id | string | `"task_{timestamp}"` |
| title | string | Short, action-oriented |
| description | string | What needs to be done |
| importance | `"important"` \| `"not-important"` | Eisenhower Y-axis |
| urgency | `"urgent"` \| `"not-urgent"` | Eisenhower X-axis |
| kanban | `"not-started"` \| `"in-progress"` \| `"done"` | Workflow status |
| projectId | string \| null | Links to project |
| milestoneId | string \| null | Links to goal/milestone |
| assignedTo | AgentRole \| null | Lead agent assignment |
| collaborators | string[] | Additional team members (agent IDs) |
| dailyActions | `DailyAction[]` | Sub-steps: `{ id, title, done, date }` |
| subtasks | `Subtask[]` | Checkable sub-items: `{ id, title, done }` |
| blockedBy | string[] | Task IDs this depends on |
| estimatedMinutes | number \| null | Estimated work time |
| actualMinutes | number \| null | Actual work time |
| acceptanceCriteria | string[] | Definition of done |
| tags | string[] | Freeform labels |
| notes | string | Additional context |
| createdAt | ISO 8601 | When created |
| updatedAt | ISO 8601 | Last modification |
| completedAt | ISO 8601 \| null | When marked done |

### goals.json — `{ "goals": Goal[] }`
| Field | Type | Description |
|-------|------|-------------|
| id | string | `"goal_{timestamp}"` or `"mile_{id}"` for milestones |
| title | string | Goal description |
| type | `"long-term"` \| `"medium-term"` | Strategic goal vs milestone |
| timeframe | string | `"Q1 2026"` or `"YYYY-MM-DD"` |
| parentGoalId | string \| null | Milestones point to parent goal |
| projectId | string \| null | Linked project |
| status | `"not-started"` \| `"in-progress"` \| `"completed"` | Progress |
| milestones | string[] | Child milestone IDs (long-term goals) |
| tasks | string[] | Linked task IDs |
| createdAt | ISO 8601 | When created |

### projects.json — `{ "projects": Project[] }`
| Field | Type | Description |
|-------|------|-------------|
| id | string | `"proj_{timestamp}"` |
| name | string | Project name |
| description | string | What this project is |
| status | `"active"` \| `"paused"` \| `"completed"` \| `"archived"` | Lifecycle |
| color | string | Hex color for UI |
| teamMembers | string[] | Assigned agent IDs |
| tags | string[] | Freeform labels |
| createdAt | ISO 8601 | When created |

### agents.json — `{ "agents": AgentDefinition[] }`
| Field | Type | Description |
|-------|------|-------------|
| id | string | URL-safe slug (e.g. `"researcher"`) |
| name | string | Display name |
| icon | string | Lucide icon name |
| description | string | What this agent handles |
| instructions | string | Full system prompt (multi-line markdown) |
| capabilities | string[] | What this agent can do |
| skillIds | string[] | Links to skills-library entries |
| status | `"active"` \| `"inactive"` | Agent lifecycle |
| createdAt | ISO 8601 | When created |
| updatedAt | ISO 8601 | Last modification |

### skills-library.json — `{ "skills": SkillDefinition[] }`
| Field | Type | Description |
|-------|------|-------------|
| id | string | `"skill_{name}"` |
| name | string | Skill display name |
| description | string | When to use this skill |
| content | string | Full skill markdown (injected into agent prompts) |
| agentIds | string[] | Which agents have this skill |
| tags | string[] | Freeform labels |
| createdAt | ISO 8601 | When created |
| updatedAt | ISO 8601 | Last modification |

### brain-dump.json — `{ "entries": BrainDumpEntry[] }`
| Field | Type | Description |
|-------|------|-------------|
| id | string | `"bd_{timestamp}"` |
| content | string | Raw idea/note (keep short) |
| capturedAt | ISO 8601 | When captured |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MeisnerDan/mission-control](https://github.com/MeisnerDan/mission-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
