---
trigger: always_on
description: 本文件是给后续 Codex、Agent、自动化脚本和人类协作者看的工程说明。进入本仓库后，先读本文件，再动代码。
---

# AGENTS.md

本文件是给后续 Codex、Agent、自动化脚本和人类协作者看的工程说明。进入本仓库后，先读本文件，再动代码。

## 项目定位

Pragma 是一个多专家 Agent 编排系统的长期工程底座，目标是沉淀可扩展、边界清晰、协议可治理、运行时可替换的 Agent 编排平台。

项目演进优先级是：架构合理性、功能完整性、实现清晰度、可验证质量。允许为了合理架构引入 breaking change；不要为了兼容旧代码保留无用适配层、废弃字段、空实现或迁移期分支。发现已经弃用且没有长期价值的代码，应直接删除并同步更新调用方、类型和文档。

## 技术基线

```text
Node.js: >= 22
Package Manager: pnpm
Monorepo: pnpm workspace + Turborepo
Language: TypeScript
Module System: ESM
Lint: ESLint Flat Config
Formatting: Prettier
Test: Vitest
Web: Next.js
Server: Fastify
Worker: Node.js + TypeScript
```

所有新增 TypeScript 代码必须满足严格模式。所有 package 必须是 ESM。

## 目录结构

```text
apps/
  web/       Next.js Web 应用，页面与浏览器交互入口
  server/    Fastify HTTP API 应用
  worker/    Node Worker 应用
  desktop/   未来 Desktop App，本地 Agent 桥接入口

packages/
  shared/         跨进程、跨端协议、领域模型和值对象、跨端纯函数工具
  client/         浏览器或客户端使用的 HTTP SDK
  server/         Node 服务端基础设施边界，例如数据库边界
  core/           ExpertAgent、Context、工具、插件、Runtime Adapter 与默认 Runtime
  memory/         Host 内置 Memory Plane、Evidence adapter、Module registry 与联邦 Context
  mission-board/  Host 可复用的 Mission 白板 Context bindings 与使用 Guide
  context-filesystem/ Host 侧文件系统 Context adapter 出口
  interpreter/    Pragma YAML DSL 的 AST、解析、校验、编译、扩展 registry 与 dump
  eslint-config/ 共享 ESLint 配置出口
  tsconfig/      共享 TypeScript 配置

docs/
  architecture/ 架构说明
  adr/          架构决策记录
  conventions/ 编码约定

infra/
  compose/      基础设施编排目录
```

未来 Desktop 本地 Agent 桥接目录规划：

```text
apps/
  desktop/             Desktop App，负责本地登录、设备绑定、权限确认、连接云端、本地 Agent 调用

packages/
  core/src/local-agent-bridge/     云端与 Desktop App 的桥接协议、消息类型、能力注册模型
  server/src/runtime-gateway/      云端 Runner/Device 注册、会话管理、Run 下发、事件接收
```

不要新增 `apps/local-runner`。本地 Agent 的产品入口是 Desktop App，而不是独立 CLI runner。

关键根文件：

```text
package.json
pnpm-workspace.yaml
turbo.json
eslint.config.mjs
prettier.config.mjs
tsconfig.base.json
```

## Package 命名

统一使用 `@pragma/*` scope。

当前 package：

```text
@pragma/shared
@pragma/client
@pragma/server
@pragma/core
@pragma/memory
@pragma/interpreter
@pragma/evaluation
@pragma/built-in-agents
@pragma/runtime-pi
@pragma/runtime-codex
@pragma/runtime-claude-code
@pragma/runtime-qodercli
@pragma/runtime-antigravity
@pragma/desktop
@pragma/examples
@pragma/eslint-config
@pragma/tsconfig
```

不要新增模糊名称，例如：

```text
common
base
shared-lib
helpers
lib
```

新增 package 前，必须先明确它属于 `shared`、`client`、`server`、`core`、`interpreter`、`built-in-agents`、`runtime-*`、`plugins/*`、`examples`、`apps/*` 还是配置工具。

## 模块依赖规范

下面的箭头统一表示“左侧可以依赖右侧”。

核心原则：

- `shared` 是最底层协议、领域模型和纯工具，不依赖任何运行环境层。
- `client` 是浏览器/客户端 SDK，只依赖 `shared`，不直接碰 Server 内部实现或 Agent。
- `core` 是专家 Agent 的执行抽象和 Runtime Adapter 边界，只依赖 `shared` 和 core 内部模块，不依赖具体 runtime、`client` 或 `server`。
- `evaluation` 是独立测评领域包，拥有 Evaluation 协议、Run Dry 执行器与结果模型；只依赖 `core`，不依赖 `interpreter` 或应用层。
- `interpreter` 是 Pragma DSL 的语言实现，拥有 AST、解析、链接、校验、编译、扩展 registry 和 dump；可以依赖 `evaluation` 和 `core`，但 `core` 与 `evaluation` 不得反向依赖 `interpreter`。
- `built-in-agents` 是六个内置 Agent（Pragma、Memory Curator、Store Revision、Skill Revision、Skill Evaluation、Evaluation Judge）的跨 Host 产品能力包。所有 Agent 均由静态 DSL 定义；包内拥有 descriptor/compiler、独立宿主端口、提示词、结构化输出解析、修订规则与纯状态机。Host 负责 Runtime 执行、权限、持久化、Mission 和 UI 适配，不要求六个 Agent 使用统一调用接口。
- `memory` 是 Host 内置 Memory Plane，拥有 Evidence adapter、Module SPI、独立消费状态和联邦只读 Context；只依赖 `core` 与 `shared`，不得反向进入 Core。
- `mission-board` 是 Host 可复用的 Mission-scoped 通用白板，只依赖 `core` Context 合约，不依赖文件系统、Memory 或应用。
- `context-filesystem` 是显式 Node/Host 文件系统 adapter 出口；Mission Board 与 Memory 不得依赖它。
- `runtime-*` 是具体 Runtime Adapter 实现，依赖 `core`、`shared` 和该 runtime 自己的 SDK；不同 runtime 包相互独立。
- `server` 是服务端控制面与基础设施层，可以依赖 `shared` 和 `core` 抽象。
- `apps/server` 和 `apps/worker` 是云端运行入口，未来由它们调度专家 Agent；不是 Agent 反过来依赖 Server。
- `apps/desktop` 是未来本地 Agent 桥接入口，主动连接云端，承载本地权限闸门和本机 Agent 调用。

```text
apps/web    -> client -> shared
apps/server -> server -> core -> shared
apps/worker -> server -> runtime-* -> core -> shared
apps/desktop    -> built-in-agents -> interpreter -> core -> shared
apps/desktop    -> interpreter -> evaluation -> core -> shared
apps/desktop    -> runtime-* -> core -> shared
apps/desktop    -> memory -> core -> shared
plugins/*   -> core -> shared
examples    -> runtime-* / plugin-* / core -> shared
```

更具体地说：

| 来源                       | 允许依赖                                                                                                                                             |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `apps/web`                 | `@pragma/shared`、`@pragma/client`                                                                                                                   |
| `apps/server`              | `@pragma/shared`、`@pragma/server`、`@pragma/core`                                                                                                   |
| `apps/worker`              | `@pragma/shared`、`@pragma/server`、`@pragma/core`、具体 `@pragma/runtime-*`                                                                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pqpo/pragma](https://github.com/pqpo/pragma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
