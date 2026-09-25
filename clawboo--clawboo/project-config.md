---
trigger: always_on
description: REST reference for the agent registry-of-record: list, create, sync, per-agent reads, files, sessions, delete, ghost cleanup, and the native shell switch.
---


REST surface for the [agent registry-of-record](/appendices/glossary). SQLite is the source of truth for who exists; an `AgentSource` syncs each upstream (the OpenClaw Gateway, the in-process native runtime) INTO SQLite. Reads serve SQLite, so the agent list, an agent record, and an agent's files keep answering even when the Gateway connection is down; a `stale` flag marks that case. Writes (create), file PUTs, and live session lists delegate to the owning source and return **503** when the source needs a live upstream that is disconnected.

<Note>
These routes return clawboo-native `AgentRecord` shapes, not OpenClaw protocol shapes. The `OpenClawAgentSource` adapts the Gateway in both directions; a native (`clawboo-native`) agent is owned by a peer source whose reads and writes are pure SQLite and always work offline. See [agent-source internals](/internals/agent-source) for the sync discipline and [agent model](/concepts/agent-model) for the registry concept.
</Note>

Per-agent routes are multi-source: each operation routes to the source that OWNS the row (its `sourceId`); an unknown id falls back to the default (OpenClaw) source so its 404 semantics hold. All POST/PUT routes read a JSON body parsed by `express.json({ limit: '2mb' })`. Static paths (`sync`, `registry/health`, `cleanup-ghosts`) are registered BEFORE `/:agentId` so the param does not swallow them.

## Routes

| Method | Path                               | Summary                                                    | Stream? |
| ------ | ---------------------------------- | ---------------------------------------------------------- | ------- |
| GET    | `/api/agents`                      | Aggregate every source's agent list from SQLite            | No      |
| POST   | `/api/agents`                      | Create an agent through its source                         | No      |
| POST   | `/api/agents/sync`                 | Manual / browser-fallback sync of the OpenClaw source      | No      |
| GET    | `/api/agents/registry/health`      | Server-side OpenClaw connection state (always 200)         | No      |
| POST   | `/api/agents/cleanup-ghosts`       | Sweep stale local OpenClaw rows not in the live set        | No      |
| GET    | `/api/agents/:agentId`             | One agent record from its source                           | No      |
| DELETE | `/api/agents/:agentId`             | Archive upstream + clean local rows                        | No      |
| GET    | `/api/agents/:agentId/files/:name` | Read one agent file                                        | No      |
| PUT    | `/api/agents/:agentId/files/:name` | Write one agent file                                       | No      |
| GET    | `/api/agents/:agentId/sessions`    | List the agent's live sessions                             | No      |
| GET    | `/api/agents/:agentId/workspaces`  | The task worktrees assigned to this agent                  | No      |
| GET    | `/api/agents/:agentId/shell`       | Whether a native Boo may ask to run commands               | No      |
| POST   | `/api/agents/:agentId/shell`       | Turn that switch on or off (native only)                   | No      |
| PATCH  | `/api/agents/:agentId/model`       | Change a native/hermes agent's model + provider (404 else) | No      |
| POST   | `/api/agents/:agentId/chat`        | Drive one 1:1 turn on a native agent (detached, 202)       | No      |
| POST   | `/api/agents/:agentId/chat/stop`   | Abort the agent's in-flight 1:1 turn                       | No      |
| GET    | `/api/agents/:agentId/chat/stream` | SSE live-tail of the agent's 1:1 chat session              | SSE     |

The `AgentRecord` shape (returned by `GET /api/agents`, `GET /api/agents/:agentId`, and inside the create `201`):

```ts
interface AgentRecord {
  // Identity
  id: string // clawboo PK (SQLite-native)
  sourceId: 'openclaw' | 'claude-code' | 'codex' | 'hermes' | string // owning source
  sourceAgentId: string // upstream id (Gateway-synced)
  // Display (merged)
  displayName: string // Boo-Zero override ▸ identity.name ▸ name ▸ id
  emoji: string | null
  avatarUrl: string | null
  avatarSeed: string | null
  // Live runtime state (Gateway-synced; may be stale when disconnected)
  status: 'idle' | 'running' | 'error' | 'sleeping' | 'archived'
  sessionKey: string | null
  isDefault: boolean // true when sourceAgentId === the source's defaultId (Boo Zero)
  // clawboo-native config (SQLite-native; preserved across re-sync)
  teamId: string | null
  personalityConfig: unknown | null
  execConfig: unknown | null
  model?: string | null // native: the AgentConfig primaryModel; absent for sources whose model lives elsewhere
  providerReady?: boolean | null // native: this agent has a runnable candidate (a key in its envVar slot, one of its fallbacks, or keyless Ollama); null/absent = not applicable
  // Classification (dormant seams)
  participantKind: 'agent' | 'human'
  runtime: 'openclaw' | 'claude-code' | 'codex' | 'hermes' | string
  capabilities: unknown | null
  tenantId: string | null // multi-tenant seam — null = single implicit tenant
  // Lifecycle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clawboo/clawboo](https://github.com/clawboo/clawboo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
