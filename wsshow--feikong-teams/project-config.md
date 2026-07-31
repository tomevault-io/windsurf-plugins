---
trigger: always_on
description: 基于 CloudWeGo Eino ADK 的多智能体协作系统，支持 CLI、Web UI、纯 API 服务和消息通道（Discord/QQ/微信）多种交互方式。
---

# fkteams

基于 CloudWeGo Eino ADK 的多智能体协作系统，支持 CLI、Web UI、纯 API 服务和消息通道（Discord/QQ/微信）多种交互方式。

## 构建与运行

```bash
# 开发
make web-build                          # 生成内嵌前端产物（web/dist 不提交）
go build ./...                          # 编译检查（需先生成 web/dist）
go vet ./...                            # 静态检查（需先生成 web/dist）
go run ./cmd/fkteams                    # 启动 CLI 聊天
go run ./cmd/fkteams web                # 启动 Web 服务（默认 :23456，需先生成 web/dist）
go run ./cmd/fkteams serve              # 启动纯 API 服务

# 构建
make native                             # 当前平台 -> release/fkteams_<goos>_<goarch>
make all                                # 预设平台（darwin/arm64, windows/amd64, linux/amd64）
make build t=linux:amd64                # 指定平台
make clean                              # 清理 release/

# 生成配置示例
go run ./cmd/fkteams generate config
```

## 项目架构

```
cmd/fkteams/main.go         # 入口，调用 internal/adapters/transport/cli/commands.Root().Run()
internal/app/               # 应用用例层，入口只调用这里
  config/                   #   TOML 配置加载、保存、热重载和示例生成
  version/                  #   应用版本和构建时间元数据
  appdata/                  #   应用数据目录、workspace/session/share/runtime 路径
  appstate/                 #   应用实例运行时状态（记忆服务 / 资源清理器）
  chat/                     #   RunTurn / 输入构建 / 入口上下文装配
    taskstream/             #   运行中任务事件流、队列、interrupt 状态管理
  agent/                    #   Runner 工厂、团队组装和 mode/agentName 解析
    catalog/                #   内置智能体定义、注册表、AgentBuilder 和成员工具元信息
  tools/                    #   工具注册、解析、策略标记和运行时无关内置工具实现
  memory/                   #   长期记忆检索、注入、提取、BM25 和 Markdown 持久化
  schedule/                 #   定时任务用例入口、后台任务结果收集，工具/HTTP/CLI 只调用这里
  skill/                    #   技能 provider、安装、移除、搜索结果和本地文件管理
  lifecycle/                #   Application 生命周期编排内核
                            #   用例层禁止依赖 agentcore 旧门面
                            #   用例层禁止依赖 pterm 等终端展示库
internal/domain/
  memory/                   #   MemoryEntry / Message / MemoryType 等长期记忆值对象
  schedule/                 #   Task / Status / HistoryEntry 等调度领域模型
  session/                  #   会话 ID 与 context 绑定
                            #   领域层禁止依赖框架/基础设施 SDK；稳定无状态小依赖可直接使用
internal/runtime/           # 运行时无关内核
  turn/                     #   回合执行内核、HITL handler、hooks/context 装配
  events/                   #   事件分发、Emitter、协议校验、友好错误归一化
  registry/                 #   runtime engine 注册表和默认 runtime 选择
  model/                    #   运行时无关 ChatModel 工厂注册表
  env/                      #   FEIKONG_* 环境变量读取
  log/                      #   日志 facade 和文件轮转
  atomicfile/               #   原子文件写入
  pathguard/                #   工作区路径逃逸防护
  typeutil/                 #   运行时类型名辅助
  hooks/                    #   HookBus 实现、context 绑定和 hook 调用
  checkpoint/               #   checkpoint 存储实现
  mdiff/                    #   文件差异和补丁基础能力
  resources/                #   运行期资源清理器
  retry/                    #   模型重试和迭代限制策略
                            #   运行时内核禁止依赖 agentcore 旧门面
internal/ports/             # 运行时无关端口契约
  hooks/                    #   HookPoint、HookHandler 和明确 payload 类型
  memory/                   #   LLMClient 等长期记忆外部能力端口
  runtime/                  #   Runtime / Engine / Runner / Model / Tool 等端口
  scheduler/                #   Scheduler / TaskExecutor 调度端口
  storage/                  #   SessionMessageReader 等存储读取端口
  tools/                    #   MCPProvider 等工具外部能力端口
internal/adapters/scheduler/
  filecron/                 #   文件存储 + cron 轮询调度器
internal/adapters/tools/
  builtin/git/              #   go-git backed git_* 工具适配器
  builtin/scheduler/        #   schedule_* 工具适配器，只委托 app/schedule
  builtin/ssh/              #   SSH/SFTP backed ssh_* 工具适配器
  mcp/                      #   MCP client、缓存和 runtime tool provider 桥接
internal/adapters/transport/
  cli/commands/             #   CLI 命令定义（urfave/cli/v3），参数解析和生命周期连接
  http/                     #   Gin HTTP 服务、Router、Handler、Middleware 和 origin 策略
  cli/runtime/              #   CLI 会话、输入、查询执行和交互运行时编排
  cli/eventview/            #   CLI 事件渲染和 JSON 输出回调
  cli/tui/                  #   CLI 终端 UI 组件、Markdown 渲染和交互控件
  cli/report/               #   CLI Markdown 报告导出 HTML 适配器
  cli/update/               #   CLI 自更新、下载、校验和替换适配器
  channel/                  #   Discord / QQ / 微信消息通道适配器和 Bridge
internal/adapters/runtime/
  eino/                     # CloudWeGo Eino ADK 适配层，唯一允许 import Eino 的目录
    runner.go               #   ADK AgentEvent -> events 协议转换，HITL resume 适配
    engine/engine.go        #   runtime.Engine 的 Eino 实现
                            #   adapter 与 middlewares 直接使用 internal/ports/runtime 与 domain 类型，禁止依赖 agentcore 旧门面
    middlewares/            #   autocontinue / summary / skills / dispatch / inject / fkfs
    middlewares/tools/      #   warperror / trimresult / patch / destructiveguard
    providers/              #   OpenAI / DeepSeek / Claude / Ollama / Ark / Gemini / Qwen / OpenRouter / Copilot
internal/adapters/model/
  providers/                #   模型 provider 注册、检测、模型列表和 Copilot 支撑
  memory/                   #   runtime ChatModel 到长期记忆 LLMClient 的适配
    providerkit/            #   provider 共用 HTTP/config 辅助
    copilot/                #   GitHub Copilot OAuth/token/HTTP 支撑
internal/adapters/storage/
  file/history/             #   append-only transcript、会话 metadata、历史投影和文件读写

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wsshow/feikong-teams](https://github.com/wsshow/feikong-teams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
