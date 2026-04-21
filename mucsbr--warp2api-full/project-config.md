---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ 重要注意事项 - 2025-09-24 更新

### Git Diff 单文件约束
**重要**: 使用 `git diff` 或类似命令查看文件更改时，**必须每次只检查一个文件**，避免执行问题。

✅ **正确示例**:
```bash
git diff file1.py
git diff file2.py
```

❌ **错误示例**:
```bash
git diff file1.py file2.py  # 这会导致执行失败！
```

### 已知问题和解决方案
- **空 content 错误**: 已修复 - 工具调用展开时使用空字符串而非 None
- **工具调用序列不完整**: 已添加自动清理逻辑
- **长文本响应中断**: 已实现智能分段（每段 1000 字符）
- **上下文重置**: 会自动提供任务延续提示

详细故障排查请参考：[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

## 执行原则（强制）

- 读→改→再读验证（Read→Edit→Read）：进行任何文件修改前，必须先 Read 获取最新内容；完成 Edit 后，必须再次 Read 验证变更已生效且未误伤其它位置。
- 单点精确替换：使用唯一且完整的旧片段作为锚点进行替换，避免大范围模糊替换导致串改。
- 失败即复核：遇到 Edit 失败或不匹配，先 Read 核对真实文件内容，再调整旧片段后重试，直至验证通过。

### 文件更新操作提示词与规范（强制）

- 工具约束与常见原因
  - Edit 失败的常见原因：
    1) old_string 与文件内容不完全一致（换行/空格/缩进/标点差异）
    2) old_string 在文件中出现多处（非唯一），需要提供更大上下文或启用 replace_all
    3) 先前未 Read，导致基于过期上下文构造的 old_string
  - Edit 工具要求：
    - old_string 必须与文件中目标文本逐字符匹配（不含行号前缀）
    - 默认只替换首个匹配；若匹配不唯一且需全部替换，应设置 replace_all=true

- 推荐操作流程
  1) Read 目标文件，定位精确锚点（尽量包含上下文以确保唯一性）
  2) 构造 Edit：
     - old_string：粘贴文件中“完整且唯一”的原始文本片段（保持缩进与空白）
     - new_string：仅替换需要的内容，其余保持不动
     - 如需多处同样替换，使用 replace_all=true
  3) Read 再次验证变更：对目标行前后各读取若干行，确认已生效且未误伤

- 提示词模板
  - 单点替换（推荐）：
    「我将先 Read 文件，随后用 Edit 以‘唯一锚点片段’进行精确替换，完成后再 Read 验证。」
  - 批量替换：
    「该文本在文件中出现 N 处，需要全量替换，我将使用 replace_all=true，并在完成后 Read 检查所有位置。」
  - 失败处理：
    「Edit 失败/不匹配。我将重新 Read 获取最新内容，调整 old_string 的上下文后重试，直至验证通过。」

## 项目概述

Warp2Api 是一个基于 Python 的桥接服务，为 Warp AI 服务提供多种 API 兼容性。该项目采用双服务器架构，通过 protobuf 与 Warp AI 服务通信，同时为客户端提供多种兼容的 API 接口。

**核心特点**：
- 无需预填 token，程序自动获取匿名访问令牌（50次调用额度）
- **双 API 支持**：同时支持 OpenAI Chat Completions API 和 Anthropic Messages API
- 支持 system prompt 和 function calling（通过 MCP 兼容）
- 采用"偷梁换柱"方式处理历史消息和工具调用
- **完整格式转换**：支持 OpenAI ↔ Anthropic 格式双向转换
- 支持多模态数据包（可进一步开发）
- **流式响应兼容**：支持两种 API 格式的流式响应

## 常用命令

### 开发环境启动
```bash
# 安装依赖（推荐使用 uv）
uv sync

# 启动 Protobuf 桥接服务器 (端口 8000) - 必须先启动
python server.py
# 或使用项目脚本
warp-server

# 启动 API 服务器 (端口 8010) - 支持 OpenAI 和 Anthropic 格式 - 需要等待"热身"完成
python openai_compat.py
# 注意：pyproject.toml 中的 warp-test 脚本配置有误，应该指向 openai_compat:main
```

### 健康检查
```bash
# 检查 Protobuf 桥接服务器
curl http://localhost:8000/healthz

# 检查 API 服务器
curl http://localhost:8010/healthz
```

### 测试 API
```bash
# 测试 OpenAI Chat Completions 端点
curl -X POST http://localhost:8010/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-sonnet",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'

# 测试 Anthropic Messages 端点
curl -X POST http://localhost:8010/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'
```

### 代码质量检查
```bash
# 项目使用 uv 管理依赖，暂无配置的 lint 或 test 命令
# 可以手动运行基本的 Python 语法检查
python -m py_compile server.py openai_compat.py

# 检查项目依赖
uv tree
```

## 架构概览

### 双服务器架构
项目采用分层架构设计，包含两个主要服务器：

1. **Protobuf 桥接服务器** (`server.py`, 端口 8000)
   - 处理与 Warp AI 服务的 protobuf 通信
   - 提供 JSON ↔ Protobuf 编解码服务
   - 包含 WebSocket 监控功能
   - 管理 JWT 认证和令牌刷新
   - 解析 `server_message_data` 字段（Base64URL → 结构化对象）

2. **多格式 API 服务器** (`openai_compat.py`, 端口 8010)
   - 提供 OpenAI Chat Completions API 兼容接口
   - **新增 Anthropic Messages API 兼容接口**
   - 处理客户端请求并转换为 protobuf 格式
   - 支持两种格式的流式响应 (SSE)
   - 实现消息重排序和格式转换
   - **双向格式转换**：OpenAI ↔ Anthropic 自动转换
   - 启动时需要"热身"请求

### 核心模块结构

#### `protobuf2openai/` - 多格式 API 兼容层
- `app.py` - FastAPI 应用主入口，包含启动健康检查
- `router.py` - API 路由定义，处理 `/v1/chat/completions` 和 `/v1/messages` 端点
- `models.py` - Pydantic 数据模型，定义 OpenAI 和 Anthropic API 兼容的数据结构
- **`anthropic_converter.py` - Anthropic ↔ OpenAI 格式转换器**
- **`anthropic_sse_transform.py` - Anthropic 格式流式响应处理器**
- `bridge.py` - 桥接逻辑，连接多格式 API 和 Protobuf 服务
- `sse_transform.py` - OpenAI 格式服务器发送事件 (SSE) 流式响应处理
- `reorder.py` - 消息重排序逻辑，适配 Anthropic 风格对话
- `config.py` - 配置管理，读取环境变量和设置
- `packets.py` - 核心转换逻辑，处理消息格式转换
- `helpers.py` - 辅助函数，处理内容格式化

#### `warp2protobuf/` - Warp Protobuf 通信层
- `api/` - API 路由和处理逻辑
- `core/` - 核心功能模块
  - `auth.py` - JWT 认证管理，包括匿名令牌获取、刷新和验证
  - `protobuf_utils.py` - Protobuf 编解码工具函数
  - `logging.py` - 日志配置和管理
  - `schema_sanitizer.py` - 输入模式清理和验证
  - `session.py` - 会话管理
  - `protobuf.py` - Protobuf 运行时管理
  - `server_message_data.py` - server_message_data 字段解析
- `config/` - 配置文件和设置
- `warp/` - Warp 特定的通信逻辑

### 数据流架构

```
客户端应用 → 多格式 API 服务器 → Protobuf 桥接服务器 → Warp AI 服务
(8010端口)      (FastAPI)          (Protobuf处理)      (Warp服务)
                   ↓                     ↓
    OpenAI/Anthropic 格式 ←→ Protobuf JSON格式 ←→ Protobuf二进制
                   ↕
            格式自动转换层
```

## 关键技术细节

### 认证机制
- **匿名访问**: 程序自动获取匿名账号令牌（50次调用额度）
- **JWT 管理**: 自动令牌验证和刷新
- **令牌获取**: 通过匿名接口获取鉴权 token 和 refresh_token
- **配置**: 可选环境变量 `WARP_JWT` 和 `WARP_REFRESH_TOKEN`
- **实现**: 认证逻辑在 `warp2protobuf/core/auth.py` 中实现

### 消息处理流程
1. **客户端请求**: 发送 OpenAI 格式请求到 API 服务器
2. **消息重排序**: 验证请求格式并进行消息重排序（适配 Anthropic 风格）
3. **格式转换**: 请求转换为 protobuf JSON 格式
4. **Protobuf 编码**: 转换为 protobuf 二进制发送到桥接服务器
5. **Warp 通信**: 桥接服务器与 Warp AI 服务通信
6. **响应返回**: 通过相同路径返回给客户端


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mucsbr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
