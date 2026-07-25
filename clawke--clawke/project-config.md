---
trigger: always_on
description: **⚠️ AI 沟通原则：说话必须极度精简，只讲重点，拒绝废话和过度解释，不要浪费时间。**
---

# Clawke - Claude 开发指南

## 项目简介

**⚠️ AI 沟通原则：说话必须极度精简，只讲重点，拒绝废话和过度解释，不要浪费时间。**

Clawke 是一款基于 **Edge-Cloud（边缘-云协同）** 架构的下一代多智能体（MAS）原生工作空间与富客户端 UI 渲染引擎。

核心创新是 **CUP (Clawke UI Protocol)**：Clawke Server 下发标准化 JSON，Native Client 作为无状态"渲染容器"，动态组装原生 UI 积木（Widgets）。

## 产品定位

Clawke 是**全平台产品**，目标覆盖 macOS、Windows、iOS、Android 四端。MVP 阶段优先 macOS Desktop 验证技术闭环，但架构设计和技术选型必须考虑多端兼容性。**严禁做出"仅桌面端"或"仅个人使用"的假设**——性能优化、数据库设计、状态管理等决策都必须兼顾移动端（CPU/内存受限）场景。

## 富客户端设计哲学

Clawke 不是普通的即时通讯应用（IM），而是**专为 OpenClaw 及其他 AI Agent 打造的富客户端工作空间**。如果只是文本对话，用户用任何 IM 都可以。Clawke 的核心价值在于：

- **所有 Agent 能力都必须有原生交互**：如 exec 审批、工具调用确认等，必须用原生按钮/卡片呈现，严禁降级为文本命令（如 `/approve xxx`）
- **全链路改造优先于局部补丁**：宁愿修改 Gateway → Server → Client 三层代码实现完整体验，也不做"发文本命令"的妥协方案
- **交互体验是差异化竞争力**：每个 Agent 交互场景（审批、文件操作、数据库查询等）都应设计专门的 SDUI 组件，让用户感受到与桌面端（ClawX）同等甚至更优的原生体验

> **决策原则**：当遇到"用文本命令快速实现" vs "用原生 UI 组件完整实现"的选择时，永远选择后者。

> **禁止临时兼容方案**：当某层（如 Gateway）的代码没有生效时，**禁止在其他层（如 Server）做 hack 绕过**。正确做法是找到根因（如编译缓存、进程未重启）并解决它。临时方案会变成屎山，且掩盖真正需要修复的问题。重编 + 重启永远优先于 workaround。

## 技术栈


| 层级               | 技术               | 说明                                                                          |
| ------------------ | ------------------ | ----------------------------------------------------------------------------- |
| Native Client      | Flutter            | 全平台（macOS/Windows/iOS/Android），MVP 优先 macOS Desktop，后续同时编译多端 |
| Mock Clawke Server | Node.js 或 Python  | 轻量 WebSocket 脚本，代码量 < 200 行                                          |
| 通信协议           | WebSocket (ws://)  | 全双工，流式文本 + SDUI JSON 广播                                             |
| 状态管理           | 内存级（MVP 阶段） | 无 SQLite，断连即清空                                                         |

## MVP 目标

验证完整技术闭环：

1. **全双工 WebSocket 通信**（Client ↔ Mock Server）
2. **CUP 协议解析**（服务端下发 JSON → 客户端动态渲染 Widget）
3. **纯原生组件渲染**（Flutter 原生绘制，零 WebView）
4. **交互事件回传**（用户点击 → 事件上报 → Server 打印日志）

## 项目结构

```
clawke/
├── CLAUDE.md              # 本文件
├── docs/                  # 📖 开源说明文档（对外公开，随代码提交）
│   ├── MRD.md             # 市场需求文档
│   ├── PRD.md             # 产品需求文档
│   ├── competitive-analysis/  # 竞品分析文档
│   ├── mvp/
│   │   └── MVP.md         # MVP 核心定义
│   └── plans/             # 实施计划（AI 生成）
├── docs/private/          # 🔒 内部开发文档（.gitignore 排除，不对外公开）
├── client/                # Flutter 原生客户端
├── server/                # Clawke Server（Node.js，双模式：mock / openclaw）
├── gateways/
    └── openclaw/
        └── clawke/        # Clawke 渠道插件（连接 Clawke Server 的 Gateway）
            ├── index.ts
            ├── openclaw.plugin.json
            └── src/
                ├── channel.ts   # ChannelPlugin 定义 + outbound 适配器
                ├── config.ts    # 渠道配置（url / enabled / allowFrom）
                ├── gateway.ts   # WebSocket 连接 Clawke Server + 自动重连
                └── runtime.ts   # PluginRuntime 注入
```

### 文档分类规则


| 目录             | 性质         | Git 追踪            | 内容                                                   |
| ---------------- | ------------ | ------------------- | ------------------------------------------------------ |
| `docs/`          | **开源文档** | ✅ 提交到仓库       | 产品文档、架构说明、用户指南 — 面向开源社区和外部用户 |
| `docs/private/`  | **内部文档** | ❌`.gitignore` 排除 | 开发笔记、分析报告、调试记录 — 仅供团队内部参考       |

**决策原则**：涉及内部实现细节、调试过程、竞品分析等敏感内容放 `docs/private/`；产品架构、协议规范、使用说明等适合公开的放 `docs/`。

## 模块级规则索引

根 `CLAUDE.md` 只保留跨模块总纲和红线。修改具体模块前必须读取对应模块规则：

- 修改 `client/**`：先读取 `client/AGENTS.md`
- 修改 `server/**`：先读取 `server/AGENTS.md`
- 修改 `gateways/**`：先读取 `gateways/AGENTS.md`

## 整体架构思路

遵循 **SDUI（Server-Driven UI）** 设计思路：服务端掌控 UI 逻辑，客户端仅负责渲染。具体通过 **CUP（Clawke UI Protocol）** 协议实现——Server 下发标准化 JSON 描述 UI 组件树，Client 解析后动态组装原生 Widget。这意味着：

- **新增 UI 能力不需要发版客户端**：Server 下发新的 `widget_name` + `props`，Client 侧只需注册对应 Builder
- **客户端零业务逻辑**：所有决策（该显示什么、何时显示）由 Server 决定
- **未知组件优雅降级**：遇到不认识的 `widget_name`，显示 `UpgradePromptWidget` 而非崩溃

### 瘦客户端原则

项目目标是多端支持（Mac → iOS → Android），为降低迁移成本，Client 端必须保持"瘦"：

1. **客户端只做渲染 + 交互反馈**，不做业务判断。按 CUP `payload_type` / `widget_name` 分发渲染即可
2. **新功能通过 CUP 组件下发**，不在客户端硬编码业务逻辑（协议级流式特性如 thinking 块除外）
3. **DB 只存消息和会话元数据**，不存业务状态
4. **平台差异用 `LayoutBuilder` / `MediaQuery` 适配**，避免 `if (Platform.isMacOS)` 式的硬分支
5. **共享代码放 `lib/core/`、`lib/models/`、`lib/providers/`**，平台特定 UI 放 `lib/screens/` 或 `lib/widgets/`

### Gateway 反腐败层（Anti-Corruption Layer）

Gateway 层是**唯一允许理解上游 Agent 私有协议的地方**。上游 Agent（如 OpenClaw）的格式变化、API 升级、协议变更，**必须全部在 Gateway 层吸收**，不得传导到 CS Server 或 Client：

```
Flutter Client ←CUP→ CS Server ←CUP→ Gateway A (OpenClaw)
                                 ←CUP→ Gateway B (Future Agent)
                                 ←CUP→ Gateway C (Future Agent)
```

**核心规则**：

1. **Gateway → CS Server 的输出必须是标准化 CUP 消息**，不含任何 Agent 特定的格式/术语
2. **CS Server 和 Client 永远不知道**后面接的是 OpenClaw 还是其他 Agent
3. **OpenClaw 升级时只改 Gateway 层**，CS Server 和 Client 代码零修改
4. **CUP 协议的交互能力（如审批、确认、工具调用）必须设计为通用机制**，不绑定特定 Agent 的概念（如不叫 `exec_approval`，而叫 `action_confirmation`）
5. **严禁在 Gateway 层解析非结构化文本**（如试图 regex 匹配 AI 回复中的命令），必须使用上游 Agent 提供的结构化 API/事件/Hook

> **决策原则**：新增功能时先问"这是 Agent 特定的还是通用的？"。Agent 特定逻辑放 Gateway，通用交互放 CUP 协议。

## 架构概览

```
Flutter Client ←ws:8765→ Clawke Server ←ws:8766→ OpenClaw Gateway（192.168.0.7）
```

- **Clawke Server**（`server/`）：双端口协议翻译网关，`MODE=mock` 用 Mock 数据，`MODE=openclaw` 接真实 AI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clawke/clawke](https://github.com/clawke/clawke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
