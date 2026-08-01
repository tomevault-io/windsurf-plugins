---
trigger: always_on
description: Core business logic package for Craft Agent:
---

# CLAUDE.md — `@craft-agent/shared`

## Purpose
Core business logic package for Craft Agent:
- Agent backends and session-scoped tools
- Sources, credentials, sessions, and config
- Permission modes and validation

## Key folders
- `src/agent/` — `claude-agent.ts`, `pi-agent.ts`, `base-agent.ts`, tools, permissions
- `src/sources/` — source storage/types/services
- `src/sessions/` — session persistence/index
- `src/projects/` — workspace-scoped projects (config + assets); sessions bind via `projectId`
- `src/config/` — config/preferences/theme/watcher
- `src/credentials/` — encrypted credential management

## Commands
From repo root:
```bash
cd packages/shared && bun run tsc --noEmit
```

## Hard rules
- Permission modes are fixed: `safe`, `ask`, `allow-all`.
- Source types are fixed: `mcp`, `api`, `local`.
- Keep credential handling in `src/credentials/` pathways (no ad-hoc secret storage).
- Keep user-facing tool contracts backward-compatible where possible.

## Notes
- `ClaudeAgent` is the primary class in `src/agent/claude-agent.ts`.
- **Reserved "Task" labels.** Task flows tag each task's whole family (orchestrator + subtasks) with a per-task ITEM label — a child of the plain root `Task` label, named `TASK-<slug>-<N>` (no valueType; N = max counter across the root's TASK-named children + 1, never recycled — unrelated children of an adopted user root don't feed it). Mint/inherit only via `SessionManager.applyTaskLabel` (which uses `ensureTaskLabel`/`ensureTaskItemLabel` in `labels/crud.ts`); resolve only via `findTaskLabel` / `findTaskItemLabelId` / `resolveTaskScopeLabelId` (`labels/filter.ts`). Never assume literal ids — slugs collide-shift, so always use the resolved id (surfaced as `TaskCreateResult.taskLabelId`). Legacy `task::N` valued entries still filter under the root; `ensureTaskLabel` converges a legacy `valueType: 'number'` root to a plain label but adopts a user's own root "Task" label as-is (shape + children untouched).
- **Single label-filter predicate.** `matchesLabelFilter` (`labels/filter.ts`, browser-safe) is the only implementation of "session matches a label filter" (descendants, `__all__`, optional `projectId` scope). The session list, AppShell filtered set, and NavigationContext auto-select all route through it — do not hand-roll label matching in feature code.
- Claude SDK subprocess env is sanitized to strip Claude-specific Bedrock routing vars (`CLAUDE_CODE_USE_BEDROCK`, `AWS_BEARER_TOKEN_BEDROCK`, `ANTHROPIC_BEDROCK_BASE_URL`). Pi Bedrock uses its own AWS env path instead.
- Backward alias export (`CraftAgent`) exists for compatibility.
- Prefer routing new model vendors through the existing Pi path (`providerType: 'pi'` + `piAuthProvider`) unless they truly need a distinct runtime/backend. The Pi provider catalog and display metadata live in `src/config/models-pi.ts`.
- Custom endpoint model capabilities must preserve explicit per-model overrides end-to-end. In particular, `supportsImages: true` enables image input for one model and `supportsImages: false` must remain available to override a global endpoint image default. Active Pi custom-endpoint sessions refresh runtime capabilities via `updateRuntimeConfig`; capability changes are pushed proactively from the `llmConnections.SAVE` handler through `SessionManager.refreshConnectionRuntime`, with the lazy `getOrCreateAgent` path acting as a backstop. The session layer still gates image attachments at send time so disabled images are not sent even if a subprocess refresh fails.
- `update_runtime_config` IPC carries `model, providerType, authType, baseUrl, customEndpoint, customModels` only — `piAuthProvider`, `slug`, and the broader credential/provider routing state cannot be re-routed inside a live Pi subprocess. `runtime-config.ts:buildRestartRequiredSignature` hashes those fields separately from the in-place-safe ones; when the restart signature drifts, `tryRefreshAgentRuntime` skips the in-place attempt and goes straight to dispose + recreate so the new auth/provider state actually takes effect.
- Session lifecycle distinguishes **hard aborts** from **UI handoff interrupts**:
  - use hard aborts for true cancellation/teardown (`UserStop`, redirect fallback)
  - use handoff interrupts for pause points where control moves to the UI (`AuthRequest`, `PlanSubmitted`)
- Remote workspace handoff summaries are injected as one-shot hidden context on the destination session's first turn.
- **Task creation is a shared core.** `createTaskFromSpec` / `finishTaskOrchestrator` (`packages/server-core/src/tasks/create-task.ts`) implement "create the task on the board without running it" (task.yaml + orchestrator session + reserved TASK label + spec sources, fail-soft on label/sources). Both the `tasks:create` RPC fresh path and the agent-facing `create_task` session tool call it — never re-implement the flow. The tool path derives the slug via `uniqueTaskSlug` (`shared/tasks/slug.ts`, never overwrites; the TaskEditor keeps its own `slugify` copy because this barrel pulls Node fs code), inherits the invoking session's project unless `projectId` explicitly overrides it, and synthesizes the required single `main` node from the description; DAG authoring stays with the editor/`tasks:generate`. Running remains exclusively `tasks:run`/TaskRunner.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craft-ai-agents/craft-agents-oss](https://github.com/craft-ai-agents/craft-agents-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
