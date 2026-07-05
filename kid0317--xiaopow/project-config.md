---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**XiaoPaw (小爪子)** is a Feishu (Lark) local work assistant that uses a Skills ecosystem to give an AI agent extensible tool capabilities, with all execution isolated in AIO-Sandbox (Docker). It connects via Feishu WebSocket (no public IP needed), making it suitable for local/intranet deployment.

The design document is in `DESIGN.md` (Chinese, compressed overview). Detailed sub-module docs are in `docs/`:
- `docs/design-modules.md` — §4 模块设计（FeishuListener, Runner, Main Agent, Sub-Crew, CronService, TestAPI 等）
- `docs/design-data.md` — §5 数据设计（Session 存储、Trace、CronJob、Skill 定义、SkillLoaderTool I/O）
- `docs/design-api.md` — §6 接口设计（飞书消息收发、文件下载接口）
- `docs/design-observability.md` — §13 可观测性（日志规范、Prometheus 指标、/metrics 接口）

All code and comments should be written in Chinese where user-facing, English for code identifiers.

## Tech Stack

- **Python** (async, asyncio-based)
- **CrewAI** — Agent orchestration (Main Agent + Sub-Crew pattern)
- **lark-oapi** — Feishu SDK (WebSocket client + REST API)
- **Qwen3-max** — LLM model for agents
- **AIO-Sandbox** — Docker-based MCP server for isolated code execution
- **croniter** — Cron expression parsing for scheduled tasks
 - **prometheus_client** — Metrics export for observability
 - **AliyunLLM adapter** — Custom CrewAI `BaseLLM` implementation in `xiaopaw/llm/aliyun_llm.py` for calling Qwen via DashScope-compatible API (supports retries, function calling, multimodal image inputs)

## Architecture (Key Concepts)

**Message flow**: Feishu WebSocket → FeishuListener → SessionRouter (routing_key) → Runner → Main Agent → SkillLoaderTool → Sub-Crew (sandbox) → FeishuSender

**Three routing key types**: `p2p:{open_id}` (DM), `group:{chat_id}` (group chat), `thread:{chat_id}:{thread_id}` (topic thread)

**Two Skill types**:
- **reference** — SKILL.md content returned to Main Agent for self-reasoning
- **task** — Spawns an isolated Sub-Crew with sandbox MCP tools (4-tool whitelist: bash, code, file_ops, editor)

**Main Agent has exactly one tool**: `SkillLoaderTool` (progressive disclosure pattern). All other capabilities come through Skills.

**Credentials never enter the LLM** — written to sandbox `.config/feishu.json` at startup, read directly by Skill scripts.

## Module Layout

```
xiaopaw/
├── main.py                  # Entry: starts Listener + CronService + CleanupService + TestAPI
├── config.yaml              # Workspace config (feishu creds via env vars)
├── llm/
│   └── aliyun_llm.py        # AliyunLLM: CrewAI BaseLLM adapter for Aliyun Qwen
├── feishu/
│   ├── listener.py          # WebSocket event → InboundMessage
│   ├── downloader.py        # File/image download to session uploads/
│   ├── sender.py            # Send messages (create/reply), routing_key-aware
│   └── session_key.py       # routing_key resolution
├── api/
│   └── test_server.py       # Test API (aiohttp): simulate Feishu events, debug mode only
├── runner.py                # Core orchestrator: session → slash cmd → agent → store → send
├── agents/
│   ├── main_crew.py         # Main Crew (single SkillLoaderTool)
│   └── skill_crew.py        # Sub-Crew factory (build_skill_crew)
├── tools/
│   ├── skill_loader.py          # SkillLoaderTool (progressive disclosure + Sub-Crew trigger)
│   ├── add_image_tool_local.py  # AddImageToolLocal: local image → base64 data URL for multimodal LLM
│   ├── baidu_search_tool.py     # BaiduSearchTool: Baidu Qianfan web_search wrapper
│   └── intermediate_tool.py     # IntermediateTool: save intermediate thinking products
├── observability/
│   ├── logging_config.py        # Logging setup: console + JSON log file in data/logs/xiaopaw.log
│   ├── metrics.py               # Prometheus metrics definitions and helper functions
│   └── metrics_server.py        # Lightweight aiohttp server exposing /metrics for Prometheus
├── session/
│   ├── manager.py           # index.json + JSONL read/write
│   └── models.py            # Session / SessionEntry dataclasses
├── cron/
│   ├── service.py           # asyncio timer + mtime+size hot-reload
│   └── models.py            # CronJob / CronSchedule / CronPayload
├── cleanup/
│   └── service.py           # Storage cleanup by policy (sweep + ensure_workspace_dirs + write_feishu/baidu_credentials)
├── skills/                  # SKILL.md + scripts per skill
│   ├── pdf/                 # PDF parsing and text extraction
│   ├── docx/                # Word document processing
│   ├── pptx/                # PowerPoint processing
│   ├── xlsx/                # Excel spreadsheet processing
│   ├── feishu_ops/          # Read docs, send messages via Feishu API
│   ├── scheduler_mgr/       # Create/list/delete cron jobs (config only, not execution)
│   ├── baidu_search/        # Baidu Qianfan web search (credentials: .config/baidu.json)
│   ├── web_browse/          # Web content extraction + browser automation (browser_* tools)
│   └── history_reader/      # Paginated conversation history reader (reference skill)
└── data/                    # Runtime data (.gitignore)
    ├── sessions/            # index.json + {sid}.jsonl

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kid0317/xiaopow](https://github.com/kid0317/xiaopow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
