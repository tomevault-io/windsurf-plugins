---
trigger: always_on
description: > 本文档面向后续维护本项目的 AI 智能体，完整阐述项目的实现原理、协议细节、代码架构和已知问题。**请先全文阅读本文档再开始任何修改**。
---

# AGENTS.md — DeepSeek Chat API Proxy (Preview)

> 本文档面向后续维护本项目的 AI 智能体，完整阐述项目的实现原理、协议细节、代码架构和已知问题。**请先全文阅读本文档再开始任何修改**。

---

## v2.2.0 变更摘要（2026-06）

新增模块：`logger.py`、`crypto.py`、`ip_utils.py`、`token_counter.py`、
`rate_limiter.py`、`model_router.py`、`session_cache.py`。

主要变化：

- **安全**：默认 `HOST=127.0.0.1`；公网 + 默认密码启动时 `SystemExit(2)`；结构化 JSON 日志；`data/accounts.json` 可选 Fernet 加密（启动时透明迁移）；CORS 严格化（空 = 同源）；XFF 白名单 (`TRUSTED_PROXIES`)；安全响应头。
- **Bug**：`adapter.chat()` 现在返回 `(content, thinking)` 元组；`anthropic_format.build_nonstream_response` 接收 `thinking_text`；`StreamSieve` 改为迭代式排空、`feed("")` 递归被替换为 `while`、捕获缓冲硬上限 1 MB（可配置 `DSML_MAX_BUFFER_BYTES`）；账号错误达到 3 次后自动后台 `check_health` 恢复。
- **新功能**：`MODEL_ROUTES` 模型路由；`SESSION_CACHE_TTL` 多轮会话（OpenAI `X-Conversation-Id` 头 / Anthropic `metadata.user_id` / 消息摘要 fallback）；`CLIENT_RPM_PER_KEY` / `CLIENT_RPM_PER_IP` 双维度滑动窗口限流；`usage` 字段返回真实 token 数（tiktoken cl100k_base，缺失时字符启发式）；`/admin/api/stats` 增加 p50/p95/p99 + 成功率。
- **工程化**：`tests/test_*.py` 56 个 pytest 用例；CI 矩阵 (3.10/3.11/3.12)；SSE 错误帧（OpenAI 流末尾发 `data: {"error":...}` 后 `data: [DONE]`）替代直接抛 500。

---

## 目录

1. [项目概述](#1-项目概述)
2. [目录结构](#2-目录结构)
3. [核心数据流](#3-核心数据流)
4. [PoW 反爬机制](#4-pow-反爬机制)
5. [会话管理](#5-会话管理)
6. [SSE 协议格式](#6-sse-协议格式)
7. [专家模式 (Expert Mode)](#7-专家模式-expert-mode)
8. [工具调用 (DSML)](#8-工具调用-dsml)
9. [StreamSieve 流式筛分引擎](#9-streamsieve-流式筛分引擎)
10. [OpenAI 兼容层](#10-openai-兼容层)
11. [Anthropic 兼容层](#11-anthropic-兼容层)
12. [多轮对话与消息组装](#12-多轮对话与消息组装)
13. [ContentPart 支持](#13-contentpart-支持)
14. [配置系统](#14-配置系统)
15. [账号池 (AccountPool)](#15-账号池-accountpool)
16. [管理后台 (Admin API)](#16-管理后台-admin-api)
17. [关键算法细节](#17-关键算法细节)
18. [已知限制与边界情况](#18-已知限制与边界情况)
19. [常见调试方法](#19-常见调试方法)
20. [协议变更预警](#20-协议变更预警)

---

## 1. 项目概述

本项目将 DeepSeek Chat 网页版（https://chat.deepseek.com）反向代理为 OpenAI 兼容 API。

### 1.1 解决的问题

- DeepSeek 官方 API 需要付费且对非中国大陆用户受限
- DeepSeek Chat 网页版免费但使用其私有协议，不兼容 OpenAI SDK
- 网页版有 PoW（Proof of Work）反爬机制，需要自动求解
- DeepSeek 提供"快速模式"和"专家模式"两种对话能力，接口行为不同

### 1.2 核心能力

| 能力 | 状态 | 说明 |
|------|------|------|
| OpenAI 兼容接口 | ✅ | `/v1/chat/completions`, `/v1/models`, `/health` |
| Anthropic 兼容接口 | ✅ | `/v1/messages` — Anthropic Claude API 格式 |
| 流式输出 (SSE) | ✅ | OpenAI chunk 格式 + Anthropic SSE 格式 |
| 非流式输出 | ✅ | 完整响应 |
| 多轮对话 | ✅ | 客户端通过 messages 数组管理上下文 |
| 普通对话 | ✅ | 快速模式 |
| 专家模式 | ✅ | DeepSeek 推理模型，含 thinking tokens |
| 工具调用 | ✅ | 基于 DSML 提示词注入（非原生） |
| ContentPart 数组 | ✅ | content 支持 str 和 array 格式 |
| PoW 自动求解 | ✅ | WASM 本地求解，0.1-0.3s |
| 会话管理 | ✅ | 每次请求新建 DeepSeek 会话 |
| 联网搜索 | ⚠️ | 参数透传，行为取决于 DeepSeek 服务端 |
| MODE 环境变量控制 | ✅ | 通过 `.env` 强制模式：`auto`/`quick`/`expert` |
| THINKING 环境变量控制 | ✅ | 通过 `.env` 强制思考：`auto`/`enabled`/`disabled` |
| SEARCH 环境变量控制 | ✅ | 通过 `.env` 强制联网搜索：`auto`/`enabled`/`disabled` |
| model_type 与 thinking_enabled 解耦 | ✅ | 两者独立，所有 4 种组合均有效 |
| finish_reason SSE 结束帧 | ✅ | 流式响应末尾发送 `delta: {}, finish_reason: "stop"` |

### 1.3 项目版本

- 开源版：`v2.0.0` — 基础功能 + 工具调用
- 预览版 (--pre)：`v2.2.0-pre` — 基础功能 + 工具调用 + 专家模式 + 联网搜索 + 可配置端口

---

## 2. 目录结构

```
D:\the llaa\DS反代 --pre\
├── server.py              # FastAPI 服务入口（路由、请求模型、SSE 组装）
├── adapter.py             # DeepSeek API 适配器（PoW、会话、SSE 解析）
├── anthropic_format.py    # Anthropic /v1/messages 格式转换层
├── account_pool.py        # 多账号池管理（CRUD、状态追踪、轮询分配、健康检查）
├── admin.py               # 管理后台 API（密码认证、请求统计、账号池管理）
├── tool_dsml.py           # DSML 格式解析器（工具调用协议）
├── tool_sieve.py          # 流式筛分引擎（实时检测工具调用标签）
├── sha3_wasm_bg.wasm      # PoW 哈希引擎 WASM 二进制
├── start.bat              # Windows 一键启动脚本
├── webui/                 # 管理面板前端（纯静态 SPA，零 build 依赖）
│   ├── index.html
│   ├── app.js
│   └── style.css
├── .env                   # 环境配置（含 TOKEN/COOKIE，不提交）
├── .env.example           # 配置模板（提交到仓库）
├── requirements.txt       # Python 依赖
├── README.md              # 用户文档
└── AGENTS.md              # 本文件
```

### 2.1 文件详细说明

#### 核心模块

| 文件 | 职责 | 不负责 |
|------|------|--------|
| `server.py` | HTTP 路由、请求/响应模型、OpenAI 格式组装、工具调用协调 | PoW 求解、原生 SSE 解析 |
| `adapter.py` | 与 DeepSeek API 通信、PoW 求解、SSE 流解析、会话创建 | HTTP 路由、OpenAI/Anthropic 格式 |
| `anthropic_format.py` | Anthropic `/v1/messages` 请求解析、响应组装、SSE 生成 | DeepSeek 协议、HTTP 路由 |
| `account_pool.py` | 多账号 CRUD、状态追踪（idle/busy/error）、轮询分配、健康检查/重登录 | HTTP 路由、DeepSeek 协议 |
| `admin.py` | 管理后台 API 端点（密码认证、统计查询、账号池 CRUD、重登录触发） | 前端渲染、DeepSeek 协议 |
| `tool_dsml.py` | DSML XML 解析与生成、工具提示词构建 | 流式检测、HTTP |
| `tool_sieve.py` | 流式文本中实时检测 DSML 工具调用标签 | 协议解析、HTTP |

**核心原则：** `adapter.py` 的输出是"裸 token 流"（str 或 dict），`server.py` 和 `anthropic_format.py` 分别负责包装成特定 API 格式。`adapter.py` 不知道任何 API 格式的存在。

#### 辅助文件

| 文件 | 用途 |
|------|------|
| `sha3_wasm_bg.wasm` | PoW 哈希引擎 WASM 二进制，从 DeepSeek 前端 JavaScript 中提取。通过 `wasmtime` 加载，导出 `wasm_solve` 等函数。版本与 DeepSeek 前端发布绑定。 |
| `start.bat` | Windows 一键启动脚本。自动检测并释放 `PORT` 环境变量指定的端口（默认 8080），然后启动 uvicorn。如果端口被其他进程占用，会先强制终止该进程及其子进程。 |
| `webui/` | 管理面板前端（纯静态 SPA）。包含 `index.html`、`app.js`、`style.css`。零 npm/build 依赖，通过 FastAPI 静态文件挂载在 `/webui/` 路径。 |
| `requirements.txt` | Python 依赖声明。关键依赖版本要求见下表。 |
| `.env.example` | 环境配置模板。复制为 `.env` 后填入凭证。提交到仓库但不含敏感信息。 |

**依赖版本说明：**

| 包 | 最低版本 | 用途 | 备注 |
|---|---------|------|------|
| `fastapi` | 0.100.0 | Web 框架 | 0.100+ 支持 Pydantic v2 |
| `uvicorn` | 0.20.0 | ASGI 服务器 | 0.20+ 支持 lifespan |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snake-aabb-wtf/deepseek-web2api-free](https://github.com/snake-aabb-wtf/deepseek-web2api-free) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
