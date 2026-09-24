---
trigger: always_on
description: 蓝妹（lanmei-dream）是一个跨平台 IM 聊天机器人：通过反向 WebSocket 网关接入 OneBot 12/11 协议的 IM 平台（Onebots 网关连接微信、钉钉、Telegram 等，NapCat 直连 NTQQ）。消息路由由 Conduit 引擎驱动（行为树 + 管线）；对话能力来自 LLM 角色扮演、LOD 分级记忆与 RAG 检索；功能以插件扩展；另带内嵌管理面板与 Vue 前端。
---

# CLAUDE.md — 项目规则（AI 助手必读）

## 项目概况

蓝妹（lanmei-dream）是一个跨平台 IM 聊天机器人：通过反向 WebSocket 网关接入 OneBot 12/11 协议的 IM 平台（Onebots 网关连接微信、钉钉、Telegram 等，NapCat 直连 NTQQ）。消息路由由 Conduit 引擎驱动（行为树 + 管线）；对话能力来自 LLM 角色扮演、LOD 分级记忆与 RAG 检索；功能以插件扩展；另带内嵌管理面板与 Vue 前端。

## 技术栈

| 层 | 技术 |
|---|---|
| 语言 | Go 1.26（go.mod 声明 1.26.5，Dockerfile 用 golang:1.26）；管理面板前端 Vue 3 + TypeScript + Vite + TDesign |
| 消息引擎 | github.com/zrurf/conduit v0.2（行为树 + 管线，核心管线以 PassID 动态注册） |
| 网关 | lxzan/gws 反向 WebSocket 服务端，OneBot 12/11 协议自适配 |
| AI 组件 | CloudWeGo Eino；LLM 走 OpenAI 兼容接口（OpenAI / DeepSeek / Qwen / Moonshot / 火山方舟 / Ollama），Embedding 走 OpenAI 兼容接口，火山方舟 doubao-embedding-vision 走专用多模态实现 |
| 数据库 | PostgreSQL 18 + pgvector（HNSW 索引）+ pg_trgm；GORM + pgx，参数化查询 |
| 缓存 | Redis 7（Conduit StateStore、消息去重、话题状态；容器 allkeys-lru） |
| 对象存储 | RustFS（S3 兼容，aws-sdk-go-v2 s3），多媒体缓存与内网图片转 base64 |
| 插件 | 内置业务插件（Go，配置驱动）+ Wasm 插件（Extism + wazero，WIT ABI） |
| 权限 | Casbin RBAC（策略持久化 plugin_casbin_rule）+ Capability 配额与审计日志 |
| 管理 API | gofiber/fiber v3 内嵌服务（默认 :8090），前端在 manager/ |
| 日志 | zap + lumberjack（可选文件轮转） |

LLM / Embedding 通过 LANMEI_AI_LLM_* / LANMEI_AI_EMBEDDING_* 配置为 OpenAI 兼容端点，Key 未配置时角色扮演与 RAG 自动关闭；管理面板启用时 LLM 由 llm.ProviderManager 托管（Provider 存库、密钥 AES-256-GCM 加密、面板热切换并计费）。

## 目录结构

```
cmd/lanmei/          → 入口 main.go，组装全部依赖
internal/
  ai/                → ChatService 对话编排、LOD 压缩、记忆维护、流式分段、视觉理解、防注入
    intent/          → 意图分析（chat/command/tool/ignore + 群聊提及判断）
    llm/             → LLMClient 接口 + Eino 实现 + ProviderManager 多 Provider 热切换
    embedding/       → Embedder 接口 + OpenAI 兼容实现 + 火山方舟多模态实现
    memory/          → MemoryStore 接口 + 多路召回合并（向量/关键词/时间）
    prompt/          → Prompt 片段化组装（prompts.toml + assembly_template）
    skill/           → 技能系统（skills/ 目录 + skills.toml 启停）
    tool/            → AI 工具注册表（LLM ToolCalling）
  bot/               → Conduit 引擎装配、行为树、Pass 实现、消息回调与流式段落投递、去重
  command/           → 斜杠命令系统（注册/解析/分发）
  config/            → 配置加载（viper：config.toml + LANMEI_ 环境变量 + 命令行参数）
  database/          → GORM 连接、迁移、各实体 CRUD、LOD 查询、插件 KV
  gateway/           → 反向 WS 服务端 + OneBot 12/11 适配 + 连接管理 + 消息标准化
  infra/             → 基础设施生命周期（PG/pgvector + Redis + RustFS + zap）
  kb/                → 知识库：provider 抽象（local/feishu/sheet）、多路召回、隐式召回与工具
  manager/           → 管理面板后端（Fiber API、认证、Conduit 控制平面、计费、审计、Trace）
  media/             → 多媒体 MIME 与 RustFS 对象存储封装
  model/             → 数据模型与事实/置信度合并逻辑
  plugin/            → 插件系统（注册表、Wasm 运行时与 ABI、Casbin 授权、KV、配额）
  topic/             → 群聊话题系统（提及决策、状态机、归档）
  bizplugin/         → 内置业务插件（签到/欢迎/戳一戳/答题/海龟汤/表情/美图等）
manager/             → 管理面板前端（Vue 3 + Vite + TDesign）
config/              → 运行时配置（config.toml、skills.toml、onebots/napcat 配置）
prompts/ skills/ quizdata/ → Prompt 片段、技能、编程答题题库
schema/plugin/       → Wasm 插件 WIT 接口定义
docs/                → 架构与数据流文档
```

## 约定

- **注释**：准确简洁，中文为主，与所在模块风格保持一致；不复述代码，说明"为什么"。
- **文案**：聊天文案面向最终用户，允许活泼；日志/错误面向运维者，保持平实。
- **命名**：Go 导出符号大驼峰、包名小写短词；数据库表/字段用英文蛇形。
- **错误处理**：fmt.Errorf 带 %w 包装；日志统一用 zap 结构化字段，不吞错；管线内可降级错误用 conduit.NewSoftError（引擎记录日志但不中断管线）。
- **SQL 安全**：一律 GORM 参数化，绝不拼接用户输入；原生 SQL 仅限 DDL/索引/扩展；向量参数必须显式 ::vector 转换（否则报 operator does not exist: vector <=> text）。
- **依赖注入**：构造函数注入（NewXxx）；可扩展点用包级注册表（如 kb.RegisterProvider）；不用全局变量传递运行时依赖。
- **消息路由**：行为树决定走哪条管线，管线内 Pass 链式执行，不写 if-else 面条。
- **新增功能**：优先写内置业务插件（internal/bizplugin/，纳入 [plugin.builtins] 配置驱动注册，不在 main.go 硬编码）；需要进核心链路时新增 Pass 实现 conduit.Pass 并注册到管线；对外分发走 Wasm 插件（PLUGIN_DEVELOPMENT.md、schema/plugin/lanmei-plugin.wit）。
- **基础设施**：新增外部连接统一放 internal/infra/，经 infra.Setup() 初始化、Close() 释放，不在业务包直连。
- **幂等**：入口按 message_id 去重（Redis SETNX，存储故障时放行）。
- **平台标识**：用户以 (platform, platform_user_id) 唯一标识，platform ∈ qq/napcat/wechat/telegram（未识别回退 unknown）。
- **权限主体（Casbin）**：用户 user::<platform>::<platformUserID>；插件安装实例 plugin::<pluginID>::<installationID>；宿主 system::<name>。机器人超管另有静态配置 + bot_admin 表动态合并。
- **向量存储**：使用 pgvector 扩展（HNSW），不引入独立向量数据库；关键词/模糊检索用 tsvector 与 pg_trgm。

## 关键设计决策

**行为树结构（优先级从高到低）**

插件子树 → 流式段落重入 → 互动事件 → 管理员命令（/admin）→ 斜杠命令（/）→ 多媒体消息 → 话题门控（内部转意图分析）。

**核心管线（均以 PassID 动态注册，面板可编辑 Pass 顺序）**

| 管线 ID | 职责 |
|---|---|
| pipeline.admin | 超管校验 + 管理员命令执行 |
| pipeline.command | 斜杠命令解析 + 执行 |
| pipeline.intent_analysis | LLM 意图分析（RouterPass 动态路由） |
| pipeline.intent_command_exec | 自然语言命中命令时执行（参数由 LLM 提取） |
| pipeline.intent_ignore | 无需回复的消息仅存对话历史 |
| pipeline.roleplay | 流式角色扮演（yield 挂起 + 工具调用循环） |
| pipeline.roleplay_segment | 流式段落交付（子消息重入） |
| pipeline.media | 媒体下载/缓存/视觉理解后二次路由 |
| pipeline.notice | 互动事件兜底（插件子树优先消费） |
| pipeline.topic_gate | 群聊话题决策；私聊直接放行到意图分析 |
| pipeline.topic_ignore | 话题外消息静默保存 |
| pipeline.fallback | 引擎超时兜底话术 |

引擎参数：4 worker、消息超时 20s、超时走 pipeline.fallback、开启 Tracing（管理面板采集执行链路）。

**意图分析**：一次 LLM 调用把消息分为 chat / command / tool / ignore；群聊在同一次调用中附带"是否在跟机器人说话"的提及判断。LLM 不可用或结果非法时降级为 chat（LLM 完全未配置时不注册角色扮演管线，直接走 fallback）；意图分析有独立超时（默认 8s），避免吃满 20s 消息预算。

**角色扮演链路**：私聊直接进意图分析，群聊先过 topic_gate → RoleplayStreamPass 流式生成（独立 context，上限 60s）→ 段落经子消息重入引擎走交付管线，按"字数 × 打字速度"加抖动与上下限逐段发送（模拟真人节奏）→ LLM 返回 ToolCalls 时循环执行工具（最多 5 轮）→ 结束后存 L0 对话并异步触发记忆与压缩。

**LOD 记忆压缩**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LanshanTeam/lanmei-dream](https://github.com/LanshanTeam/lanmei-dream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
