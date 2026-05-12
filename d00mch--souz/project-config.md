---
trigger: always_on
description: Souz is a Kotlin Multiplatform desktop AI assistant built with Compose for Desktop.
---

# Souz

Souz is a Kotlin Multiplatform desktop AI assistant built with Compose for Desktop.

## Note for LLM

Keep this file updated whenever top level details changes.
If you are not sure about something, left a note for other developers to review.

### UI architecture principles

- UI layers (Screens and Composables) should not do neither business logic, nor IO operations.
- UI-logic should be coordinated from ViewModels. ViewModel may delegate business logic to UseCases.

### Development principles

- Prefer composition to inheritance.
- Do not mix coroutines with the JVM low level concurrency primitives such as: Volatile, Synchronize, ThreadLocal, etc).
- Utilize open closed principle.

## Features

- **Graph-based agent runtime** with explicit nodes, transitions, retries, and session history.
- **Standalone ClawHub/OpenClaw skills support across `:agent` and `:runtime`**: bundle parsing, canonical hashing, safe filesystem bundle loading, per-user bundle persistence under `~/.local/state/souz/skills/`, separate file-backed validation caching under `~/.local/state/souz/skill-validations/`, LLM-based skill selection and validation, and a dedicated prompt-only skills activation node in the main agent graph between classification and MCP that injects approved skill context into history/system prompts without changing the active tool list.
- **Shared sandbox abstraction for tools and skills** in `:runtime`: local and Docker-backed sandbox filesystem/path/process contracts now sit under `runtime/src/main/kotlin/ru/souz/runtime/sandbox/`; runtime sandbox selection is now mode-driven via `SOUZ_SANDBOX_MODE=local|docker`, singleton tools resolve the active sandbox per invocation from `ToolInvocationMeta`, desktop keeps a local-default singleton scope, and backend runtime sandboxing is currently user-scoped only (no conversation-scoped backend sandboxes yet).
- **Multi-model LLM integrations** for GigaChat (REST/voice), Qwen, AiTunnel, Anthropic Claude, and OpenAI APIs.
- **Local llama.cpp provider** with a thin native bridge, strict JSON tool contract, a RAM-gated local model catalog (Qwen plus Gemma 4 chat profiles), linked local EmbeddingGemma GGUF downloads/usage for embeddings, background preload/warmup on local chat model selection, prompt-family-aware rendering (Qwen ChatML and Gemma 4 turns), prompt-prefix/KV reuse inside the native runtime, settings-driven context windows for local inference within model caps, model storage under `~/.local/state/souz/models/`, and extracted native bridge libraries under `~/.local/state/souz/native/`.
- **Shared JVM runtime layer** in `:runtime` for provider clients, config/settings access, file utilities, and backend-safe tool categories (`FILES`, `WEB_SEARCH`, `CONFIG`, `DATA_ANALYTICS`, `CALCULATOR`) reused by both desktop and backend agent execution.
- **HTTP backend agent runtime** in `:backend` with trusted-proxy `/v1/**` APIs for bootstrap, first-run onboarding state/completion, per-user settings, chats, chat rename/archive lifecycle, chat messages, option answers, execution cancellation, event replay, and per-chat WebSocket streams; stage-10 supports in-memory, filesystem, and Postgres product repositories for chats, messages, agent state, executions, options, events, and tool-calls, auto-provisions a local `users` namespace immediately after trusted proxy auth, resolves effective settings plus persisted onboarding completion per opaque trusted user identity, normalizes missing settings, legacy partial settings payloads, and invalid provider-key rows into stable bootstrap/onboarding responses, enforces ownership on chat/execution/option resources, persists explicit `AgentExecution` lifecycle for every `/v1/chats/{chatId}/messages` turn, honours effective `streamingMessages` in the shared agent runtime, writes durable request-scoped runtime/message/execution/option events into `AgentEventRepository`, stores tool call audit rows with redacted/truncated previews only, broadcasts both persisted durable events and live-only ephemeral events through an in-process event bus, keeps `MESSAGE_DELTA` live-only so it is never assigned a fake durable `seq` or returned by replay, supports capped `GET /v1/chats/{chatId}/events?afterSeq=` and capped `WS /v1/chats/{chatId}/ws?afterSeq=` replay/live flows, clamps chat/message/event list limits to hard maximums, switches initial `/messages` turns into async `running` responses only when both effective `streamingMessages` and `wsEvents` are enabled, persists heavy `Option` rows on `option.requested`, resumes the same `executionId` after `POST /v1/options/{optionId}/answer`, keeps synthetic continuation input inside backend runtime state only, uses optimistic locking on `agent_conversation_state` with `state_conflict` failure semantics, and keeps durable DB-backed event replay optional behind `SOUZ_FEATURE_DURABLE_EVENT_REPLAY`.
- **Key-aware model selection in Settings**: chat, embeddings, and voice recognition model lists are filtered by configured provider keys; invalid saved selections are normalized to available providers.
- **MCP integration** over `stdio` and `http` with OAuth discovery and token refresh support.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [D00mch/souz](https://github.com/D00mch/souz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
