---
trigger: always_on
description: Quick reference for Agent APIs (agentstudio-hosted Agent/Application/Assistants, plugins and MCP): parameters, inputs/outputs, and error codes
---

# Agent

## Use Cases and Selection
- `dashscope.Application`: calls applications published in the Bailian console (AppBuilder/RAG/workflow/plugin apps) directly by `app_id`; no need to manage models or tools — the simplest option.
- `dashscope.agentstudio`: full lifecycle management of AgentStudio-hosted Agents (agents/sessions/environments/files/skills/deployments), with SSE event streaming, client-side tools and MCP; suitable when you need a hosted runtime and multi-turn sessions.
- `dashscope.Assistants/Threads/Runs`: OpenAI-like Assistants API. **Deprecated** (DeprecationWarning in source; migrate to the Responses API); use only for maintaining legacy code.

## SDK APIs
### agentstudio (`from dashscope.agentstudio import Client, AsyncClient`)
- `Client(api_key=, workspace=, region=None, base_url=None, uid=None, timeout=None, max_retries=2)`; if base_url is not given, `workspace` (or env var `DASHSCOPE_WORKSPACE`) is required; base_url can also be set via `DASHSCOPE_AGENTSTUDIO_URL`/`AGENTSTUDIO_URL`; default region `cn-beijing`, timeout 600s.
- Resource entry points: `client.agents / .sessions / .environments / .files / .skills / .vaults / .deployments / .deployment_runs`.
- `agents.create(*, name, model, description=None, system_prompt=None, tools=None, mcp_servers=None, skills=None, metadata=None)` → `Agent`; `retrieve(agent_id, *, version=None)` (alias `get`); `update(agent_id, *, version, ...)` (version must equal the server's current version, otherwise HTTP 409); `archive(agent_id)`; `list(*, limit, page, include_archived)` → `CursorPage`; `list_versions(agent_id)`.
- `sessions.create(*, agent, environment_id=None, title=None, resources=None, metadata=None)` → `Session`; also `retrieve/update(title, metadata)/list(*, limit, page, agent_id, statuses, created_at_gt/gte/lt/lte)/archive/delete`.
- `environments.create(*, name, config, description=None, scope="organization", metadata=None)`; `files.upload(file, *, mime_type=None, progress=None)` (file accepts a path/binary stream/(name, fileobj)); `skills.create(*, file_id=None, file=None, mime_type=None)`, `skills.versions.create(skill_id, ...)`.

### agentstudio SSE Streaming Execution (Core)
1. `client.sessions.events.send(session_id, events)`: sends an event list (at least 1 event, otherwise `ValueError`); request body is `{"input": [...]}`. Use the constructors in `types`: `user_message(text_or_blocks, *, session_thread_id=None, metadata=None)`, `user_interrupt()`, `user_tool_confirmation()`, `user_tool_result()`, `user_custom_tool_result()`, `user_define_outcome()`.
2. `with client.sessions.events.stream(session_id, *, timeout=None) as stream:` opens the SSE stream (GET `/sessions/{id}/events/stream`); iterating yields `Message` (`ServerEvent`) objects: `event.type`, `event.content` (block list), `event.stop_reason`, `event.session_status`.
3. `stream.text_stream`: yields only the text of blocks with `type=="text"` from `message` events, and stops automatically when `session_status` becomes `idle/terminated/rescheduling` — the most convenient option.
- Event types (`SSEEventType`): clients may send `message/interrupt/tool_confirmation/function_call_output/tool_call_output/define_outcome`; the server additionally sends `reasoning/tool_call/tool_call_output/mcp_call/mcp_call_output/session_status/error/thread_*/model_request_*/outcome_evaluation`, etc. Session state machine: `idle → running → (rescheduling) → idle | terminated`.

### Application (Bailian App)
`Application.call(app_id, prompt=None, history=None, workspace=None, api_key=None, messages=None, **kwargs)`; `app_id` is required (raises `InputRequired` if missing); at least one of `prompt` or `messages` is required. kwargs: `stream`, `incremental_output`, `session_id` (for multi-turn continuation, pass the session_id from the previous response), `biz_params` (flow/plugin business parameters), `has_thoughts`, `doc_tag_codes`, `doc_reference_type` (`simple`/`indexed`), `memory_id`, `image_list`, `file_list`, `rag_options`, `temperature/top_p/top_k/seed`.

### Assistants / Threads / Runs (Deprecated)
- `Assistants.call/create(*, model, name=None, description=None, instructions=None, tools=None, file_ids=[], metadata=None, workspace=None, api_key=None, top_p/top_k/temperature/max_tokens=None)`; `retrieve/list/update/delete`.
- `Threads.create(*, messages=None, metadata=None)`; `Messages.create(thread_id, *, content, role="user", file_ids=[], metadata=None)`.
- `Runs.create(thread_id, *, assistant_id, instructions=None, tools=None, stream=False, ...)`; `Runs.create_thread_and_run(*, assistant_id, thread=None, stream=False, ...)`; `Runs.retrieve(run_id, *, thread_id)`; `Runs.submit_tool_outputs(run_id, *, thread_id, tool_outputs, stream=False)`; `Runs.cancel(...)`.

## Inputs/Outputs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dashscope/dashscope-sdk-python](https://github.com/dashscope/dashscope-sdk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
