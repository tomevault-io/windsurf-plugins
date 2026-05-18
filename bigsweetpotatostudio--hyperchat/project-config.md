---
trigger: always_on
description: * import .mts 文件时，使用 import { xxx } from './xxx.mjs' 的方式导入。
---

# 请回复中文

## typescript使用指南
* import .mts 文件时，使用 import { xxx } from './xxx.mjs' 的方式导入。
* xx.ts.bak ， .bak 文件是老逻辑代码，不用阅读，也不用修改和删除。

## 项目概述

HyperChat 是一个多平台的 AI 聊天应用，该项目拥有完善的 MCP（模型上下文协议） 支持，并集成了包括 OpenAI、Claude、Gemini、Qwen、Deepseek 等在内的多种大语言模型 API。

### 多平台支持
* **核心**: nodejs
* **Web前端**: 通过浏览器访问，支持h5
* **Electron**: 桌面应用，自带浏览器
* **命令行前端**: 类似Claude Code，已集成到core包中，配置通过web前端完成
* **VSCode插件**: 通过webview访问构建

### 包结构
* `packages/shared` - @dadigua/hyperchat-shared，共享代码和类型定义+zodSchemas，前后端通用
* `packages/core` - Node.js 后端服务 + CLI命令行工具
* `packages/web` - Web 前端的实现
* `packages/electron` - Electron 桌面应用

## 开发逻辑

### 类型安全
* 尽量使用 TypeScript 的类型系统来确保代码的类型安全。尽量少使用any类型。
* packages/shared/src/types.mts 定义了常用的类型，包括前端和后端交互的类型，确保前后端的数据结构一致。
* packages/shared/src/zodSchemas文件夹 定义了 Zod schema，用于数据验证和前端表单生成。所有的 schema 都是基于 TypeScript 类型定义的，确保类型一致性。
* packages/core/src/data/managers 文件夹 包含了各种数据管理器类对应packages/shared/src/zodSchemas，这些类使用typescript类型 和 zodSchemas来确保数据的类型安全。
* 使用 Zod schema 进行数据验证，通过 zod-to-json-schema 转换为 JSON Schema 用于前端表单生成。
* 不允许 await import()。这样逻辑更加清晰，避免了动态导入带来的复杂性。

### 环境变量系统
* 实现了5层优先级的环境变量系统（默认值 < process.env < 全局.env < 工作区.env < CLI参数）

### web前后端通信
* 前端发送消息给后端，默认通过 packages/core/src/command.mts 实现，前端通过调用 call 的方法来实现与后端的交互。
* electron提供更多electron接口 packages/electron/src/command.mts， 前端通过调用 callElectron 的方法来实现与electron的交互。
* 后端发送消息给前端是通过websocket实现的 packages/core/src/message_service.mts，前端通过监听 websocket 的消息来接收后端发送的消息。

### 组件设计原则
* 优先使用现有的 Ant Design 组件库
* 遵循 React Hooks 最佳实践，使用 useCallback、useMemo 等优化性能
* 表单使用 Ant Design Form 组件，支持 Form.List 处理动态数组
* 错误处理和用户体验优先，提供清晰的错误提示和加载状态

## i18n 国际化
* i18n 相关的代码在 packages/shared/src/i18n 中。 软件默认使用英文，然后通过转成 JSON 的方式来支持国际化。t`english`, 里面不应该有${xxx}。
* packages/shared/src/i18n/i18n.json 不用修改。后续我会提供一个脚本来自动生成 i18n.json 文件。

## ✅ 新架构决策 (2.0版本 - 已完成实现)

### 核心理念：进程/会话 = 全局配置 + 工作区配置（覆盖模式）
**目标**：统一CLI和Web端的架构，实现更简单、一致的用户体验

### 配置层次结构
```
一个CLI进程/Web会话 = 全局配置基础 + 当前工作区配置覆盖

启动流程：
1️⃣ 加载全局配置 (~/.hyperchat/)
   ├── 全局AI模型配置
   ├── 全局MCP工具配置  
   ├── 全局Agent配置
   └── 全局系统设置

2️⃣ 检测/选择当前工作区 (./.hyperchat/ 或项目根目录)
   ├── 工作区AI模型配置 (覆盖全局)
   ├── 工作区MCP工具配置 (补充/覆盖全局)
   ├── 工作区Agent配置 (补充全局)
   └── 工作区项目设置

3️⃣ 合并配置启动服务
   ├── 最终AI模型列表
   ├── 最终MCP客户端列表
   ├── 最终Agent列表
   └── 统一运行环境
```







## ✅ 已完成的架构重构

### Workspace配置合并架构重构 (完成) 🆕
**核心文件**: `packages/core/src/workspace/workspace.mts`


#### 三种工作模式
1. **非工作区目录**: 自动回退到全局配置
2. **工作区目录**: 全局配置 + 工作区配置合并
3. **全局工作区**: 直接使用全局配置




## 当前构建命令 🚀

### 可用的构建脚本 (更新)
```bash
# 构建
npm run build             # 构建所有包（按依赖顺序）
npm run build:shared      # 构建 shared 包
npm run build:web         # 构建 Web 前端
npm run build:core        # 构建 Core 后端 + CLI
npm run build:electron    # 构建 Electron 应用

# 开发模式
npm run dev:shared        # shared 包开发模式（watch）
npm run dev:web          # Web 开发服务器
npm run dev:core         # Core + CLI 开发模式
npm run dev:electron     # Electron 开发模式

# 工具
npm run clean            # 清理所有构建产物
npm run typecheck        # 所有包类型检查
```

### 构建顺序 (更新)
1. **shared** - 必须最先构建，其他包依赖它
2. **web** - React 前端构建
3. **core** - Node.js 后端 + CLI 构建
4. **electron** - 桌面应用（依赖 web 构建产物）

### CLI使用方式 (更新)
```bash
# 直接运行
node packages/core/dist/cli/index.mjs --help

# 安装后使用 (如果全局安装core包)
hyperchat --help
hc workspace current

# 常用命令
hyperchat chat                  # 直接AI对话
hyperchat "你好"                    # 直接AI对话
hyperchat serve                   # 启动Web服务器 (包含 Web 界面)
hyperchat run                     # 启动核心服务 (不包含 Web 界面，适合后台运行)
hyperchat agent list              # 列出AI代理
hyperchat agent [agent_name] "你好"          # 使用某个agent直接AI对话
hyperchat agent [agent_name] chat          # 使用某个agent进行对话
```

## 🗂️ 项目结构 (更新)

### 当前包结构
```
HyperChat/
├── packages/
│   ├── shared/          # 共享类型和工具库
│   ├── web/            # React Web前端
│   ├── core/           # Node.js后端 + CLI (合并后)
│   │   ├── src/cli/    # ✨ CLI命令行工具
│   │   │   ├── index.mts          # CLI主入口
│   │   │   ├── commands/          # 命令实现
│   │   │   │   ├── agent.mts     # 代理管理
│   │   │   │   ├── chat.mts      # AI聊天
│   │   │   │   ├── config.mts    # 配置管理
│   │   │   │   ├── server.mts    # 服务器控制
│   │   │   │   └── workspace.mts # 工作区管理
│   │   │   └── utils/            # CLI工具函数
│   │   ├── src/workspace/        # 工作区管理
│   │   ├── src/command.mts       # API命令层
│   │   └── src/mcp/              # MCP协议实现
│   └── electron/       # Electron桌面应用
```

### 全局配置目录
```
~/Documents/HyperChat/
    .hyperchat/
    ├── mcp.json                  // 全局主控程序 (MCP) 配置文件
    ├── agents/
    │   ├── agent1-key/
    │   │   ├── memory.md         # Agent记忆
    │   │   ├── sub_agents/       # 子代理文件夹（类似 agents 文件夹）
    │   │   ├── agent.yaml        # Agent配置
    │   │   └── chatlogs/         # 聊天记录文件夹
    │   │       ├── chat1.yaml
    │   │       ├── chat2.yaml
    │   │       └── ...
    │   └── ...
    ├── tasks/                    // 任务文件夹
    │   ├── task1.yaml            # 单个任务的定义文件
    │   ├── task2.yaml
    │   └── ...
    └── ...
```

### 项目工作区结构
```
/projects/
    project1/
      .hyperchat/
      ├── mcp.json                  // 全局主控程序 (MCP) 配置文件
      ├── ai_models.json            // AI 模型配置文件，包含所有可用的 AI 模型信息
      ├── agents/
      │   ├── agent1-name/
      │   │   ├── memory.md         # Agent记忆
      │   │   ├── sub_agents/       # 子代理文件夹（类似 agents 文件夹）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BigSweetPotatoStudio/HyperChat](https://github.com/BigSweetPotatoStudio/HyperChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
