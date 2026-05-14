---
trigger: always_on
description: ThreatCaddy is a client-side threat intelligence and incident response platform. Chrome extension + React SPA + optional team server. All investigation data lives in IndexedDB via Dexie. The extension proxies LLM API calls and handles CORS-bypassing fetches.
---

# ThreatCaddy — Claude Code Guidelines

## What This Is

ThreatCaddy is a client-side threat intelligence and incident response platform. Chrome extension + React SPA + optional team server. All investigation data lives in IndexedDB via Dexie. The extension proxies LLM API calls and handles CORS-bypassing fetches.

## Architecture

- **SPA**: React + TypeScript + Vite + Tailwind. Entry: `src/App.tsx`
- **Database**: Dexie (IndexedDB). Schema: `src/db.ts`. Currently version 28.
- **Extension**: `extension/src/` — `background.js` (LLM streaming, fetch proxy, notifications), `bridge.js` (page↔extension message relay), `content.js` (capture UI)
- **Team Server**: `server/` — Hono + Drizzle + PostgreSQL. Syncs investigations, runs server-side agents, manages bots.
- **CaddyAI Chat**: `src/components/Chat/ChatView.tsx` + `src/hooks/useLLM.ts`. Human-driven conversational AI. Stays mounted in background when switching tabs.
- **AgentCaddy**: `src/components/Agent/` + `src/lib/caddy-agent*.ts` + `src/hooks/useCaddyAgent.ts`. Autonomous multi-agent system.

## Agent System

### 17 Builtin Profiles (`src/lib/builtin-agent-profiles.ts`)
**Executive (can dismiss/spawn agents):** CISO, Chief of Staff
**Leadership:** Lead Analyst
**Security Specialists:** IOC Enricher, Timeline Builder, Hypothesis Writer, Threat Hunter, Malware Analyst, Network Forensics, Digital Forensics, Vulnerability Analyst
**Business Stakeholders (observer):** Legal Counsel, Compliance Officer, Communications Lead, Business Continuity
**Cross-Case (specialist role):** Pattern Hunter, Reporter
**Security:** Forensicate Scanner

> Hypothesis Writer reuses the `ap-case-analyst` profile id (preserves user state). The 2026-04-16 audit (`claudewiki/wiki/concepts/agentcaddy-profile-audit.md`) walks the reasoning. Pattern Hunter dropped `lead` → `specialist` because `executeDelegateTask` is single-folder-scoped — the lead-only delegation tools couldn't usefully delegate cross-case findings.

### Key Concepts
- **Profiles** (`AgentProfile`): Reusable config with role (executive/lead/specialist/observer), systemPrompt, allowedTools, policy, readOnlyEntityTypes, optional `soul` (cross-investigation identity).
- **Deployments** (`AgentDeployment`): Profile assigned to an investigation. Each gets its own audit ChatThread. Supports competitiveness (cooperative/competitive/independent), shift state (active/resting), and explicit `handoffState` (client/handoff-pending/server/reclaim-pending).
- **Tool allowlist**: `buildAgentToolset` in `caddy-agent.ts` is the single source of truth for the LLM-visible tool list AND the runtime authorization gate — they can't drift. Locked down by 9 invariant tests in `src/__tests__/agent-toolset.test.ts`.
- **Execution**: `caddy-agent-manager.ts` runs deployments in parallel (max 5 concurrent) via `Promise.allSettled`. Skips deployments where `shouldBlockNewCycle()` is true (server owns it, or client hasn't reconciled). Falls back to legacy single-agent mode when no deployments exist.
- **Delegation**: Lead/executive agents get `delegate_task` + `list_agent_activity` + `review_completed_task` tools. `review_completed_task` requires a structured `requestedDelta` on reject and auto-escalates after 3 rejections (Task gains `rejectionCount`, `rejectionHistory`, `escalated`). Escalated tasks are frozen to all agents at the dispatcher layer in `llm-tools.ts`.
- **Meetings**: `caddy-agent-meeting.ts` — round-robin discussion with a `MeetingPurpose` (`redTeamReview` / `dissentSynthesis` / `signOff` / `freeform`). Structured purposes hard-cap at 2 rounds. Per-turn `[[confidence=N]]` tag drives early termination. Synthesizer emits a JSON artifact matching the purpose's schema, persisted on `AgentMeeting.structuredOutput`.
- **Handoffs**: `runHandoffCall` for shift swaps. Client↔server handoff goes through the explicit `handoffState` machine in `src/lib/agent-handoff.ts` (legal-edges table; transition helpers `markHandoffPending` / `markServerOwned` / `markReclaimPending` / `markClientRecovered` / `reconcileAfterHandoff` / `acknowledgeReconciliation`).
- **Idempotency**: auto-executed write tool calls carry an `idempotencyKey` (`${deploymentId}:${cycleStartedAt}:${toolName}:fnv1a(args)`) so client crashes and handoff boundaries can't double-write.
- **Supervisor**: `caddy-agent-supervisor.ts` — global cross-investigation analysis on a timer. Rolling retention of 200 newest notes; 3 `create_note` calls per cycle.
- **Server-Side**: `server/src/bots/caddy-agent-bridge.ts` converts profiles to BotConfig. `heartbeat-manager.ts` manages client→server handoff (30s heartbeat, 90s grace). Client-side `useServerAgents` (`src/hooks/useServerAgents.ts`) wires heartbeat success/failure into the handoff state machine. **Known gap:** the server bot manager only inserts into `bot_runs`, not `agent_actions`, so the client's "pull what the server did while away" loop currently reads an empty source — see `claudewiki/wiki/concepts/agentcaddy-server-action-gap.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peterhanily/threatcaddy](https://github.com/peterhanily/threatcaddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
