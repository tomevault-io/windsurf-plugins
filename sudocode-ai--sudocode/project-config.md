---
trigger: always_on
description: **sudocode** is a git-native context management system for AI-assisted software development. It provides a 4-tiered abstraction structure:
---

# sudocode Repository Guide for AI Coding Agents

## Project Overview

**sudocode** is a git-native context management system for AI-assisted software development. It provides a 4-tiered abstraction structure:

1. **Spec** - User intent and requirements (WHAT you want)
2. **Issue** - Agent-scoped work items (Tasks within agent scope)
3. **Execution** - Agent run trajectory (HOW it was executed)
4. **Artifact** - Code diffs and output (Results)

**Core Value Proposition:** Treats context as code: git-tracked, distributed, mergeable.

---

## Monorepo Structure

```
sudocode/
├── types/          # Shared TypeScript definitions (build first)
├── cli/            # CLI (@sudocode-ai/cli) - core operations
├── mcp/            # MCP server (@sudocode-ai/mcp) - wraps CLI
├── server/         # Local backend (@sudocode-ai/local-server) - executions + API
├── frontend/       # React UI (@sudocode-ai/local-ui) - web interface
├── sudocode/       # Meta-package (bundles all)
└── .sudocode/      # Example project data (self-hosting)
```

**Package Dependencies:**
- `types` → standalone (no deps)
- `cli` → depends on `types`
- `mcp` → depends on `cli` (wraps via child_process)
- `server` → depends on `cli`
- `frontend` → independent (talks to server via REST/WS)

---

## Build & Test

```bash
npm run build                    # Build all packages
npm run test                     # Run all tests
npm --prefix <pkg> test -- --run # Package-specific tests
npm --prefix <pkg> test -- --run tests/path/to/file.test.ts
```

**Test Organization:**
- Frontend: `tests/components/`, `tests/pages/`, `tests/hooks/`, `tests/contexts/`
- Backend: `tests/unit/`, `tests/integration/`
- Naming: `*.test.ts` (unit), `*.test.tsx` (React components)

---

## Core Architecture

### Storage: Distributed Git Database

```
Markdown Files (.sudocode/specs/*.md)
    ↕ (syncs via watcher)
JSONL Files (specs.jsonl, issues.jsonl) ← git-tracked
    ↕ (import/export)
SQLite Cache (cache.db) ← QUERY ENGINE (gitignored, rebuilt from JSONL)
```

**Source of Truth Configuration** (`config.json`):
- `"sourceOfTruth": "jsonl"` (default) - JSONL files are authoritative, markdown is derived
- `"sourceOfTruth": "markdown"` - Markdown files are authoritative, JSONL is derived

**Key Principles:**
1. **Configurable source of truth** - Choose between JSONL or Markdown as authoritative
2. **JSONL always exported** - For git tracking, regardless of source of truth mode
3. **SQLite is query cache** - Gitignored, rebuilt after `git pull`
4. **Markdown is human interface** - Can be authoritative or derived based on config
5. **Git handles distribution** - AI handles merge conflicts

### Data Model

| Entity | ID Format | Purpose | Key File |
|--------|-----------|---------|----------|
| **Spec** | `s-xxxx` | User intent, requirements | `types/src/index.d.ts` |
| **Issue** | `i-xxxx` | Actionable work for agents
- Storage: JSONL (markdown optional)
- Status: blocked/open → in_progress → needs_review/closed

**Execution** (Agent Run)
- Purpose: Track agent execution on an issue
- Agent Types: claude-code, codex, copilot, cursor
- Status: preparing → pending → running → paused/completed/failed/cancelled/stopped
- Captures: Git commits, logs, exit code, files changed, session_id
- Modes: worktree (isolated), local (in-place)
- Workflow Support: `workflow_execution_id`, `step_type`, `step_index`, `step_config` for multi-step chains
- Follow-ups: `parent_execution_id` links execution chains together

**IssueFeedback** (Implementation → Requirements Loop)
- Purpose: Issues provide anchored feedback on specs **or other issues**
- Types: comment, suggestion, request
- Anchoring: Line-based with smart relocation (LocationAnchor → FeedbackAnchor)
- Polymorphic: `from_id` (issue) → `to_id` (spec or issue)
- Bidirectional: Closes loop from implementation back to requirements

**Relationship** (Graph Edges)
- Types: `blocks`, `implements`, `depends-on`, `references`, `discovered-from`, `related`
- Polymorphic: Can link any entity types (spec→spec, issue→issue, issue→spec)
- Bidirectional: Tracked in both directions

### ID Generation

- **Hash-based IDs**: `s-xxxx` (specs), `i-xxxx` (issues)
  - Generated from: `${entityType}-${title}-${timestamp}`
  - 4-8 characters, collision-resistant, git-merge friendly
- **UUIDs**: Each entity has both `id` (hash) and `uuid` (UUID v4)
  - `uuid` used for distributed sync/deduplication
  - `id` used for user-facing references

### Cross-References in Markdown

```markdown
[[s-abc123]]              # Spec reference
[[@i-xyz]]               # Issue reference
[[s-abc123|Display]]     # With display text
[[i-xyz]]{ blocks }      # With relationship type
```

Extracted via regex, creates bidirectional relationships automatically.

### Key Concepts

- **Execution Chains:** Root execution + follow-ups via `parent_execution_id`
- **Worktree Isolation:** Each execution gets isolated git worktree
- **Multi-Agent:** Supports claude-code, codex, copilot, cursor (see `types/src/agents.d.ts`)
- **Polymorphic Feedback:** Issues can provide feedback on specs OR other issues

## Key Files by Package

### types/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sudocode-ai/sudocode](https://github.com/sudocode-ai/sudocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
