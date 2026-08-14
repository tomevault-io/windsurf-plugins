---
trigger: always_on
description: 本文件定义 `agent-adaptor` v1 的架构边界、公共语义、迁移纪律与发布门禁。后续设计、实现、重构、评审和文档均必须以本文件为准。
---

# agent-adaptor AGENTS

本文件定义 `agent-adaptor` v1 的架构边界、公共语义、迁移纪律与发布门禁。后续设计、实现、重构、评审和文档均必须以本文件为准。

## 1. 项目定位

`agent-adaptor` 是一个可嵌入宿主的 Go SDK。

它负责：

- 以统一 API 调用不同本地 coding agent
- 统一构造、批处理、流式执行与有状态对话语义
- 提供可选的 Thread、workspace、tool、skill、MCP、profile、runtime service 注入点
- 提供稳定的 Driver SPI 与一致性测试
- 允许宿主嵌入 CLI、桌面应用、HTTP/gRPC 服务、工作流和定时任务

它不负责：

- 在 core 内内置面向宿主的通用 HTTP/gRPC server、队列、调度器、租户、鉴权或数据库；`WithTools` 私有、loopback-only 的 MCP transport 属于 Agent capability 交付实现，不构成宿主服务框架
- 自动决定一次任务应使用哪个 Agent
- 定义宿主的团队角色、业务流程或路由策略
- 强制任何持久化、分布式锁或服务框架依赖

Bridges 可以提供协议适配和便捷 handler，hosttools 可以提供可选宿主组件，但这些能力不得反向污染 core 执行语义。

## 2. 六个核心名词

v1 的消费者心智只能由以下六个核心名词组成：

| 名词 | 含义 |
|---|---|
| `Agent` | 一个配置完整、构造后即可执行的智能体 |
| `Thread` | 一段由宿主 key 标识、可持续续接或分叉的对话 |
| `Stream` | 一次正在进行的执行 |
| `Event` | 执行过程中发生的一件 typed 事件 |
| `Result` | 一次执行的最终结果与审计信息 |
| `Driver` | 一种 agent CLI/provider 的接入实现，属于扩展方 SPI |

硬约束：

- 不存在中央 SDK 对象。
- 消费者执行动词只有 `Run` 与 `Stream`，不得增加 `Start` 或其他平行执行入口。
- 不得重新引入 binding、默认 Agent、字符串查找 Runner 等平行抽象。
- 根包面向应用开发者；SPI 必须收敛在 `driver/`，不得再次把扩展方合同铺回根包 godoc。

## 3. 包布局与依赖方向

目标包布局：

```text
github.com/agent-dance/agent-adaptor          package adaptor
├── driver/                                    Driver SPI
├── codex/ claude/ cursor/ codebuddy/          各驱动 Config 与 Driver(Config)
├── tool/                                      宿主定义 Tool 词汇
├── skill/                                     skill 词汇与来源
├── mcp/                                       MCP 声明
├── profile/                                   profile 与资源声明
├── threadstore/ memory/                       Thread 持久化合同与内存实现
├── bridges/{sse,agui,a2a,subagentstream}/     协议桥
├── hosttools/{a2adelegation,sessionrecorder}/ 宿主可选组件
├── clients/a2a/                               A2A 客户端
└── adaptertest/                               Driver 一致性套件
```

依赖方向必须单向：

- 根包可以依赖 `driver`、词汇包和 `internal/engine`。
- `internal/engine` 不得 import 根包。
- `driver` 不得依赖根包或具体 provider 包。
- `tool`、`skill`、`mcp`、`profile`、`threadstore` 不得反向 import 根包。
- bridges 与 hosttools 只能消费公开 `Runner`、`Stream`、`Event`、`Result` 等合同，不得调用内部 engine 或直接派发 Driver。
- 公共类型不得通过 alias、字段或方法签名泄露 `internal/*` 类型。

## 4. 唯一构造语义

消费者唯一构造入口：

```go
agent := adaptor.New(
	codex.Driver(codex.Config{Model: "gpt-5.4"}),
	adaptor.WithWorkspace("/repo"),
)
```

合同：

```go
func New(d driver.Driver, opts ...Option) *Agent
```

- 内置驱动包拥有自己的真实 `Config` 类型，并以 `Driver(Config) driver.Driver` 返回已捕获配置的 Driver。
- 第三方扩展方直接实现 `driver.Driver`，与内置驱动走同一个 `adaptor.New`。
- `New` 对 nil Driver 等编程错误可以 panic；运行环境、能力或配置错误必须在执行或 Inspect 时结构化返回。
- Driver 配置不得以 `internal/engine` 类型别名对外暴露。
- Inspect、Run、Stream 和 capability probe 必须观察同一份构造期配置，不得向已配置 Driver 的 probe 静默传入丢失语义的 nil config。

## 5. 选项与唯一执行管线

选项采用一套词汇、两个作用域：

- `Option`：只允许用于 `New`
- `CallOption`：只允许用于 `Run` / `Stream`
- `SharedOption`：同时适用于构造默认值和单次调用覆盖

作用域错误必须尽可能成为编译错误。合并规则只有一条：

> 近处覆盖远处；skills 追加；其余能力按公开合同替换或显式合并。

所有执行必须收敛到同一内部管线：

1. 读取 Agent 默认值与本次 CallOption
2. 生成唯一的 resolved invocation
3. 校验 Driver 能力、policy、schema 与审批模式
4. 解析 workspace、profile、Tools、skills、MCP 与 runtime services
5. 若接收者是 Thread，协调 store、lease、resume/fork 与兼容性
6. 通过唯一 Event sink 调用一次 `driver.Run`
7. 形成 Event、Result、RunError 与 checkpoint
8. 原子持久化有效 Thread 状态
9. 关闭事件流并释放 workspace、runtime service、lease 等资源

硬约束：

- `Run` 必须等价于 `Stream` + drain Events + `Result()`，不能拥有第二份默认值合并、资源解析、Driver 派发或结果归档逻辑。
- SDK 的统一 Event 管线与 provider transport 的 streaming 选择不是同一个概念；transport 必须按 resolved invocation 和 Driver capability 明确协商，不能仅因调用 `Run` 或 `Stream` 就无条件切换 provider 协议。
- `Agent` 与 `Thread` 都实现同一个 `Runner`。
- Thread 只是在统一管线中增加状态协调阶段，不得复制执行管线。
- bridges、hosttools、structured output 和 Inspect 不得产生第二套执行入口或默认值语义。

进程生命周期合同：

- Claude、CodeBuddy 与 Codex 对显式 Thread 默认允许复用常驻进程；Cursor 与无状态 Agent 调用仍逐轮启动。
- `WithSpawn()` 是双作用域选项，显式强制使用本轮新进程；该进程不得注册为后续轮次的常驻 writer。
- 常驻只是 Driver 内部 transport 生命周期选择，不得形成平行执行入口、事件流或结果合同。
- `Agent.Close(ctx)` 必须幂等回收 Driver 管理的全部常驻进程；Close 开始后所有 Agent/Thread 新运行稳定返回 `ErrAgentClosed`。
- Thread record 重绑、配置漂移、临时单次形态和预热之间必须保持单 writer：旧进程完成有界退出后才可启动 replacement。
- prompt 交付前的常驻启动失败可以安全回退一次；prompt 可能已经交付后不得自动重放。

## 6. Thread 语义

默认无状态。只有显式注入 `WithThreadStore(store)` 后才启用有状态对话。

公共动作：

```go
th := agent.Thread(key)                 // 有则续、无则建
th := agent.Thread(key, ResumeOnly())   // 只续不建
branch := th.Fork(newKey)               // 从父对话分叉
checkpoint, err := th.Checkpoint(ctx)
```

合同：

- Thread key 是宿主提供的单一、不透明业务字符串，SDK 必须逐字保存和比较。
- 宿主主动开启无上下文的新对话时必须分配新的 Thread key；core 不提供同 key 的 `NewThread`、`start_new` 或重绑入口。continue-or-start 仅可在配置不兼容或 provider 拒绝 resume 的安全回退中原子替换内部 checkpoint。
- 内部存储或 bridge 不得通过未转义分隔符拼接多个维度；任何复合 key 必须使用无碰撞编码。
- Driver 的 resume ID 是内部 checkpoint 细节，不得升级成第二个消费者身份体系。
- 同一 Thread 同时只能有一个持有有效 lease 的运行。
- lease 的 acquire、renew、finalize、release 必须以 owner 和 token 防止过期运行覆盖新状态；释放必须有界且错误可观察。
- `Fork(newKey)` 必须验证父 checkpoint、Driver、配置、identity 与 fingerprint 兼容性，并持有必要的父记录协调租约。
- Fork 不得归档或修改父 Thread；目标 key 已存在时必须执行明确、有测试的冲突策略，不能静默留下多条 active record。
- continue-or-start 在 provider 拒绝 resume 时最多按合同回退一次到新会话；旧状态只能在新 checkpoint 成功持久化后归档。
- Store 的 Finalize 必须对 save、archive、rebind 与 lease 校验提供原子语义。
- 不完整、冲突或同时指定多个 selector 的请求必须在获取资源前稳定拒绝；不得创建无法按 key 找回的孤儿记录。

兼容 fingerprint 必须确定性、跨进程稳定，并覆盖所有影响续接正确性的已解析状态，包括：

- Driver 类型及构造配置
- 模型与 identity
- 实际解析后的 workspace，而非仅调用方原始字符串

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-dance/agent-adaptor](https://github.com/agent-dance/agent-adaptor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
