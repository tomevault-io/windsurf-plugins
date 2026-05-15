---
trigger: always_on
description: FastAPI service providing pluggable primitives (memory, observability, llm, tools, identity, code_interpreter, browser, policy, evaluations, knowledge) for AI agent infrastructure. Includes a declarative agents subsystem that runs LLM tool-call loops server-side. Separate async Python client in `client/`.
---

# Agentic Primitives Gateway

FastAPI service providing pluggable primitives (memory, observability, llm, tools, identity, code_interpreter, browser, policy, evaluations, knowledge) for AI agent infrastructure. Includes a declarative agents subsystem that runs LLM tool-call loops server-side. Separate async Python client in `client/`.

## Project Structure

- `src/agentic_primitives_gateway/` — Server package
  - `main.py` — FastAPI app creation, lifespan, error handlers, router registration, UI serving
  - `middleware.py` — `RequestContextMiddleware` (extracts AWS creds, service creds, provider routing from headers)
  - `config.py` — Pydantic-settings, YAML config loading with env var expansion
  - `registry.py` — Dynamic provider loading, per-request resolution via context
  - `context.py` — Request-scoped contextvars (AWS creds, service creds, provider overrides)
  - `metrics.py` — Prometheus MetricsProxy wrapping all providers
  - `watcher.py` — Config file hot-reload watcher
  - `models/` — Pydantic request/response models and StrEnum definitions (`enums.py`)
  - `primitives/` — Abstract base classes + backend implementations per primitive; `_sync.py` provides `SyncRunnerMixin` for executor-based async wrappers
  - `routes/` — FastAPI routers, one per primitive plus health, agents, and credentials; `_helpers.py` provides `@handle_provider_errors` decorator and `require_principal()`
  - `enforcement/` — Policy enforcement layer: `base.py` (PolicyEnforcer ABC), `noop.py` (default allow-all), `cedar.py` (local Cedar evaluation via cedarpy), `middleware.py` (Starlette middleware mapping requests to Cedar principals/actions/resources)
  - `auth/` — Authentication subsystem: `base.py` (AuthBackend ABC), `models.py` (AuthenticatedPrincipal), `noop.py`, `api_key.py`, `jwt.py` (OIDC/JWKS), `middleware.py` (AuthenticationMiddleware), `access.py` (check_access, require_access)
  - `credentials/` — Per-user credential resolution subsystem: `base.py` (CredentialResolver ABC), `models.py` (ResolvedCredentials, APG_PREFIX), `noop.py` (default no-op), `oidc.py` (OIDC resolver with Admin API + userinfo fallback, convention-based apg.* mapping), `cache.py` (in-memory LRU), `middleware.py` (CredentialResolutionMiddleware), `writer/` (CredentialWriter ABC, `noop.py`, `keycloak.py` with Admin API + User Profile auto-declaration)
  - `audit/` — Governance event subsystem: `base.py` (AuditSink ABC + AuditReader protocol), `models.py` (AuditEvent + AuditAction taxonomy), `router.py` (fan-out router with per-sink async queues + graceful shutdown), `emit.py` (`emit_audit_event()` helper + `audit_mutation` context manager), `middleware.py` (emits `http.request` events), `redaction.py` (metadata + log-line secret scrubbing), `log_formatter.py` (JSON formatter + sanitization filter), `sinks/{noop,stdout_json,file,redis_stream,observability}.py`
  - `routes/_background.py` — `BackgroundRunManager` (asyncio.Task + Queue decoupling), `EventStore` ABC, `RedisEventStore`, `sse_response()` helper, `reconnect_event_generator()` for SSE reconnection
  - `agents/` — Declarative agent orchestration
    - `runner.py` — `AgentRunner` with `_RunContext` dataclass; `run()` (non-streaming) and `run_stream()` (SSE) share init/request/finalize via helpers
    - `store.py` — `AgentStore` ABC + `FileAgentStore` (JSON persistence, YAML seed with overwrite)
    - `redis_store.py` — `RedisAgentStore` + `RedisTeamStore` (Redis hash-backed, optional)
    - `session_registry.py` — `SessionRegistry` ABC + `InMemorySessionRegistry` + `RedisSessionRegistry`
    - `namespace.py` — Shared namespace resolution for agent memory (`resolve_memory_namespace` for private per-user memory, `resolve_shared_pools` for cross-user pools, `resolve_actor_id` for conversation history). Pure-function layer; runners set the corresponding contextvars.
    - `team_runner.py` — `TeamRunner` orchestrates multi-agent team execution (plan → execute → synthesize)
    - `team_store.py` — `TeamStore` ABC + `FileTeamStore`
    - `team_agent_loop.py` — `run_agent_with_tools()` and `run_agent_with_tools_stream()` with `invocation_id` tracking for per-invocation token attribution
    - `base_store.py` — Generic `SpecStore[T]`, `FileSpecStore[T]`, `RedisSpecStore[T]` base classes; agent/team stores inherit from these
    - `checkpoint.py` — `CheckpointStore` ABC, `RedisCheckpointStore`, `ReplicaHeartbeat` (heartbeat + orphan scanning), `recover_orphaned_runs()`, `_recovery_tasks` tracking
    - `checkpoint_utils.py` — `serialize_auth_context()`, `restore_auth_context()`, `apply_provider_overrides()`, `restore_provider_overrides()` — shared between AgentRunner and TeamRunner
    - `tools/` — Tool system package
      - `handlers.py` — Handler functions per primitive (memory, browser, code_interpreter, tasks, tools, identity). Handlers read per-primitive context (memory namespaces, session IDs, team_run_id, agent_role, shared pools) from contextvars set by the runner — not from positional arguments.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentic-community/agentic-primitives-gateway](https://github.com/agentic-community/agentic-primitives-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
