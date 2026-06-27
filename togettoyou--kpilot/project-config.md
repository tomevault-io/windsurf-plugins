---
trigger: always_on
description: **Kubernetes 多集群管理 + GPU 算力调度 + 模型服务的一体化控制面。**
---

# KPilot

**Kubernetes 多集群管理 + GPU 算力调度 + 模型服务的一体化控制面。**

六个顶级平台：

| 平台 | 范围 | 详细文档 |
|---|---|---|
| 集群管理 (`/clusters`) | 通用 K8s 资源管理：节点、工作负载、监控、日志 | [docs/clusters.md](docs/clusters.md) |
| 算力调度 (`/compute`) | 基于 Volcano 的批量调度：Queue / Job / PodGroup CR 浏览，调度策略，vGPU 切分（volcano-vgpu-device-plugin），GPU-Hour 治理 | [docs/compute.md](docs/compute.md) |
| 模型服务 (`/models`) | 模型仓库、推理部署、调试、路由、训练任务 | [docs/models.md](docs/models.md) |
| 插件管理 (`/plugins`) | Helm chart 注册表，前三个平台的能力底座 | [docs/plugins.md](docs/plugins.md) |
| 系统管理 (`/system`) | KPilot 控制面自身的运维门面：系统监控（runtime / 业务计数器 / pprof）+ 系统日志（跨节点查询 + 实时跟随 + 下载），都走 PG-backed 1d 历史，server + worker 同结构 | [docs/system.md](docs/system.md) |
| KPilot AI (`/kpilot-ai`) | 能驾驶其余五平台的运维助手：OpenAI 兼容模型 + 工具循环（K8s 代理 / PromQL / LogsQL）+ 渐进式披露技能（go:embed 预置文档）+ 自进化 curator + 跨会话记忆 + 写操作 UI 审批 + 操作审计。模型直连（env 配 base_url/key，非 worker 隧道），全 PG-backed | [docs/kpilot-ai.md](docs/kpilot-ai.md) |

Server(中心控制面)+ Worker(集群侧 Operator),通过 **yamux 多路复用**(裸 TCP 连接;生产部署在集群 ingress 层做 TLS 终止,server 进程本身不带 TLS,见 `cmd/server/main.go::net.Listen`)连接 —— 每个 RPC / 流式会话开独立的 yamux stream,由 yamux 内置 flow-control + 公平调度处理 HOL/取消,应用层不再做这些。详见 [docs/transport-v2.md](docs/transport-v2.md)。

---

## 架构原则

- Server 的所有运行时数据 **100% 来自 Worker**，Server 不持有任何集群的 kubeconfig
- Worker 主动连接 Server（适合跨网络场景）
- 所有 K8s 操作由 Worker 代理执行

---

## 整体数据流

```
浏览器
  │  REST API
  ▼
Server (Go + PostgreSQL)
  │  yamux session over TLS（Worker 主动 dial 进来；
  │  每个 RPC / 流式会话开独立 stream）
  ▼
Worker (K8s Operator, Go)
  │  controller-runtime + client-go (Watch / dynamic / Helm SDK)
  ▼
K8s Cluster
```

---

## Worker 注册流程

1. 管理员在 Server UI 创建集群条目
2. Server 生成唯一 ClusterToken（只展示一次，可在 UI 重新生成）
3. 管理员将 ClusterToken + Server transport 地址配置到目标集群,部署 Worker
4. Worker 启动,携带 Token 发起 TCP + yamux 多路复用连接(详见 [docs/transport-v2.md](docs/transport-v2.md))
5. Server 验证 Token，将连接与集群绑定，标记集群 Online

---

## yamux 传输协议

**一条 TCP 连接 + hashicorp/yamux 多路复用**。worker 主动 dial server(NAT 友好);worker 端可选 `grpcs://` / `https://` 触发 `tls.Dial`(到 ingress 那一跳的加密),server 进程 `net.Listen("tcp")` 不带 TLS —— 生产部署在集群 ingress 层(nginx-ingress / Traefik / cloud LB)做 TLS 终止。dial 成功后 worker 端 yamux.Client + server 端 yamux.Server 各自起 session;之后**每个 RPC / 流式会话开一条独立的 yamux stream**(≤一行代码 `session.Open()`/`Accept()`)。

为什么是 yamux 而不是 bidi gRPC：HTTP/2 的多流复用我们用不到（gRPC 设计是"一条 stream 一个 RPC"，bidi stream 当多路复用器用就得在应用层重搓 chunk + per-request 调度 + cancel 帧 + accumulator 等十几项手搓优化）。yamux 直接给你这些。详细的对比 + 实测见 [docs/transport-v2.md](docs/transport-v2.md)。

### Stream 类型（`pkg/common/proto/v2/pilot.proto::StreamKind`）

| Kind | 用途 | 谁开 | 帧序列 |
|---|---|---|---|
| `STREAM_REGISTER` | 鉴权握手 | worker | RegisterRequest → RegisterAck → close |
| `STREAM_RESOURCE_REQUEST` | K8s 资源代理（list/get/apply/update/patch/delete/describe） | server | ResourceRequest → ResourceResponse → close |
| `STREAM_HTTP_REQUEST` | 反代 HTTP（buffered / streaming 两 mode） | server | HTTPRequestStart + body bytes → HTTPResponseStart + body bytes → close |
| `STREAM_PLUGIN_COMMAND` | Helm 插件 enable/disable（chart .tgz blob 直接走 stream 字节流） | server | PluginCommand + blob bytes → PluginCommandAck → close |
| `STREAM_PLUGIN_STATUS_PUSH` | Plugin CR status 上报（事件驱动） | worker | PluginStatusPush → close |
| `STREAM_PLUGIN_LOG_PUSH` | 插件安装日志流 | worker | PluginLogChunk*（哨兵 0-payload 后跟）PluginLogEnd → close |
| `STREAM_POD_LOGS` | Pod 日志流 | server | LogsStartRequest →（worker 端）LogsChunk*（哨兵 0-payload 后跟）LogsEnd → close |
| `STREAM_POD_EXEC` | Pod exec 终端 | server | ExecStartRequest →（双向）ExecStdin/ExecResize ↔ ExecOutput → ExecEnd → close |
| `STREAM_WS_PROXY` | 反代 WebSocket | server | WSStartRequest →（双向）WSFrame ↔ WSFrame → WSEnd → close |

### 关键设计点

- **每个 stream 携带 `StreamHeader{Kind, RequestID, Gzip}`**（第一帧 length-prefix prefix），不再有 `WorkerMessage`/`ServerMessage` oneof envelope
- **取消 = `stream.Close()`** —— yamux FIN 传到对端，对端用 cancel-watcher goroutine（阻塞 1-byte Read）观察。**FIN 不是 RST**：peer 收到 FIN 后仍可继续写,所以**绝对不能在 request 写完后立刻 CloseWrite**（详见 [docs/transport-v2.md 第 16 节](docs/transport-v2.md#16-上线后修订2026-05)）
- **大消息直接当字节流写**（HTTP body、chart blob、ResourceResponse data）—— yamux 内置 flow-control window 自动反压,不需要 chunked 帧 / accumulator
- **per-stream 公平调度**：yamux 内部 round-robin,大流不会 HOL block 小流,不需要应用层 prioritySender
- **per-stream 压缩**：codec 按 `StreamHeader.Gzip` 决定是否裹 gzip.Reader/Writer。reader 延迟初始化(避免 net.Pipe 上双向 EnableGzip 死锁,见 `pkg/transport/yamux/codec.go`)
- **离线检测**：yamux 内置 KeepAlive PING（默认 30s,我们配 20s）+ `Session.IsClosed()`；不再有应用层 Heartbeat
- **一条 stream 上多种 message type**（如 LogsChunk vs LogsEnd）通过 **0-payload 哨兵帧**切换 —— worker MUST 在切类型前发一个空帧。完整 contract 在 `pkg/server/gateway/stream.go` 的 package doc

### 包结构

| 包 | 职责 |
|---|---|
| `pkg/transport/yamux` | 协议无关的传输层：Session 包装、Codec(length-prefix protobuf + lazy gzip)、Stream 包装 |
| `pkg/server/gateway` | server-side yamux Accept + 业务流分发(Send* helpers + Open*Stream typed openers) |
| `pkg/worker/tunnel` | worker-side yamux Dial + STREAM_REGISTER 握手 + Handlers struct(OnResource / OnHTTP / OnPlugin / OnLogs / OnExec / OnWS) |
| `pkg/worker/proxy`, `pkg/worker/plugin` | 业务侧 `HandleStream(ctx, *transportv2.Stream)` 入口 |

---

## 项目结构

```
kpilot/
├── cmd/
│   ├── server/              # Server 入口
│   └── worker/              # Worker 入口
├── pkg/
│   ├── server/
│   │   ├── api/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [togettoyou/kpilot](https://github.com/togettoyou/kpilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
