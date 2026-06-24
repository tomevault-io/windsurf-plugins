---
trigger: always_on
description: > **所有更改必须通过 `npm run compile` / `npx tsc --noEmit` 编译检查无错误通过。**
---

# OpenCode Go Copilot Provider — AGENTS.md

> **所有更改必须通过 `npm run compile` / `npx tsc --noEmit` 编译检查无错误通过。**  
> **每次更改后，必须同步更新本文档 (`AGENTS.md`) 以反映代码变更。**

---

## 目录

1. [项目详细介绍](#1-项目详细介绍)
2. [详细逻辑架构](#2-详细逻辑架构)
3. [程序文件索引](#3-程序文件索引)
4. [函数定义大全](#4-函数定义大全)
5. [编译与构建](#5-编译与构建)
6. [开发规范](#6-开发规范)

---

## 1. 项目详细介绍

### 1.1 概述

**OpenCode Go Copilot Provider** 是一个 VS Code 扩展，它将 OpenCode Go 平台的 AI 语言模型集成到 GitHub Copilot Chat 中。用户可以在 VS Code 的 Copilot Chat 界面中选择并使用 OpenCode Go 提供的各种模型（如 DeepSeek、GLM、Qwen、MiMo、MiniMax、Kimi 等系列），享受智能代码补全、聊天对话、Git 提交消息生成等功能。

### 1.2 核心能力

| 能力 | 说明 |
|------|------|
| **Chat 模型提供商** | 实现 `LanguageModelChatProvider` 接口，向 VS Code 注册为 `opencodego` 厂商 |
| **多模型支持** | 内置 17 个模型定义，覆盖 6 大模型系列，统一通过推理强度选择器切换思考模式。可选开启 OpenCode Zen 免费模型（6 个） |
| **OpenCode Zen 免费模型** | 通过设置开关启用，从 Zen API 获取模型列表并过滤出 6 个免费模型（Big Pickle、DeepSeek V4 Flash、MiniMax M3、MiniMax M2.5、Ring 2.6 1T、Nemotron 3 Super），以 `OpenCode Zen` 标识追加到模型选择器。支持内存缓存（5 分钟 TTL），API 不可用时静默降级 |
| **双 API 模式** | 同时支持 **OpenAI 兼容格式** (`/chat/completions`) 和 **Anthropic 格式** (`/v1/messages`) |
| **流式推理** | 支持 SSE (Server-Sent Events) 流式响应，实时输出文本和工具调用 |
| **Thinking/推理** | 支持模型的推理过程展示 ("thinking" 状态)，包括 XML think 块解析 |
| **工具调用 (Tool Calling)** | 支持 VS Code 的 LanguageModelToolCallPart 机制 |
| **图片代理 (Tool-based)** | 为不支持视觉的模型注入 `ask_image` 工具，模型可自主选择调用视觉模型（默认 Qwen3.6-Plus）回答关于图片的具体问题，支持两轮 API 请求完成"调用工具→提问→获取答案→继续回答"的完整流程。与旧版 `describe_image` 不同，`ask_image` 允许模型针对图片提出具体问题（如"按钮是什么颜色？"），视觉模型会针对性回答。视觉模型 ID、查询提示词和思考模式均可通过设置配置；视觉代理会在同一个 thinking 块中显示“正在根据图片提问：[问题]”并实时追加视觉模型流式输出 |
| **Token 计数** | 使用 `o200k_base` tiktoken 分词器精确统计 token 用量 |
| **状态栏** | 实时显示当前会话 token 使用量、累计用量、缓存命中率 |
| **原生 Token 指示器** | 始终启用，向 Copilot Chat 原生 Token 指示器报告 token 用量。通过发送 MIME 类型为 `usage` 的 `LanguageModelDataPart`（TextEncoder 编码 JSON）实现，无需自建状态栏。依赖 VS Code/Copilot Chat 1.116+ 对外部模型 `usage` data part 的识别 |
| **高级 Token 指示器** | 可通过 `opencodego.enableThirdPartyTokenIndicator` 配置（默认开启）控制 VS Code 状态栏中的高级Token计数器。关闭后仅显示原生指示器 |
| **Git 提交消息生成** | 一键生成 Conventional Commit 格式的 Git 提交消息，支持 `auto` 语言模式自动从历史提交检测语言 |
| **多仓库支持** | 支持多根工作区 (multi-root) 中多个 Git 仓库的提交消息生成 |
| **模型预设** | 支持通过命令面板快速切换 temperature/top_p 预设（🎯 Precise/⚖️ Balanced/🔥 Creative），也支持手动自定义输入 |
| **国际化** | 内置简体中文 (zh-cn) 中英文双语界面 |
| **重试机制** | 可配置的指数退避重试策略，应对网络抖动和限流 (429) |
| **请求延迟** | 可配置的请求间隔延迟，避免触发 API 限流 |
| **超时控制** | 可配置的请求超时时间（默认 10 分钟） |
| **立即取消** | 取消请求时通过 `reader.cancel()` 立即中断流式读取，停止后台接收 |
| **视觉代理配置** | 支持通过设置 `opencodego.visionProxyModel`、`opencodego.visionProxyThinking` 配置图片代理所使用的视觉模型和思考模式。`opencodego.visionProxyThinking` 默认关闭，关闭时内部请求通过 `modelOptions.thinking={ type: false }` / `reasoning_effort="disabled"` 禁用视觉模型思考，最终 OpenAI 兼容请求体发送 `thinking: { type: false }` |
| **安装欢迎页 (Walkthrough)** | 首次安装且未配置 API Key 时自动打开引导向导，指引用户设置 API Key 和打开语言模型管理器。包含 3 个步骤：设置 API Key、显示模型、高级设置。通过 `onStartupFinished` 激活事件确保在 VS Code 启动后立即检测 |

### 1.3 模型清单

#### 内置模型

| 系列 | 模型 ID | 视觉 | 推理强度选择器 | API 格式 |
|------|---------|------|----------------|----------|
| GLM | `glm-5.1`, `glm-5` | ❌ | `思考`（不支持思考切换） | OpenAI |
| Kimi | `kimi-k2.5`, `kimi-k2.6`, `kimi-k2.7-code`¹ | ✅ | `思考`（不支持思考切换） | OpenAI |

> ¹ `kimi-k2.7-code` 不支持设置 Temperature 参数。
| DeepSeek | `deepseek-v4-pro`, `deepseek-v4-flash` | ❌ | `禁用思考` / `高` / `极高` | OpenAI |
| MiMo | `mimo-v2-pro`, `mimo-v2-omni`, `mimo-v2.5-pro`, `mimo-v2.5` | mimo-v2-omni ✅ | `禁用思考` / `思考` | OpenAI |
| MiniMax | `minimax-m3`, `minimax-m2.7`, `minimax-m2.5` | ❌ | `禁用思考` / `自动` | OpenAI (m2.7 使用 Anthropic) |
| Qwen | `qwen3.7-max` | ❌ | `禁用思考` / `自动` | Anthropic |
| Qwen | `qwen3.6-plus`, `qwen3.5-plus` | ✅ | `禁用思考` / `自动` | Anthropic |

#### OpenCode Zen 免费模型（可选）

通过设置 `opencodego.enableZenFreeModels`（默认关闭）启用，从 Zen API 获取模型列表并与硬编码 ID 过滤后加入模型选择器。

| 显示名 | 模型 ID | 视觉 | 推理强度选择器 | API 格式 | 备注 |
|--------|---------|------|----------------|----------|------|
| Zen/Big Pickle Free | `big-pickle` | ❌ | `思考`（不支持思考切换） | OpenAI | 限时免费 |
| Zen/DeepSeek V4 Flash Free | `deepseek-v4-flash-free` | ❌ | `禁用思考` / `高` / `极高` | OpenAI | 限时免费 |
| Zen/MiniMax M3 Free | `minimax-m3-free` | ✅ | `禁用思考` / `自适应` | OpenAI | 限时免费；1M 上下文，仅支持 `adaptive` / `disabled` 思考模式 |
| Zen/MiniMax M2.5 Free | `minimax-m2.5-free` | ❌ | `禁用思考` / `思考` | OpenAI | 限时免费 |
| Zen/Ring 2.6 1T Free | `ring-2.6-1t-free` | ❌ | `禁用思考` / `思考` | OpenAI | 限时免费 |
| Zen/Nemotron 3 Super Free | `nemotron-3-super-free` | ❌ | `禁用思考` / `思考` | OpenAI | 限时免费 |

在模型选择器中，内置模型归入 `OpenCode Go` 分组（`family="OpenCodeGo"`），Zen 免费模型归入 `OpenCode Zen` 分组（`family="OpenCode Zen"`）以作区分。

> 所有模型在模型选择器中均显示**一个条目**，通过**推理强度选择器**（中文标签）切换思考模式。  
> 所有模型在模型选择器中均显示**一个条目**，通过**推理强度选择器**（中文标签）切换思考模式。  
> - `thinkingMode="switchable"`：用户可选择`禁用思考`、`自动`或启用思考（强度可配置）  
> - `thinkingMode="adaptive"`：仅`禁用思考`和`自动`两档选择，无强制启用思考选项  
> - `thinkingMode="always"`：推理始终启用，选择器中不显示`禁用思考`选项（模型特性）  
> - `thinkingMode="always"`：推理始终启用，选择器中不显示`禁用思考`选项（模型特性）  
> 
> **关于图像输入：** 所有模型（包括非视觉模型）的 `imageInput` 能力均声明为 `true`，以确保 VS Code 始终传递图片数据。非视觉模型通过内部的 `ask_image` 工具代理机制处理图片，不直接支持视觉输入。

---

## 2. 详细逻辑架构

### 2.1 总体数据流

```
┌─────────────────────────────────────────────────────────────────────┐
│                        VS Code Copilot Chat                         │
│  ┌───────────────────────────────────────────────────────────────┐  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OnesoftQwQ/opencode-go-copilot](https://github.com/OnesoftQwQ/opencode-go-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
