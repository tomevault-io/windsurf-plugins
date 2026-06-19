---
trigger: always_on
description: |
---


# AVP: Agent Voyager Project

AVP is an open standard for the agent-execution case, defined by four specs that compose independently:

- **Commission** ([`avp/core/spec/v0.1/commission.md`](avp/core/spec/v0.1/commission.md)): what the supervisor sends the agent at startup (model, prompt, inline managed assets, built-in allowlists).
- **Agent Descriptor** ([`avp/core/spec/v0.1/agent-descriptor.md`](avp/core/spec/v0.1/agent-descriptor.md)): what the agent advertises about itself before a run begins.
- **Trajectory** ([`avp/core/spec/v0.1/trajectory.md`](avp/core/spec/v0.1/trajectory.md)): the stream of source-tagged events the agent emits as it runs.
- **Resolver API** ([`avp/core/spec/v0.1/resolver.md`](avp/core/spec/v0.1/resolver.md)): an optional JSON-RPC service for dereferencing opaque refs. The three data-shape specs above are the common path and do not depend on it; the in-repo agents carry inline connection material on the Commission instead of dialing a resolver.

The shape of one run: supervisor sends a Commission; the agent reads it, connects any inline managed assets, runs, and emits the trajectory. **No supervisor to agent push channel.** Once the Commission is sent, the supervisor only observes.

**AVP is built on existing standards.** Every event is a [CloudEvents 1.0](https://cloudevents.io/) envelope carrying OTel span identification (`trace_id`, `span_id`, `parent_span_id`). The `data` payload uses AVP's own `avp.*` attribute namespace (`avp.usage`, `avp.tool.name`, ...); the [OpenTelemetry GenAI](https://opentelemetry.io/docs/specs/semconv/gen-ai/) conventions are NOT on the wire, but a documented `avp.*` to `gen_ai.*` projection ships for consumers forwarding into OTel-native backends. RPC payloads are [JSON-RPC 2.0](https://www.jsonrpc.org/specification). Tool descriptors are [MCP](https://modelcontextprotocol.io/)-shaped. See `FOUNDATIONS.md` for the full mapping.

## Terms

The vocabulary below is the ubiquitous language of AVP. Every doc, type, and event uses these words consistently. When you generate code or prose, match them.

**Roles**

- **Agent**: the thing that runs the loop. Drives a model, executes tools, emits events. May own the loop directly (it inlines a `while` loop over a per-turn translator) or wrap an SDK that already owns the loop (observer / translator pattern).
- **Supervisor**: the thing that issues the Commission and observes the trajectory. Never pushes mid-run messages to the agent.

**Artifacts** (the two top-level message classes)

- **Commission**: the supervisor's charter for one run. Declares what the agent should do (`prompt`, `system_prompt`, `model`) and the environment it runs in: inline supervisor-managed assets (`mcp_servers` as `McpServerHttp` / `McpServerStdio` with connection material on the wire, `skills` with inline content), plus optional `enabled_builtin_tools` / `enabled_builtin_mcp_servers` / `enabled_builtin_subagents` / `enabled_builtin_skills` allowlists over the agent's Descriptor. Sent once at startup. Type: `Commission`. Wire payload: `run_requested.data["avp.commission"]`.
- **Agent Descriptor**: the agent's self-description. Built-in tools / subagents / skills / mcp_servers, plus `agent_name`, `agent_version`, `spec_version`, `default_model`, `supported_models`, `capabilities`. Per-build, not per-run. Printed by `<agent> describe`. Type: `AgentDescriptor`. Wire payload: `agent_described.data["avp.descriptor"]`.

**Runtime concepts**

- **Run**: one execution of an agent against one Commission. Has a `run_id`. Opens with `run_requested` then `agent_described` then `agent_started`. Closes with `agent_stopped`.
- **Trajectory**: the ordered sequence of events emitted during a run. The source of truth: a non-technical reviewer reads it top-to-bottom to reconstruct what happened.
- **Event**: one CloudEvents 1.0 envelope. Ten types in v0.1, all past-tense facts under the `avp.*` namespace: `run_requested`, `agent_described`, `agent_started`, `agent_stopped`, `assistant_message`, `tool_invoked`, `tool_returned`, `subagent_invoked`, `subagent_returned`, `error_occurred`. See [`trajectory.md`](avp/core/spec/v0.1/trajectory.md) for the full catalog.
- **Turn**: one `assistant_message`, the unit of model invocation accounting. It carries the model's content (`avp.content`) plus the per-turn `avp.usage` and `avp.cost_usd`.

**Environment primitives** (declared inline in the Commission)

- **MCP server**: an inline `mcp_servers[]` entry (`McpServerHttp` with `url` or `McpServerStdio` with `command`); the agent's MCP client dials it directly and runs `tools/list` + `tools/call`. v0.1's mechanism for supervisor-side tool dispatch.
- **Skill**: an inline `skills[]` entry carrying SKILL.md content; the agent materializes it so the model can use it.
- **Built-in allowlists**: `enabled_builtin_tools` (and the `_mcp_servers` / `_subagents` / `_skills` variants) are subtractive filters over what the agent already ships in its Descriptor.

**Wire-format vocabulary**

- **The wire**: the protocol/format level. "On the wire" means "as bytes a consumer parses." Distinct from the trajectory (the logical sequence) and the audit trail (the use case).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [portofcontext/agent-voyager-project](https://github.com/portofcontext/agent-voyager-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
