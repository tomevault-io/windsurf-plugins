---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**api2mcp4j**（内部名称 server2mcp）是一个 Spring Boot Starter 框架，能自动将 Java Spring Controller 接口暴露为 MCP（Model Context Protocol）的 Tool、Resource、Prompt 和 Complete。设计理念是非侵入式、纯增强——类似 MyBatis-Plus 与 MyBatis 的关系。

**核心思想**：在 `interface` 作用域模式下，现有 `@RestController` 方法无需改动即可成为 MCP 工具。自定义 MCP 实体则使用专用注解（`@McpTool`、`@McpResource`、`@McpPrompt`、`@McpComplete`）。

---

## 顶层授权框架（必读）

> 本项目 2026-06-24 全面继承姊妹项目 `../real-agent` 的「开发心法」（董事长批准·方案 B）。本节及下方红线、审查、导航均在「自决默认协议」框架下解读。
> 协议正本：`~/.claude/CLAUDE.md` 顶部 · [ENFORCED] · 优先级高于本文件以下所有具体规范。

**核心契约**：
- **红线场景**（不可逆操作 / 跨系统协议变更 / 全局规范修改 / 战役级启停 / 生产部署 / 删公开 API / 升级 SNAPSHOT 依赖的破坏性变更）→ **必须事前请示董事长**，按下方「开发红线总表」执行多源验证等慢档流程。
- **红线之外** → AI 自决，不推送选项给董事长，事后简报（`[DONE] 做了 X · 因 Y · 风险 Z · 回滚成本 W`）。

**三方独立制衡**（中等以上复杂度任务，全局 Rule #6）：架构师（开发，不自写测试）/ 御史台（独立测试 + Code Review，不参与开发）/ CEO（协调验证闭环）。本项目质检 agent 见 `.claude/agents/`（御史台 `imperial-censor`）。

**Briefback 宣誓**：所有 agent 执行前须输出工作协议宣誓六项（任务理解 / 红线清单 / 交付标准 / 绝不会做的事 / 偏离预案 / **环境能力自检**），未宣誓 = 产出作废。

## 开发心法与规范导航

> 心法继承自 `../real-agent`，已剥离前端/设计招式、适配本 Java 库语境。**单一权威源原则**：下方「开发红线总表」是本项目红线的唯一权威源，各 spec 只引用不重复。

### 通用心法（`docs/rules/global/`，所有会话默认生效）

| 规则 | 路径 | 核心 |
|------|------|------|
| 搜索工具序位 | `docs/rules/global/search-tool-parity.md` | grep（字面量）/ 结构层 / LSP（语义）/ 多模块（图谱）四维不互替 |
| 破坏性删除多源验证 | `docs/rules/global/destructive-deletion.md` | [ENFORCED] 删公开 API 前 grep 多形式 + LSP + 跨模块/下游兼容性评估 |
| Session 续接铁律 | `docs/rules/global/session-continuity.md` | [ENFORCED] 续接先 `git log/status` 验证，禁凭记忆断言状态 |
| 重构 commit 顺序 | `docs/rules/global/refactor-ordering.md` | [ENFORCED] 契约提供者先行 / 原子 commit，避免契约真空期 |
| Agent 能力声明 | `docs/rules/global/agent-capability-declaration.md` | [ENFORCED] 物理不可能性原则，Briefback 第六项环境自检 |
| 工作留痕 | `docs/rules/global/work-log.md` | [ENFORCED] 报告类产出 / >30 行结构化产出落 `docs/logs/` |

### 核心规范（`docs/specs/`）

| 规范 | 路径 | 核心 |
|------|------|------|
| 注册纪律宪法 | `docs/specs/REGISTRATION_DISCIPLINE_SPEC.md` | [ENFORCED] 六维成熟度 Rubric + 三把利器（本框架灵魂：注册不遗漏） |
| 文件头规范 | `docs/specs/FILE_HEADER_SPEC.md` | [ENFORCED] AI 可读性文件头 `@header-start`/`@header-end` 分级模板 |
| 测试规范 | `docs/specs/TEST_SPEC.md` | [ENFORCED] 测试金字塔 + TDD 双 commit（`[RED]`/`[GREEN]`） |
| 工作留痕规范 | `docs/specs/WORK_LOG_SPEC.md` | [ENFORCED] 落文件模板 + Agent 五要素总结 + MUST-WIN 三源验证 |

### 参考索引（`docs/reference/`）

| 文档 | 路径 | 核心 |
|------|------|------|
| 架构总览 | `docs/reference/architecture.md` | 模块流向 + 6 大设计模式 + 数据流全过程 |
| 扩展点手册 | `docs/reference/extension-points.md` | 自定义解析器 / 转换器 / 过滤器 / Context |
| 新人入门 | `docs/reference/onboarding.md` | 3 步上手 + 必读清单 + 易踩坑 |

### 团队基建（`.claude/`）

- `.claude/agents/` — 10 个 Java 库适配 agent（三分组：指挥层 / 执行层 / 质检层），注册表 `.claude/agents/agent-groups.json`，验证 `node scripts/agent-groups.mjs verify`。
- `.claude/task.json` — 物理模块注册表 · `.claude/hooks/session-start.sh` — 会话启动提示。

## 构建与测试命令

```bash
# 构建全部模块（尚未推送至 Maven Central，必须本地安装）
mvn clean install

# 仅构建 WebMVC Starter
cd server2mcp-spring-boot-starters/server2mcp-starter-webmvc && mvn clean install

# 运行测试（测试位于 server2mcp-core）
cd server2mcp-core && mvn test

# 运行单个测试类
cd server2mcp-core && mvn test -Dtest=GenSchemaUtilsTest

# 打包测试/演示应用
cd server2mcp-test && mvn clean package
```

**注意**：要求 Java 17。根 pom 全局启用了 `-parameters` 编译参数，用于反射获取方法参数名。

## 模块架构

```
server2mcp-parent (根 pom, v1.1.4-SNAPSHOT)
├── server2mcp-common          → 常量、工具类（ConvertUtil, JacksonUtils, GenSchemaUtils）
├── server2mcp-core            → 核心引擎：注解、解析器、扫描器、回调、Provider
├── server2mcp-autoconfigure   → Spring Boot 自动配置（Server2McpAutoConfiguration）
├── server2mcp-spring-boot-starters/
│   ├── server2mcp-starter-webmvc    → Spring MVC 应用的 Starter
│   └── server2mcp-starter-webflux   → WebFlux 应用的 Starter
└── server2mcp-test            → 演示应用（不在根 pom 的 modules 中，需单独构建）
```

**依赖流向**：common ← core ← autoconfigure ← starters

## 关键依赖

| 依赖 | 版本 | 用途 |
|---|---|---|
| Spring Boot | 3.4.4 | 基础框架 |
| Spring AI | 1.1.0-SNAPSHOT | MCP 集成层 |
| MCP Java SDK | 0.14.0-SNAPSHOT | MCP 协议实现 |
| JavaParser | 3.25.5 | Javadoc 注释解析 |
| VicTools JsonSchema | 4.37.0 | JSON Schema 生成（Swagger 支持） |

全部使用 SNAPSHOT 版本，来源于 Spring 仓库，必须本地 `mvn clean install`。

## 核心设计模式与处理链路

### 1. 注解驱动注册（ImportBeanDefinitionRegistrar 模式）

```
@ToolScan → McpToolScanRegistrar → McpToolScanConfigurer (BeanDefinitionRegistryPostProcessor)
    → ClassPathToolScanner → ToolBeanNameGenerator → IToolContext.addTool()
```

`@McpResourceScan`、`@McpPromptScan`、`@McpCompleteScan` 遵循完全相同的模式。

### 2. 双层解析器链（责任链模式）

**描述解析器**（`AbstractDesParser`，按 order 0-5 排列）：
`McpToolDesParser → ToolDesParser → JacksonDesParser → JavaDocDesParser → Swagger3DesParser → Swagger2DesParser`

**参数解析器**（`AbstractParamParser`，按 order 0-6 排列）：
`McpToolParamParser → ToolParamParser → MvcParamParser → JacksonParamParser → JavaDocParamParser → Swagger3ParamParser → Swagger2ParamParser`

解析器通过 `@ConditionalOnParser` 条件注册，由 `plugin.mcp.parser.param` / `plugin.mcp.parser.des` 配置控制。

### 3. 上下文容器模式（工厂模式）

每种 MCP 实体类型均有：`I{Type}Context` 接口 → `{Type}ContextFactory` 工厂 → `{Type}Context` 实现。上下文作为 Spring Bean 持有注册定义。

### 4. 回调架构（模板方法模式）

```
AbstractMcpToolMethodCallback
├── SyncMcpToolMethodCallback
└── AsyncMcpToolMethodCallback
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheEterna/api2mcp4j](https://github.com/TheEterna/api2mcp4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
