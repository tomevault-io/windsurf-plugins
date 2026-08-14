---
trigger: always_on
description: This project aims to stay aligned with the upstream OpenCode client architecture and interaction patterns from `~/opencode`, especially the web/app implementation.
---

# OpenCode iOS Client Notes

This project aims to stay aligned with the upstream OpenCode client architecture and interaction patterns from `~/opencode`, especially the web/app implementation.

## Goal

Build a native iOS client that follows upstream OpenCode behavior closely enough that future improvements can be guided by the existing OpenCode app patterns rather than ad hoc client-specific behavior.

Priority areas:

- One shared SSE/event pipeline
- Typed event handling
- Centralized bootstrap/hydration
- Project -> Session -> Chat navigation
- Session-local live state like todos/messages
- First-party UI for permissions, questions, todos, and tool activity

## Upstream Reference Files

These local upstream files were identified as the main references for architecture and behavior:

- `~/opencode/packages/app/src/context/global-sdk.tsx`
- `~/opencode/packages/app/src/context/global-sync.tsx`
- `~/opencode/packages/app/src/context/global-sync/bootstrap.ts`
- `~/opencode/packages/app/src/context/global-sync/event-reducer.ts`
- `~/opencode/packages/app/src/context/sync.tsx`
- `~/opencode/packages/sdk/js/src/v2/gen/types.gen.ts`

## Confirmed Upstream Patterns

### Shared event pipeline

Upstream uses one shared SSE/global event owner and fans events out by `directory`.

Relevant upstream file:

- `packages/app/src/context/global-sdk.tsx`

Current iOS direction:

- `OpenCodeIOSClient/API/OpenCodeEventManager.swift`

### Typed event models

Upstream uses generated discriminated event types in the SDK.

Relevant upstream file:

- `packages/sdk/js/src/v2/gen/types.gen.ts`

Current iOS direction:

- `OpenCodeIOSClient/Models/OpenCodeModels.swift`
  - `OpenCodeTypedEvent`
  - `OpenCodeEventEnvelope`
  - `OpenCodeGlobalEventEnvelope`

### Reducer-style event application

Upstream applies global and directory/session events through reducer helpers rather than scattering event mutation in views.

Relevant upstream file:

- `packages/app/src/context/global-sync/event-reducer.ts`

Current iOS direction:

- `OpenCodeIOSClient/Models/OpenCodeStateReducer.swift`

### Centralized bootstrap

Upstream bootstraps global state and directory/session state separately.

Relevant upstream file:

- `packages/app/src/context/global-sync/bootstrap.ts`

Current iOS direction:

- `OpenCodeIOSClient/API/OpenCodeBootstrap.swift`

## What We Learned About the Server

### Projects

- `GET /project/current` works
- `GET /project` returns known projects
- Non-global projects are scoped by `directory`
- `global` is special:
  - bare `GET /session` returns global sessions
  - `GET /session?directory=/` does not
- Project discovery appears implicit from directory selection/worktree state
- `PATCH /project/:id?directory=...` updates an existing project, but does not create one
- Project id for git repos appears tied to the repo's first commit hash
- Web flow appears to use directory search + session roots query + SSE `project.updated`

### Sessions

- Scoped session list: `GET /session?directory=...`
- Scoped session creation: `POST /session?directory=...`
- Discovery/warm-up pattern observed: `GET /session?directory=...&roots=true&limit=55`

### Todos

- Source of truth is `GET /session/:id/todo`
- Todo tool message detail is useful for context/debugging, but should not be treated as the canonical todo state
- Hide todo strip when all items are `completed`

### Permissions

Important corrections discovered during implementation:

- Permissions are not driven by `/tui/control/*`
- Correct live event: `permission.asked`
- Correct reply endpoint: `POST /permission/:requestID/reply`
- Initial hydration endpoint: `GET /permission`
- Actual permission list payload shape differs from earlier assumptions:
  - `permission`
  - `patterns`
  - `always`
  - `metadata`
  - `sessionID`
  - `tool.messageID`
  - `tool.callID`

### Questions

- Initial hydration endpoint: `GET /question`
- Reply endpoint: `POST /question/:requestID/reply`
- Reject endpoint: `POST /question/:requestID/reject`
- Live events:
  - `question.asked`
  - `question.replied`
  - `question.rejected`
- Upstream question payloads can omit defaultable fields:
  - `multiple` may be missing and should default to `false`
  - `custom` may be missing and should default to `true`
- Question live-event decoding is easy to break if the generic event envelope model does not carry question-specific fields like `questions`

### Streaming

Important streaming findings:

- The client now receives raw SSE payloads correctly
- On-device event framing required parser adjustments beyond naive blank-line assumptions
- Upstream reducer behavior for `message.part.delta` is simple append-to-field on an existing typed part
- Reasoning vs answer text should be determined by the server-provided part `type` (`reasoning` vs `text`), not by parsing streamed text content
- Deltas that arrive before the corresponding `message.part.updated` should be ignored rather than creating a placeholder `text` part, so reasoning streams are never misclassified as answer text
- iOS still preserves accumulated text when a later empty `part.updated` would otherwise wipe it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ntoporcov/openclient](https://github.com/ntoporcov/openclient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
