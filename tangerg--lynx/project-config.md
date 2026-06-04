---
trigger: always_on
description: > Go monorepo，12 个 sub-module 各自有 `go.mod`：`core` / `agent` / `models` / `vectorstores` / `tools` / `pkg` / `rag` / `chatmemory` / `documentreaders` / `mcp` / `otel` / `lyra`。每个模块的具体形态见各自的 `CLAUDE.md`，本文件只放**跨模块共用**的约定。
---

# CLAUDE.md — lynx monorepo 项目级上下文

> Go monorepo，12 个 sub-module 各自有 `go.mod`：`core` / `agent` / `models` / `vectorstores` / `tools` / `pkg` / `rag` / `chatmemory` / `documentreaders` / `mcp` / `otel` / `lyra`。每个模块的具体形态见各自的 `CLAUDE.md`，本文件只放**跨模块共用**的约定。

---

## 第一法则 —— 绝不为一时方便留历史债务

> **最高优先级，凌驾于本文件其余所有约定之上。**

**项目处于快速开发 / 试错阶段，没有历史债务、也没有外部兼容包袱** —— store schema、暴露的 API（exported 函数 / 类型 / 字段 / 协议 wire shape）、命名，**全都可以调整**。正因如此：

- ❌ **绝不为"少改几处 / 降低前期开发量 / 避免迁移 / 赶进度"留下任何历史债务** —— 迁就外部库或旧名字的命名、为兼容留的字段、推测性 shim、"以后再清"的 TODO，一律不留。
- ✅ **发现设计不对，就在源头改对**，不在错的设计上叠补丁。**现在改成本最低，往后只会更贵。**
- 命名 / shape 按**本质第一性**决定；参考业界（Anthropic / Codex 等）**只取思想、不作命名锚** —— 名字恰好相同，只因它在独立评估下最优，不为兼容或省迁移。
- **唯一允许背的"债"是"设计还没想清楚"本身**；绝不允许"明知更好、却为省事不改"。
- 这条**不豁免**"破坏性公开 API 改动须先咨询用户"的约定（见下）：**先咨询，但默认倾向改对、而非将就。**

---

## 一句话定位

`lynx` 是一套**面向 AI agent / RAG / LLM 集成的 Go 基础设施**：`core` 定义协议、`models` 适配 38 个 LLM provider、`vectorstores` 适配 27 个向量库、`tools` 提供工具集、`agent` 跑 planner 驱动的 agent runtime、`lyra` 是 in-house 的 backend runtime（实现 Lyra Runtime Protocol）。所有模块共享一套**设计原则 + 重构节奏 + Go idiom 纪律**，下文规约。

## 子模块导览

| 模块 | LOC | 角色 |
|---|---|---|
| [`core/`](core/CLAUDE.md) | ~13k | Document / Message / Store / CallHandler 协议层 |
| [`agent/`](agent/CLAUDE.md) | ~13k | Planner / Blackboard / Runtime / Workflow |
| [`models/`](models/CLAUDE.md) | ~15k | 38 个 LLM provider 适配器 |
| [`vectorstores/`](vectorstores/CLAUDE.md) | ~21k | 27 个向量 DB 后端 |
| [`tools/`](tools/CLAUDE.md) | ~7k | LLM-callable 工具集 |
| [`pkg/`](pkg/CLAUDE.md) | ~9k | Generics / 并发 / 流式 工具库 |
| [`rag/`](rag/CLAUDE.md) | ~1.6k | 5 阶段 RAG pipeline |
| [`chatmemory/`](chatmemory/CLAUDE.md) | ~1.6k | Chat history DB 后端 |
| [`mcp/`](mcp/CLAUDE.md) | ~1.3k | MCP client + server 桥 |
| [`documentreaders/`](documentreaders/CLAUDE.md) | ~0.7k | Markdown / HTML / PDF readers |
| [`otel/`](otel/CLAUDE.md) | ~0.3k | OTel dev exporter |
| [`lyra/`](lyra/CLAUDE.md) | ~10k | Lyra Runtime backend |

## 共用强约定（违反 = 回归）

- **Go 1.26.3 统一版本**：所有模块 `go.mod` 同步；用 `iter.Seq2` / `slices.*` / `maps.*` / `atomic.Int32` 等现代 stdlib
- **依赖接口，不依赖具体类型**：跨包消费一定走 interface，**接口在消费方定义**（不在被消费方）。如果一个新模块要拿到 `*Engine` / `*Platform` / `*Service` 整体，先停下来想能不能拆成只用的几个方法
- **ISP 切碎接口**：典型例子 `approval.Console` vs `approval.Gate`（lyra），`tool.ToolSource` 一方法接口（lyra）。消费者只 import 自己用的那侧
- **`errors.New` 优先于 `fmt.Errorf("constant")`**。`fmt.Errorf` 只在真要格式化时用，包装其他错误必须 `%w` 才能 `errors.Is/As`
- **没有 Java 味**：禁 `impl.go` 文件 / `Impl` / `Service` / `Manager` / `Helper` / `Handler` 这种空白后缀 / `GetX/SetX` getter / `NewBuilder().With().Build()` 链。文件名描述内容（`inmemory.go` / `engine.go` / `sqlite/session.go`），struct 名描述本质
- **现代 Go**：`atomic.Int32` / `atomic.Pointer[T]` / `sync.Map` 优先于自家 atomic wrapper；`slices.*` / `maps.*` 替代手写 loop；`iter.Seq2` 替代 channel-based 流
- **Logging 走 OTel，不用 stdlib `log` / `log/slog`**：所有内部错误 / 事件记录通过 OTel span（`otel.Tracer(...).Start()` + `span.RecordError(err)` + `span.SetStatus(codes.Error, ...)` + `span.End()`），不用 `log.Printf` / `slog.Default()`。参考 `chatmemory/internal/tracing/` 和 `lyra/rpc/transport/http/tracing.go` 的 helper pattern。**例外**：(a) `otel/log/` 和 `otel/slog/` 子包本身是 OTel→stdlib 的导出器（设计就是 bridge）；(b) 公开 API 接受 `slog.Level` 等 stdlib 类型作为入参（如 `mcp.LogToClient`，是用户选择不是内部 logging）；(c) `core/model/chat/middleware.NewSlogLogger` 是给用户用 slog 的 optional convenience provider（用户也可以自己写 OTel 实现）
- **设计原则**（高内聚低耦合 / KISS / DRY / SOLID / YAGNI）—— 是判断标准，详见下方"## 设计原则"段
- **目前开发阶段，公开 API 可以调整**：不写 legacy 兼容代码、不写 migration、schema / exported type / 函数签名变了直接换；注释里不提"Legacy …"。**但任何破坏性公开 API 改动必须先咨询用户**（不只是大重构 —— 改一个 exported 函数签名 / 删一个 exported 类型 / 改 struct field 也算），列清楚 scope + 影响面 + 备选方案，等用户确认再动手。这条规则适用于**所有 sub-module**
- **加文档？先问** —— 每个 sub-module 已有 `CLAUDE.md`，本根级也已有。其他默认不写

## 共用强反向不变量（已知错的方向）

- ❌ **加 retry layer**：SDK 内部已有 retry 就够，不在 `pkg/retry` 引入 Transient / NonTransient 分类
- ❌ **structured output 自己开 converter 链**：`chat.JSONParser[T]` / `ListParser` / `MapParser` 已覆盖 spring-ai converter family，Reasoning 是 first-class
- ❌ **`DefaultOptions` 返回 `*Options` 指针**：必须返值（intentional immutability）
- ❌ **手写 `fmt.Errorf("xxx is nil")`**：换 `errors.New`。包装 err 一律 `%w`
- ❌ **新增模块直接 import `*Engine` / `*Platform` / `*Service` 整体**：定义自己包内的窄接口
- ❌ **接口里塞所有 method**：subscriber 只用 3 个、producer 用另外 2 个 —— 拆 ISP
- ❌ **复制公共类型**（典型：enum 双份）：留一份，import 一下
- ❌ **stub interface placeholder**（`// M5 wires this` 这种）：真要做时再定义；当前删
- ❌ **给 LLM provider 加 OAuth / token refresh**：用户填 API key、err 401 让 UI 提示重填

## 设计原则

判断"这段代码该不该这么写 / 这个 PR 该不该 merge"的硬尺子。每条都配 lynx 真实命中的例子。

> **本段是速查红线版。** 这些原则背后的**组织哲学 + 包设计规范 + 编码规范的"为什么"**(薄核 + 三形态变体 + 窄腰 + 一个扩展机制 + 库优于框架,经 embabel convergent design 与 Go 团队 MCP SDK 双重印证)见 [`DESIGN_PHILOSOPHY.md`](DESIGN_PHILOSOPHY.md)。设计新能力 / 新包 / 改公开 API 前,先用它的 §1 试金石与 §6 自检清单过一遍。

### 高内聚低耦合（High Cohesion / Low Coupling）

- **高内聚** = 一个 package / struct 内的东西**为同一个目的服务**。`internal/service/session/` 全是会话生命周期、`agentdoc/` 全是 AGENTS.md 发现 —— 这就是高内聚
- **低耦合** = 跨包依赖**通过最小接口**而不是具体类型。chat 服务依赖 `chat.Engine`（5 方法）不直接抱 `*engine.Engine`、autonomy 包内定义 `platform` 接口（2 方法）不抱 `*runtime.Platform`
- **二者矛盾时**：宁可包多一点（更高内聚）也别让一个包横跨多个 domain；宁可接口多一点（更低耦合）也别让一个具体类型变成跨包枢纽

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tangerg/lynx](https://github.com/Tangerg/lynx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
