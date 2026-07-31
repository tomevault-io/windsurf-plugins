---
trigger: always_on
description: Guide for implementing JS SDK GenAI AI agent framework instrumentations (LangChain, LangGraph, OpenAI Agents, Mastra, Claude Agents SDK, Google ADK, Strands Agents, LlamaIndex). Use when creating or modifying framework instrumentations in sdk/typescript/src/instrumentation/.
---


# JS SDK GenAI Framework Instrumentation Guide

This guide defines how every AI agent **framework** instrumentation in the JS SDK must be implemented. This is a companion to the **provider** instrumentation guide (`js-sdk-genai-instrumentation.mdc`). The Python SDK (`sdk/python/`) is the reference. Both SDKs must emit identical telemetry.

## CRITICAL: No Backwards Compatibility

The JS SDK has **zero backwards compatibility constraints**. You MUST:
- Rewrite existing instrumentations (LangChain, LlamaIndex) from scratch if needed
- Freely change APIs, span names, attribute keys, file structure, or behavior
- Remove old code entirely and replace with new implementations
- The **only alignment targets** are: (1) Python SDK behavior, (2) OTel GenAI semantic conventions
- Do NOT preserve any legacy JS SDK patterns or behavior

## CRITICAL: Zero Provider/Framework Dependencies

The `openlit` npm package must **never** list provider or framework packages (e.g. `@langchain/core`, `@openai/agents`, `@google/adk`) in `dependencies`. They belong in `devDependencies` only.

- All `index.ts` files must type `moduleExports` as `any` -- no static imports of framework packages.
- `wrapper.ts` / `processor.ts` must never import framework types. Duck-type everything.
- OTel's `InstrumentationNodeModuleDefinition` hooks modules by **string name** at runtime. If the user hasn't installed the framework, the instrumentation is a silent no-op.

---

## OTel Spec References

- Agent Spans: https://github.com/open-telemetry/semantic-conventions/blob/main/docs/gen-ai/gen-ai-agent-spans.md
- GenAI Spans: https://github.com/open-telemetry/semantic-conventions/blob/main/docs/gen-ai/gen-ai-spans.md
- GenAI Events: https://github.com/open-telemetry/semantic-conventions/blob/main/docs/gen-ai/gen-ai-events.md
- GenAI Metrics: https://github.com/open-telemetry/semantic-conventions/blob/main/docs/gen-ai/gen-ai-metrics.md

---

## 1. Agent Span Types (OTel Semantic Conventions)

Framework instrumentations emit these span types per the OTel agent semantic conventions:

### 1a. `create_agent` Span
- **When**: Agent is constructed/registered (e.g., `new Agent()`, `LlmAgent.__init__`, `Crew.__init__`)
- **Span name**: `create_agent {gen_ai.agent.name}`
- **SpanKind**: `CLIENT`
- **Operation**: `SemanticConvention.GEN_AI_OPERATION_TYPE_CREATE_AGENT` (`"create_agent"`)
- **Key attributes**: `gen_ai.agent.name`, `gen_ai.agent.id`, `gen_ai.agent.description`, `gen_ai.request.model`, `gen_ai.tool.definitions`, `gen_ai.system_instructions`
- **Store span context**: Save `span.spanContext()` in a registry keyed by agent name, for later Links from `invoke_agent` spans

### 1b. `invoke_agent` Span
- **When**: Agent is invoked/executed (e.g., `Runner.run()`, node execution, agent callback)
- **Span name**: `invoke_agent {gen_ai.agent.name}`
- **SpanKind**: `CLIENT` (remote agents) or `INTERNAL` (in-process agents like LangChain, LangGraph nodes)
- **Operation**: `SemanticConvention.GEN_AI_OPERATION_TYPE_AGENT` (`"invoke_agent"`)
- **Key attributes**: `gen_ai.agent.name`, `gen_ai.agent.id`, `gen_ai.conversation.id`, `gen_ai.request.model`
- **Links**: Link back to `create_agent` span context from registry when available

### 1c. `invoke_workflow` Span
- **When**: Coordinated multi-agent or multi-step workflow (e.g., graph invoke, crew kickoff, trace root)
- **Span name**: `invoke_workflow {gen_ai.workflow.name}`
- **SpanKind**: `INTERNAL`
- **Operation**: `SemanticConvention.GEN_AI_OPERATION_TYPE_FRAMEWORK` (`"invoke_workflow"`)
- **Key attributes**: `gen_ai.workflow.name`, `gen_ai.workflow.type`, `gen_ai.execution.mode`

### 1d. `execute_tool` Span
- **When**: Tool/function is executed (e.g., LangChain tool run, MCP tool call)
- **Span name**: `execute_tool {gen_ai.tool.name}`
- **SpanKind**: `INTERNAL`
- **Operation**: `SemanticConvention.GEN_AI_OPERATION_TYPE_TOOLS` (`"execute_tool"`)
- **Key attributes**: `gen_ai.tool.name`, `gen_ai.tool.type`, `gen_ai.tool.description`, `gen_ai.tool.call.arguments`, `gen_ai.tool.call.result`

### 1e. `chat` Span (LLM calls within frameworks)
- **When**: Framework intercepts an LLM call (e.g., LangChain `handleChatModelStart`)
- **Span name**: `chat {gen_ai.request.model}`
- **SpanKind**: `CLIENT`
- **Operation**: `SemanticConvention.GEN_AI_OPERATION_TYPE_CHAT` (`"chat"`)
- Follows all provider span rules from `js-sdk-genai-instrumentation.mdc`
- MUST set `frameworkLlmActive` flag to suppress duplicate provider spans

---

## 2. Span Creation Attributes

For agent/workflow spans, these attributes MUST be set at `tracer.startSpan()` time:

```typescript
{
  [SemanticConvention.GEN_AI_OPERATION]: operationName,        // "invoke_agent", "create_agent", etc.
  [SemanticConvention.GEN_AI_PROVIDER_NAME_OTEL]: aiSystem,    // framework system name
}
```

For `create_agent` and `invoke_agent` that have a model, also include:
```typescript
{
  [SemanticConvention.GEN_AI_REQUEST_MODEL]: requestModel,     // if known
  [SemanticConvention.SERVER_ADDRESS]: serverAddress,           // if known

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openlit/openlit](https://github.com/openlit/openlit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
