---
trigger: always_on
description: OryxOS 是用 Java 实现的面向企业场景的 **Distributed AI Agent OS**。装在企业自己的 K8s 或服务器上，作为统一底座运行多个业务 Agent，共享渠道接入、模型路由、工具调用、记忆系统、沙箱执行能力。数据完全留在企业自己的基础设施，不锁任何云生态。
---

# OryxOS — Claude Code 项目指南

OryxOS 是用 Java 实现的面向企业场景的 **Distributed AI Agent OS**。装在企业自己的 K8s 或服务器上，作为统一底座运行多个业务 Agent，共享渠道接入、模型路由、工具调用、记忆系统、沙箱执行能力。数据完全留在企业自己的基础设施，不锁任何云生态。

长期目标：走进 Apache 基金会，成为 Apache 顶级项目。

> 详细背景：`docs/DemandAnalysis.md`（需求）、`docs/TechnicalSolution.md`（技术方案）、`docs/IndustryResearch.md`（业界调研）、`docs/AiProgrammingGuide.md`（AI 编程指南）、`docs/oryxos.md`（项目定位）

---

## 技术栈

| 组件 | 选型 |
|------|------|
| 语言 / 运行时 | Java 21（必须，virtual thread 处理并发） |
| 框架 | Spring Boot 3.x |
| LLM 调用 | Spring AI Alibaba（仅用协议转换 + `@Tool` schema 生成） |
| HTTP 服务 | Spring MVC + Java 21 Virtual Thread |
| 命令行 | Picocli |
| YAML 解析 | SnakeYAML |
| 持久化 | SQLite + Spring Data JPA |
| 日志 | Logback + SLF4J（结构化 JSON） |
| 构建 | Maven 多模块 |

---

## 模块结构（9 个）

```
oryxos/
├── oryxos-core          # 核心抽象：OryxTool 接口、Session、Profile、ContextLoader、
│                        #   ReActLoop、PromptBuilder、ToolExecutor、AgentService
├── oryxos-provider      # 能力一：ProviderService、Function Calling 适配、
│                        #   多 Provider 显式映射
├── oryxos-memory        # 能力三：MemoryService 门面、LongTermMemory、
│                        #   MemoryTools（save/recall）
├── oryxos-tool          # 能力四：内置 Tool（文件/Shell/HTTP）、MCP Client、
│                        #   ToolRegistry、SandboxChecker
├── oryxos-channel-cli   # CLI Channel：oryxos chat 实现
├── oryxos-web           # 能力五：WebServer、ApiController、GlobalExceptionHandler、
│                        #   OpenAPI
├── oryxos-storage       # 持久化：SQLite、SessionRepository、
│                        #   ToolInvocationRepository、LlmCallRepository
├── oryxos-cli           # 命令行入口：Picocli 主入口、12 个子命令、ConfigLoader
└── oryxos-boot          # Spring Boot 启动模块：主类、自动配置、依赖聚合
```

模块之间通过接口解耦。新增 Channel 或 Tool 只加新模块，不改 `oryxos-core`。

**模块结构可按需演进**（宪法 v1.1.0）：模块划分跟随 Agent 的能力域，不锁死上面 9 个——可以新建模块（比如把沙箱独立为 `oryxos-sandbox`）或调整模块边界。新建/改名必须在对应特性的 plan 里声明理由，并同步更新本表与 `docs/TechnicalSolution.md` §10。跨模块契约（接口 + 值对象）放 `oryxos-core`，由下游模块实现（依赖倒置），禁止模块间循环依赖。

---

## 不可违背的原则（Constitution）

以下原则来自 `docs/AiProgrammingGuide.md` 和 `docs/TechnicalSolution.md`，所有代码必须遵守。

### 原则一：自实现 ReAct Loop

`ReActLoop` 必须自己实现，**不得**使用 Spring AI 的 Agent 抽象（如 `ChatClient.prompt().call()` 的自动工具执行）。核心循环约数十行 Java，完整掌握 Agent 工作机制，保留未来定制循环行为的空间。

### 原则二：Spring AI 只用两件事 ⚠️

Spring AI 在 OryxOS 里只做：

1. LLM Provider 协议转换（OpenAI / Anthropic / Gemini 等各家格式差异由它吸收）
2. `@Tool` 注解的 JSON Schema 生成

**必须禁用** Spring AI 的自动 tool 执行。Tool 的调度和执行完全由 `ReActLoop` + `ToolExecutor` 控制。违反此原则会导致 tool 被调两次。

```java
// 错误：不得用 Spring AI 自动执行 tool
chatClient.prompt(prompt).tools(tools).call().content();

// 正确：只用 Spring AI 做 LLM 调用，tool 调用结果自己处理
ChatResponse response = chatModel.call(new Prompt(messages, options));
// 然后自己检查 response 里的 tool call，自己执行
```

### 原则三：Provider 必须显式映射

多 Provider 并存时，**不得**靠扫描 Spring 容器里的 `ChatModel` Bean 类型来区分 Provider（因为 Bean 类型相同）。必须维护 `provider name → ChatModel` 的显式映射表：

```java
// 正确：显式映射
Map<String, ChatModel> providerMap = Map.of(
    "deepseek", deepseekChatModel,
    "qwen",     qwenChatModel,
    "kimi",     kimiChatModel
);
```

### 原则四：一个目录 = 一个 Agent；Skill 以本地软连接绑定并渐进披露

**一个目录 = 一个 Agent**：`.oryxos/agents/<name>/` 里 `AGENT.md` = frontmatter（运行配置）+ 正文（任务指令），外加可选 `skills/`（Skill 绑定视图）、`scripts/`、`REFERENCE.md`。`AgentLoader.deriveProfile(agentDir)` 把 frontmatter 派生成底座认识的 `Profile`；`.oryxos/profiles/` 取消。

公共 Skill 实体统一存放在 `.oryxos/skills/<name>/`。Agent 可见的 Skill 只由 `.oryxos/agents/<agent>/skills/<name>` 下指向公共实体的**相对软连接**表达；软连接集合是唯一绑定真相源，`AGENT.md` frontmatter 不再声明 `skills:`。

加载走三层渐进式披露：每轮 prompt 只注入当前 Agent 已绑定 Skill 的 `name + description + 本地绝对读取路径`；模型命中后用 `read_file` 读取 `SKILL.md` 正文；Skill 附属参考/脚本继续按需读取或运行。不得预载正文、不得新增 `use_skill`、Skill 不进 `ToolRegistry`。CRUD 与启动恢复必须检测 dangling/escaped/invalid-target/name-mismatch/stale-reference；公共 Skill 被引用时默认拒绝删除并返回引用 Agent。

### 原则五：审计表 Day One 写入

`tool_invocations` 和 `llm_calls` 两张审计表**核心阶段就必须写入**（不需要查询接口，但写入不能省）。不得以"日志够了"为由跳过落库，可审计是 OryxOS 的核心差异化能力。

### 原则六：不使用 Java SecurityManager；软连接必须校验真实路径

`SecurityManager` 在 JDK 17 起废弃、JDK 21 已不可用。Sandbox 通过 `SandboxChecker` 的 Path / Pattern 白名单实现：
- 文件操作：路径白名单（`file.allowed_paths`）
- Shell：命令首 token 白名单（`shell.allowed_commands`）
- HTTP：域名通配符白名单（`http.allowed_domains`）

文件目标存在时必须用 `toRealPath()` 校验真实路径仍位于白名单根；新建路径校验最近存在父目录的真实路径。Agent Skill 绑定只允许指向 `.oryxos/skills/` 的相对软连接，拒绝绝对链接和越界链接。

### 原则七：同步执行模型

核心阶段全程同步阻塞，配合 Java 21 Virtual Thread 处理并发。**不引入** Reactor / WebFlux / CompletableFuture 等异步编程模型（SSE 流式响应放扩展阶段）。

### 原则八：Tool 模块三合一

内置 Tool、MCP Client 合并在一个 `oryxos-tool` 模块，**不拆成多个模块**。`AGENT.md`（及 Agent 目录里的子指令）加载归 `oryxos-core` 的 `ContextLoader`。

---

## 工作区结构（运行时）

OryxOS 启动后在当前目录创建 `.oryxos/` 工作区：

```
.oryxos/
├── agents/             # 每个子目录 = 一个 Agent（AGENT.md + skills/软连接 + scripts/ REFERENCE.md）
├── skills/             # 公共 Skill 实体库：每个子目录 = 一个 Skill（SKILL.md + 可选附属资源）
├── memory/
│   └── MEMORY.md       # 长期记忆（Agent 通过 save_memory 写入，不得手动修改）
├── sessions/           # 会话数据（已迁入 SQLite，此目录备用）
├── logs/               # 结构化日志
├── mcp_servers.yaml    # MCP server 配置
├── oryxos.db           # SQLite 数据库
├── AGENTS.md           # Bootstrap：项目级 agent 行为说明
├── SOUL.md             # Bootstrap：agent 人格定义
└── USER.md             # Bootstrap：用户偏好（只读，agent 不写）
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oryx-labs/oryxos](https://github.com/oryx-labs/oryxos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
