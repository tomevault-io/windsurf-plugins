---
trigger: always_on
description: > This document is intended for **AI Agents** (and developers) who read or analyze this repository. The goal is to help the reader build a mental model as quickly as possible: what this service does, how the code is layered, where each kind of logic lives, and how a request flows through its lifecycle.
---

# AIGateway Service

> This document is intended for **AI Agents** (and developers) who read or analyze this repository. The goal is to help the reader build a mental model as quickly as possible: what this service does, how the code is layered, where each kind of logic lives, and how a request flows through its lifecycle.

## 1. One-Sentence Summary

**AIGateway is an OpenAI-compatible (and Anthropic Messages) AI inference gateway**: it exposes a unified `/v1/*` API externally and internally handles "model resolution → protocol routing → reverse proxy to upstream inference services → usage/billing/LLM log/trace collection and accounting." It is the entry point for external clients calling AI models on the CSGHub platform.

Service directory: `aigateway/`; entry point: `cmd/csghub-server/cmd/aigateway/launch.go` (start with `go run -tags=saas cmd/csghub-server/main.go aigateway launch --config=common/config/local.toml`).

---

## 2. Directory Structure & Responsibilities

The layering follows the repository-wide convention (`handler → component → builder`), but AIGateway has its own extensions.

| Directory | Responsibility | Key Files / Types |
|---|---|---|
| `router/` | HTTP route registration, middleware wiring | `router/aigateway.go` is the **single** `/v1/*` route table (see §5) |
| `handler/` | HTTP handler layer: request parsing, protocol adaptation, reverse proxy, response transformation, recording | `handler/openai.go` (main handler, 1100+ lines) |
| `handler/plan/` | **Three-stage pipeline skeleton** (Extract → Plan → Execute), protocol-agnostic | `interfaces.go`, `orchestrator.go`, `planner.go` |
| `handler/protocol/` | Protocol route resolver (Native / Adapter / Disabled) | `adapt.go`'s `adapterMatrix` |
| `handler/anthropic/` | Anthropic Messages API (`/v1/messages`) implementation | `handler.go`, `to_chat_adapter.go`, `to_responses_adapter.go`, `native.go` |
| `handler/responses/` | Responses API helper sub-package (routing, llmlog normalization, ID mapping, sensitivity handling) | `responses_routing.go`, `responses_id_mapper.go` |
| `handler/streamdecoder/` | SSE stream decoder | `stream_decoder.go` |
| `component/` | Business logic layer (model management, usage, sensitivity, LLM logging) | `openai.go` (model resolution), `usage_limiter.go`, `safety_policy.go`, `moderation.go`, `llmlog_*.go` |
| `component/router/` | Upstream session routing, upstream catalog normalization | `session_router.go`, `upstream_catalog.go` |
| `component/adapter/` | Multimodal provider adapters (text2image / text2video / audio / ocr) | `*_adapter.go` + provider files in each sub-directory |
| `component/availability/` | Upstream health check, circuit breaking, state caching | `health_checker.go`, `circuit_breaker.go`, `availability_manager.go` |
| `component/metrics/` | Request metrics collection (Prometheus / DB sink) | `collector_ee.go`, `sink_ee.go` |
| `component/trace/` | LLM tracing (Sigil tracer) | `llm_tracer.go`, `sigil.go` |
| `token/` | Token counting (usage/billing) | `token_counter.go`, `*_token_counter.go`, `tokenizer_*.go` |
| `task/` | Async generation task (video) polling/billing orchestration | `orchestrator.go`, `metering.go`, `service.go` |
| `task/processor/` | Async task resource processor interface + implementations (video) | `processor.go`, `video/video.go` |
| `types/` | Service-internal shared data structures (protocol, request/response, model, trace) | see §4 |
| `middleware/` | Metrics middleware (EE) | `metrics_ee.go` |
| `http/response/wrapper/` | Response body wrapper/transformer (image, ocr) | `wrapper/image.go`, `wrapper/ocr.go` |

### Build Tags (CE / EE / SAAS)

The repository has three build variants, distinguished by **Go build tags**. Files typically end with `_ce.go` / `_ee.go` / `_saas.go` or use `//go:build` annotations:

- **CE** (Community Edition): `//go:build !ee && !saas`
- **EE** (Enterprise Edition): `//go:build ee` (or `ee || saas`)
- **SAAS**: `//go:build saas`

Typical examples:
- `component/llmlog_capture_ce.go` vs `component/llmlog_capture_ee.go` (LLM training logs only enabled in EE)
- `component/openai_model_filter_{ce,ee,saas}.go` (model filtering logic differs per variant)
- `handler/metrics_helpers_{ce,ee}.go`, `middleware/metrics_ee.go` (metrics only compiled in EE/SAAS)
- `router/api_{ce,ee}.go` (`extendRoutes` extends routes per variant)
- `handler/mcp_*.go`, `handler/agent_ee.go`, `handler/sandbox_ee.go` etc. (MCP / Agent / Sandbox are EE features)

> **Search tip**: When a feature appears to have "per-variant implementations," search for `_ce/_ee/_saas` variants of the same file name first. Interfaces are typically defined in non-suffixed files, with implementations spread across suffixed files.

---

## 3. Core Architecture

### 3.1 Three-Stage Pipeline (Standard Pattern for New Protocols)

`handler/plan/` defines a unified three-stage flow: **protocol-agnostic Planner + protocol-specific Handler** combination:

```
Extract (protocol-specific)  →  Plan (protocol-agnostic)  →  Execute (protocol-specific)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenCSGs/csghub-server](https://github.com/OpenCSGs/csghub-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
