---
trigger: always_on
description: AI-native workflow engine for GTM/RevOps automation.
---

# crayon

AI-native workflow engine for GTM/RevOps automation.

## Project Structure

Monorepo using pnpm workspaces:

```
crayon/
├── packages/
│   ├── core/              # Main SDK + CLI + MCP server + Dev UI (published as `crayon`)
│   ├── ui/                # React UI components (@crayon/ui)
│   └── auth-server/       # Next.js OAuth server (private, Nango-based)
├── skills/                # Claude Code skills
├── examples/uptime-app/   # Example app using crayon
├── docs/plans/            # Design documents
└── scripts/               # MCP server launcher
```

## Claude Code Plugin

This repo is a Claude Code plugin. Load it with:
```bash
claude --plugin-dir /path/to/crayon
```

### Available Skills

- `/crayon:create-workflow` - Collaborative workflow design (guides through creating workflows with embedded descriptions)
- `/crayon:refine-node` - Refine node definitions (adds tools, guidelines, typed Zod schemas to nodes)
- `/crayon:compile-workflow` - Update workflow implementation from embedded descriptions
- `/crayon:integrations` - Generate integration nodes for external APIs (Salesforce, HubSpot, etc.)
- `/crayon:deploy` - Deploy a crayon app to the cloud. Verifies deployment files, sets up environment, and deploys.

### MCP Tools

16 tools exposed via MCP (crayon-sandbox):

- `listIntegrations` / `listConnections` / `getConnection` / `assignConnection` - OAuth connection management
- `listWorkflows` / `runWorkflow` / `runNode` - Workflow execution
- `listRuns` / `getRun` / `getTrace` - Run history and tracing
- `createVersion` / `listVersions` / `restoreVersion` - Version management (git-backed)

## Architecture Overview

**Workflow code with embedded descriptions** → **Compiler** (Claude Code skill) → **Updated implementation** → **DBOS runtime**

- Descriptions embedded in code as `description` fields (workflow-level for flow, node-level for details)
- Agent specs (`src/crayon/agents/*.md`) are markdown files colocated with agent code, used as runtime system prompts
- No separate spec files for workflows — the code IS the spec
- DBOS provides durability: workflows register as DBOS workflows, nodes run as DBOS steps

### Node Types

| Type | Location | Example |
|------|----------|---------|
| Built-in | `crayon` package | `webRead` |
| User node | `src/crayon/nodes/` in app | Custom logic functions |
| Agent | `src/crayon/agents/` in app (.ts + colocated .md spec) | AI reasoning via Vercel AI SDK |

### App Template Structure (scaffolded by `crayon init`)

```
my-app/
├── src/
│   ├── crayon/
│   │   ├── workflows/       # Compiled workflows (checked into git)
│   │   ├── nodes/           # User-defined function nodes
│   │   ├── agents/          # Agent .ts files + colocated .md specs
│   │   ├── tools/           # Agent tool implementations
│   │   ├── integrations/    # External API SDKs (Salesforce, etc.)
│   │   └── generated/       # Auto-generated (registry.ts)
│   ├── lib/crayon.ts        # crayon singleton
│   └── ...                  # Rest of Next.js app
└── dbos-config.yaml         # DBOS runtime config
```

## Test Mode (Side-Effect Safety)

Nodes with side effects (sending emails, Slack messages, updating databases) support a **test mode** where they describe what they would do without actually performing the action. This is controlled via `ctx.testMode` on `WorkflowContext`.

### Defaults by trigger

| Trigger | testMode default | Rationale |
|---------|-----------------|-----------|
| CLI `workflow run` / `node run` | ON (`--live` to disable) | Interactive dev |
| MCP tools `run_workflow` / `run_node` | ON (`test_mode: false` to disable) | Claude experimenting |
| HTTP API (webhooks) | OFF (`test_mode: true` to enable) | Production triggers |
| Cron jobs | OFF (pass `test_mode: true` in job input to enable) | Scheduled automation |

### How nodes use it

Side-effect nodes check `ctx.testMode` and skip the actual action when true. The output schema is the same in both modes — always includes action details (what was sent, to whom) plus a `testMode: boolean` field.

```typescript
execute: async (ctx, inputs) => {
  const message = `Hello ${inputs.name}`;
  if (!ctx.testMode) {
    await slack.postMessage({ channel: inputs.channel, text: message });
  }
  return { messageSent: message, channel: inputs.channel, testMode: ctx.testMode };
},
```

### Side-effect node descriptions

Nodes with side effects use an extended description format with `**Side Effect:**` and `**Test Mode:**` tags. These are documented in the skill files (`create-workflow`, `refine-node`, `compile-workflow`).

## Key Source Paths (packages/core/src/)

### SDK Core
- `index.ts` - Public API exports
- `factory.ts` - `createCrayon()` factory
- `workflow.ts` - `Workflow.create()`, WorkflowContext, DBOS integration
- `node.ts` - `Node.create()` for function nodes
- `agent.ts` - `Agent.create()` for AI agents
- `types.ts` - Executable interface, WorkflowContext, CrayonConfig
- `registry.ts` - Workflow/agent/node registry
- `discover.ts` - Auto-discovery from project directories

### Agent Execution
- `nodes/agent/executor.ts` - Agent execution with Vercel AI SDK (`generateText`)
- `nodes/agent/parser.ts` - Parse agent spec markdown files
- `nodes/agent/model-config.ts` - Model provider config (OpenAI, Anthropic)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timescale/crayon](https://github.com/timescale/crayon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
