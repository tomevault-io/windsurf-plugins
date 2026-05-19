---
trigger: always_on
description: For AI coding agents working on this repo. The README covers the
---

# AGENTS.md

For AI coding agents working on this repo. The README covers the
**product**; this file covers the **internals and conventions** an
agent needs to avoid breaking invariants. Don't restate the README — if
it's there, link to it.

## Status notes (read first)

- The multi-agent layer landed in v0.2.0: capture both `/v1/messages`
  (Anthropic Messages API) and `/v1/responses` (OpenAI Responses API)
  through the same project/message/interaction model. See `Multi-agent
  architecture` below before touching `proxy.ts`, `grouping.ts`, or
  `aggregate.ts`.
- Projects are keyed by **(cwd, agent)** since v0.2.0. `projectId =
  sha256(cwd \0 agent).slice(0,16)`. Two agents in the same cwd produce
  two distinct projects. Pre-0.2.0 single-cwd files are re-keyed (or
  split per-agent for mixed-agent files) by a one-shot migration at
  `Storage` boot — `maybeReKeyByAgent` in `storage.ts`.
- Action segments are computed for **both** agents. Anthropic pairs
  `tool_use` ↔ `tool_result`; Codex pairs `function_call` ↔
  `function_call_output` (also `custom_tool_call*` variants) — see
  `computeActionSegmentsResponses` in `aggregate.ts`. Codex shell-tool
  output is JSON-envelope-unwrapped for the preview.
- `agentmind-cli` defaults to launching **Claude Code** as of v0.2.1.
  Old dashboard-only behaviour is behind `--no-agent`. The launcher's
  dispatch logic lives in `bin/cli.js`.
- Framework-injected user-input chrome is parsed into named kinds
  (`skills`, `memory`, `agents-md`, `environment`, `permissions`,
  `tools`, `date`, …) since v0.2.3 — the classifier lives in
  `src/lib/framework-blocks.ts` and both REQUEST panels render kind-
  specific chips via `FrameworkBlockChip`. Claude wraps each injection
  in `<system-reminder>` and concatenates them into the prompt
  message; Codex splits them across distinct `input_text` blocks. The
  classifier handles both with one rule table — add new patterns
  there, not in the panels.
- `skills` and `memory` are promoted to top-level Sections — listed
  in `PROMOTED_KINDS` (framework-blocks.ts), rendered via
  `SkillsSection` / `MemorySection` (FrameworkSections.tsx) at the
  same hierarchy as `tools` / `instructions`. The first occurrence in
  the request wins (both agents re-inject identical content on every
  iter). In-message chips for these kinds are suppressed to avoid
  duplication. To promote a new kind: add it to `PROMOTED_KINDS`,
  build a corresponding section component, and wire it into both
  panels alongside the existing two.

## Core data model

```
project (cwd, agent) ─ single-agent by construction
  └── message (one user prompt)
        └── interaction (one HTTP round-trip)
              ├── request / response   ← shape varies by agentType
              └── action segment       ← both agents now
```

- **project** — keyed by `(cwd, agent)`, no idle window: every request
  from the same agent in the same working directory, across runs and
  days, lands in the same project. `projectId = sha256(cwd \0
  agent).slice(0,16)` — see `src/server/projectId.ts`. Two different
  agents in the same cwd produce two distinct projects with two
  distinct message chains. `primaryAgent` on the project record is
  always the same agent the projectId hash includes; per-interaction
  `agentType` is redundant under the new scheme but kept for read-side
  compatibility with pre-0.2.0 records. Helper calls that don't carry
  their own cwd attach to the most recent cwd **for that same agent**
  the proxy saw (per-agent `lastCwd`, not global) — otherwise a Codex
  helper would accidentally land in a recent Claude project.
- **message** — opens when the request's transcript (Anthropic
  `messages` / Responses `input`, normalised through the protocol
  adapter) isn't a prefix-extension of any existing message in the
  project, OR the appended slice contains a new user-typed prompt.
- **interaction** — one HTTP round-trip. An N-step ReAct loop = N
  interactions on the same message. Carries `agentType` ∈
  `'claude-code' | 'codex-cli' | 'unknown'`. Pre-0.2.0 records lack
  this field; readers default to `'claude-code'`. Mixed-agent legacy
  files are split at `Storage` boot so production code never sees a
  multi-agent file.
- **action segment** — reconstructed by pairing iter N's tool-call
  blocks with iter N+1's tool-result blocks. Anthropic pairs
  `tool_use`/`tool_result` (`computeActionSegments`); Codex pairs
  `function_call`/`function_call_output` plus the `custom_tool_call*`
  variants (`computeActionSegmentsResponses`). Codex `shell` output is
  JSON-envelope-unwrapped (`{output, metadata:{exit_code}}` → preview =
  `output`, non-zero exit = `isError`). The gap duration is the local
  tool-execution wall-clock.

Neither protocol carries a project/session header — everything above
is **inferred** from request shape, by the protocol adapter. Read the
comments in `adapters.ts` + `grouping.ts` before touching the
inference rules.

## Multi-agent architecture

The proxy must speak both Anthropic Messages API and OpenAI Responses
API. The split lives behind `src/server/adapters.ts`:

```
ProtocolAdapter:
  agentType        'claude-code' | 'codex-cli'
  endpointPath     '/v1/messages' | '/v1/responses'
  parseRequest()   Buffer → typed request | undefined

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imsobear/agentmind](https://github.com/imsobear/agentmind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
