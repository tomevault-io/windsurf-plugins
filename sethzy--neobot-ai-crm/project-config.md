---
trigger: always_on
description: You are an expert in Next.js, Anthropic Managed Agents, Vercel AI SDK, and Supabase. Our database is hosted on Supabase. Our serverless functions and frontend deployment are on Vercel.
---

You are an expert in Next.js, Anthropic Managed Agents, Vercel AI SDK, and Supabase. Our database is hosted on Supabase. Our serverless functions and frontend deployment are on Vercel.

> ## ⚠️ CRITICAL: ALWAYS USE HAIKU FOR TESTING IN MANAGED AGENTS
>
> **When testing anything in Managed Agents, ALWAYS use `claude-haiku-4-5` (or the latest Haiku).**
> **NEVER use Sonnet or Opus for testing.** Sonnet/Opus are reserved for production runs only.
> This applies to all local test scripts, eval runs, and any ad-hoc agent invocations during development.

## Market

**NeoBot** is an autopilot for solo practitioners in advisory sales — real estate agents, insurance advisors, financial planners, and similar client-facing roles.

We don't sell a tool. We sell the work done: CRM updated, follow-up sent, briefing prepared, inbound handled. Every improvement in the underlying model makes NeoBot faster and cheaper — it doesn't commoditize us.

Advisory sales sits in the **autopilot quadrant**: high intelligence-to-judgement ratio, work already partially outsourced to VAs and assistants. NeoBot is a vendor swap, not a reorg. The practitioner keeps the judgement (which deal to pursue, how to handle a tricky client). NeoBot handles the intelligence work (data entry, scheduling, drafting, research).

**Why NeoBot wins over time:**

- **Compounding memory is the data moat.** Per-client files (SOUL.md, USER.md, MEMORY.md) accumulate proprietary context with every run. This is not replicable by a better model — it's earned one outcome at a time.
- **The approval gate is the human backstop.** Internal work auto-runs. External-facing actions require user approval. This is the autonomy slider — we dial it up as trust compounds.
- **Model improvements accelerate us.** The harness (runner + tools + context engineering) is the constant. Better models make every run more capable without changing our architecture.

User activates in <10 minutes, useful from day 1 via web chat. Both desktop and mobile.

## Architecture

NeoBot is a general agent harness: a looping runner with tools that operates on behalf of the user.

- **Runner engine:** Anthropic Managed Agents. The agent definition (system prompt, tools, skills) is registered on Anthropic's platform via `scripts/managed-agents/create-agent.ts`. At runtime, a **session** is created per thread, an SSE event stream is opened, and a `user.message` kickoff is sent. The runner consumes the stream, dispatches `agent.custom_tool_use` events to local tool handlers, and posts results back as `user.custom_tool_result`. The agent loop itself (context management, caching, compaction, multi-step) runs on Anthropic's infrastructure. Entry point: `consumeAnthropicSession()` in `src/lib/managed-agents/session-runner.ts`.
- **Tools:** Custom tool declarations in `src/lib/managed-agents/tools/`. Registered on the agent at deploy time. At runtime, `dispatcher.ts` routes `agent.custom_tool_use` events to the matching handler. Tool response shape: `{ success: true, entity } | { success: false, error }`.
- **Memory system:** Per-client files in Supabase Storage (`SOUL.md`, `USER.md`, `MEMORY.md`, `memory/*.md`). Agent reads/writes via `storage_read`/`storage_write` tools. This is the compounding data layer.
- **Context assembly:** System prompt baked into the agent definition at registration time (`create-agent.ts`). Per-run dynamic context (client profile, CRM state, memory) is injected via the kickoff `user.message`, not reassembled in the runner on every request.
- **Safety model:** Two tiers. Internal work auto-runs. External-facing actions require approval via `agent.requires_action` → `user.tool_confirmation` round-trip. No per-action granularity — binary internal/external.
- **Thread serialization:** Chat still runs one active run per thread. Automation executions run in dedicated run threads linked back to the parent automation.
- **Automations:** Scheduled automations claim due trigger rows, start a fresh Anthropic Managed Agent run, and skip if that automation is already running.
- **Daily Orchestrator:** New clients get one seeded scheduled automation (`Daily Orchestrator`) that appears in the normal Automations list and runs through the same trigger path as any other schedule automation.
- **Improvement loop:** Langfuse traces instrument every run. Evals score tool-call correctness and safety. The feedback cycle is: run → trace → evaluate → improve context engineering → run again.
- **Tenant isolation:** `clientId` injected into tool closures + RLS double-lock on every table.
- **Model routing:** Main agent model is `claude-sonnet-4-6`, pinned by `ANTHROPIC_AGENT_VERSION`. Gemini models (via Vercel AI Gateway) are used only for cheap helpers: title generation (`google/gemini-3-flash`) and thread compaction (`google/gemini-2.5-flash-lite`).

## Capabilities

What the agent can do today (shipped):

### Agent Tools
- **CRM:** Create, read, update, delete, search, link/unlink across people, companies, deals, tasks. Configurable vocabulary + custom fields. Schema introspection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sethzy/neobot-ai-crm](https://github.com/Sethzy/neobot-ai-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
