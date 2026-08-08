---
trigger: always_on
description: 本文件为 OpenCode / AI 编程 Agent 在 ManyFoot 代码库中工作时提供高信号指导。
---

# AGENTS.md

本文件为 OpenCode / AI 编程 Agent 在 ManyFoot 代码库中工作时提供高信号指导。

AI 在本项目中不应只完成"能跑的代码"，而应以架构师视角进行设计与实现：优先保证模块边界清晰、职责单一、低耦合、高内聚、易测试、易扩展、易替换。

---

## 一、核心编码原则

### 1. 架构师式编码要求

在新增、修改代码时，必须优先考虑：

- 是否符合现有模块边界
- 是否破坏已有继承链、策略链、工厂注册机制
- 是否引入了不必要的耦合
- 是否方便后续扩展新的 Agent、模型厂商、工具、沙箱能力
- 是否可以通过配置扩展，而不是硬编码
- 是否可以独立测试
- 是否符合 Spring Boot 的 Bean 生命周期和条件装配习惯

不要为了快速实现功能而写临时逻辑、硬编码分支、跨模块直接调用、重复代码或"大而全"的类。

---

## 二、快速参考

### 后端

```bash
# 构建
mvn clean package

# 运行（端口 8100）
mvn spring-boot:run

# 测试
mvn test
```

### 前端

```bash
# 进入前端目录
cd foot-ui

# 安装依赖
npm install

# 启动开发服务器（端口 3000）
npm run dev

# 类型检查
npm run typecheck

# 生产构建
npm run build

# 预览构建
npm run preview
```

---

## 三、项目概述

ManyFoot 是基于 **Spring AI Alibaba Agent Framework** 的 Supervisor 多智能体协作系统，采用 ReactAgent 模式实现智能体编排。

### 技术栈

**后端：**

- Spring Boot 3.4.4
- Java 17
- Maven
- Spring AI 1.1.0
- Spring AI Alibaba Agent Framework 1.1.0.0
- Redis (Redisson)
- Docker Sandbox
- Apache Tika（文档解析）

**前端：**

- React 19
- TypeScript
- Vite
- Tailwind CSS CDN
- react-markdown + remark-gfm
- lucide-react

本项目定位：**可扩展的多智能体协作框架**，通过 Supervisor 模式协调 8 个智能体完成复杂任务。

---

## 四、根包与模块边界

### 后端根包

```text
com.lh.manyfoot
```

单模块 Maven 项目。

| 包路径 | 职责 |
|---|---|
| `agent.core.*` | 智能体核心接口：Agent、AbstractAgent、AbstractToolAgent、StreamingAgent、ToolAwareAgent |
| `agent.context.*` | 智能体执行上下文：AgentContext、AgentAttachment、SessionContextHolder |
| `agent.domain.*` | 智能体领域对象：PlanGraph、TaskSpec、ActionCall、ActionResult、DomainDraft 等 |
| `agent.exception.*` | 智能体异常：AgentExecutionException |
| `agent.factory.*` | 智能体工厂：AgentFactory、AgentType |
| `agent.impl.*` | 智能体实现：SupervisorAgent、PlannerRouterAgent、ResearchRetrievalAgent、DomainSpecialistAgent、DocumentSpecialistAgent、ToolActionExecutorAgent、CodeAgent、ChatAgent |
| `agent.prompt.*` | 提示词提供者：AgentPromptProvider 及各智能体专属 PromptProvider |
| `agent.registry.*` | 智能体注册表：AgentRegistry（动态发现所有 Agent Bean） |
| `agent.strategy.*` | 执行策略：ExecutionStrategy、SyncCallStrategy、StreamingStrategy |
| `agent.supervisor.*` | Supervisor 编排：AgentTool（Agent→ToolCallback 适配器）、SupervisorToolProvider |
| `agent.support.*` | 工具类：SpecialistJsonUtils、AgentMessageFactory |
| `agent.tool.*` | 工具提供者：AgentToolProvider、FullToolProvider |
| `agent.tool.sandbox.*` | 沙箱工具：SandboxTool、SandboxEngine、DocumentParser、ExecutionResultFormatter |
| `agent.tool.sandbox.domain.*` | 沙箱领域对象：SandboxExecutionRequest、SandboxExecutionResult、SandboxCodeType |
| `models.registry.*` | 模型注册：ModelResolver、AiModelRegistrar、ModelRole、AiModelInitializer |
| `models.failover.*` | 故障转移：FailoverChatModel |
| `models.support.*` | 模型支持：ChatOptionsBinder |
| `models.*` | 厂商工厂：AiModelFactory 接口及各厂商实现（Dashscope、OpenAI、Anthropic、DeepSeek、Gemini、Ollama、Qianfan 等） |
| `config.*` | Spring 配置：RedissonConfig、DockerClientConfig、McpClientConfig、ManyFootToolConfig |
| `config.properties.*` | 配置属性：AiProvidersProperties、SandboxConfig、VendorEnums |
| `controller.*` | REST API：ChatController、R（统一响应）、HttpStatus |
| `controller.dto.*` | 请求 DTO：ChatRequest |
| `service.*` | 基础设施服务：SandboxContainerManager、RedisUtils |
| `domain` | 共享 DTO / 枚举：ExecutionResult、ExecutionStatus、SandboxContainer、ContainerStatus、AiModelConfig |

### 前端目录结构

```text
foot-ui/
├── index.html              # HTML 入口（含 Tailwind CDN 主题配置）
├── package.json            # npm 脚本与依赖
├── package-lock.json       # npm 锁文件
├── vite.config.ts          # Vite 配置（开发代理、路径别名）
├── tsconfig.json           # TypeScript 配置
├── .gitignore              # 前端独立忽略规则
└── src/
    ├── main.tsx            # React 挂载入口
    ├── App.tsx             # 会话状态、流式响应聚合与主布局
    ├── types.ts            # 前端 API 与 UI 类型定义
    ├── components/         # 展示组件
    │   ├── ChatInput.tsx   # 输入 + 文件附件
    │   ├── ChatMessage.tsx # 消息渲染（Markdown）
    │   ├── Sidebar.tsx     # 会话列表
    │   └── SettingsModal.tsx # API 基础地址配置
    └── services/
        └── manyFootApi.ts  # ManyFoot 后端 API 客户端
```

### 模块边界要求

AI 修改代码时必须遵守：

- `controller` 只负责 API 入参、出参、协议适配，不写业务编排逻辑
- `agent` 只负责智能体行为抽象、提示词、策略和工具调用组织
- `models` 只负责模型解析、模型注册、厂商适配、故障转移
- `agent.tool.sandbox` 只负责代码执行和沙箱能力，不承载 Agent 决策逻辑
- `config` 只做配置绑定和 Bean 装配，不写运行时业务逻辑
- `service` 可放基础设施服务，但不要变成万能业务层
- **前端 `components/` 只负责展示，不直接调用 fetch/axios；所有 API 调用必须通过 `services/` 层**
- **前端 `services/` 只负责 API 客户端封装，不处理 UI 状态**
- **前端 `App.tsx` 负责全局状态管理；业务状态不要下沉到展示组件**

跨模块调用必须通过接口、抽象类、工厂或服务完成，避免直接依赖具体实现类。

---

## 五、入口点

### 后端

- 应用入口：`src/main/java/com/lh/manyfoot/ManyFootApplication.java`
- 智能体工厂：`com.lh.manyfoot.agent.factory.AgentFactory`

通过 `AgentFactory.getAgent(AgentType)` 获取智能体实例。

### 前端

- 应用入口：`foot-ui/index.html`
- React 挂载：`foot-ui/src/main.tsx`
- 状态根节点：`foot-ui/src/App.tsx`

---

## 六、关键架构模式

### 1. Supervisor 多智能体协作架构

本项目采用 **Supervisor 模式**，由 8 个智能体协作完成任务：

```text
┌─────────────────────────────────────────────────────────────┐
│                     Supervisor 协作流                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ PlannerRouter │───▶│  DomainSpec  │───▶│ ToolExecutor │  │
│  │   规划路由     │    │   领域专家    │    │   工具执行    │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                   ▲                   │           │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wuwahe/Many-Foot](https://github.com/wuwahe/Many-Foot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
