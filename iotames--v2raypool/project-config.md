---
trigger: always_on
description: v2raypool 是一个基于 v2ray-core 的 Go 语言代理池服务，提供 WebUI 和 gRPC 两种交互方式。
---

# AGENTS.md — v2raypool 知识库

v2raypool 是一个基于 v2ray-core 的 Go 语言代理池服务，提供 WebUI 和 gRPC 两种交互方式。

## 项目架构

```
main/                  # 入口，编译在此目录
├── main.go            # CLI flag 解析 + 默认启动服务
├── main_func.go       # 端口检查、日志初始化等工具函数
├── main_grpc.go       # gRPC CLI 客户端函数（getproxynodes 等命令）
├── conf.go            # .env 配置加载（easyconf），VP_ 前缀全部环境变量
└── build.sh           # 交叉编译脚本（Linux/Windows）

root package (v2raypool)
├── v2raypool.go       # ProxyPool 核心：节点管理、V2rayServer 控制
├── v2ray.go           # V2rayServer 结构体（进程创建、配置写入）
├── v2ray_config.go    # V4/V5 JSON 配置结构体、outbound 生成
├── v2ray_parse.go     # vmess/trojan/ss 等协议 URI 解析
├── v2ray_api.go       # gRPC API 客户端（通过 v2ray API 端口动态添加 inbound/outbound）
├── proxy_node.go      # ProxyNode 结构体 + 增删操作
├── speed.go           # 节点测速（HTTP GET 耗时）
├── nodes_storage.go   # gob 编码/解码持久化节点数据
├── grpc_server.go     # gRPC 服务端（ProxyPoolService）+ 隧道代理池全局管理
├── grpc_client.go     # gRPC 客户端辅助函数
├── tunnel_config.go   # 隧道代理池配置（TunnelConfig 结构体 + 默认值常量）
├── tunnel_pool.go     # 隧道代理池核心（HTTP 代理服务器，单端口随机出口 IP）
├── win_proxy.go       # Windows 系统代理设置（WinAPI）
├── notwin_proxy.go    # Linux/macOS 系统代理占位（空实现）
└── command_test.go    # 测试

decode/                # 协议解析
├── decode.go          # Base64 解码、订阅 URL/原始数据解析
├── clash.go           # Clash YAML 订阅解析（仅支持 trojan）
├── v2ray.go           # V2raySsNode 数据定义（核心节点结构体）
├── shadowsocks.go     # ss:// URI 解析
├── trojan.go          # trojan:// URI 解析
└── stream.go          # StreamConfig 流传输配置

v2rayapi/              # v2ray gRPC handler 协议适配
├── v2rayapi.go        # GetOutboundRequest — 构造 v2ray outbound 请求
├── freedom.go         # Freedom outbound（直连）
├── shadowsocks.go     # Shadowsocks outbound
├── trojan.go          # Trojan outbound
├── vmess.go           # VMess outbound
└── streamconfig.go    # 传输层流配置（tcp/ws/grpc）

webserver/             # WebUI（基于 glayui）
├── webserver.go       # EasyServer 初始化
├── router.go          # 路由注册（/api/nodes, /api/v2ray/*, /api/tunnel/* 等）
├── nodes.go           # 代理节点 API handler
├── v2ray_ctl.go       # v2ray 进程管理 API handler
├── tunnel_ctl.go      # 隧道代理池 API handler（启动/停止/状态查询）
├── sysproxy_ctl.go    # 系统代理切换 API handler（固定节点/隧道/无代理）
├── conf_ctl.go        # 配置修改/清理缓存 API handler
├── pub_funcs.go       # 公共工具函数
└── response_data.go   # 响应数据结构

conf/                  # 配置
├── conf_val.go        # Conf 结构体定义 + 常量默认值 + 读写

grpc/                  # 生成的 proto 文件
├── v2raypool.pb.go         # protocol buffer 消息
└── v2raypool_grpc.pb.go   # gRPC 服务/客户端桩代码

netutil/               # 网络工具
├── http.go            # 代理 HTTP 客户端
└── ip.go              # IP 工具

client/                # 实验性客户端（main.go）
main/resource/         # 前端静态资源（layui, html, css, js, ico, png）
```

## 数据流

### 主要控制流

1. **启动** (main/main.go:main → runServer):
   - `init()` 加载 `.env` → `conf.GetConf()`
   - 启动 gRPC 服务 (`vp.RunServer()`)
   - 可选启动 WebUI (`webserver.NewWebServer(webPort)`)
   - `ProxyPool.StartV2rayPool()` 启动 v2ray 内核进程 + 读取持久化节点文件 + 可选自动启动所有节点

2. **节点订阅与解析** (v2ray_parse.go:ParseV2rayNodes):
   - 从 URL 或文件获取原始数据 → `decode.ParseSubscribeByRaw()` (Base64 解码)
   - 每行 `protocol://` 格式 → `parseNodeInfo()` 按协议分发
   - 支持: `vmess`, `ss`, `trojan`, Clash YAML

3. **代理节点激活** (proxy_node.go:AddToPool):
   - 通过 v2ray API gRPC 端口动态添加 inbound (http/socks) + outbound (vmess/trojan/ss)
   - 每个节点独立本地端口（从系统代理端口 +1 开始累加）

4. **测速** (speed.go:testProxyNode):
   - 通过代理节点端口 HTTP GET 目标 URL → 记录耗时
   - 结果存入 `ProxyPool.speedMap`，支持按域名查询

5. **隧道代理池** (tunnel_pool.go:TunnelPool):
   - 对外暴露单一 HTTP 代理端口（如 `127.0.0.1:1080`），每次新请求随机选一个测速合格的节点转发
   - 复用 ProxyPool 的 `testOneNode()` 主动测速，结果自动写回 speedMap 供全局共享
   - 测速刷新间隔由 `VP_TUNNEL_REFRESH_INTERVAL` 控制（默认 1200 秒即 20 分钟）
   - 与多端口代理池模式完全兼容，可同时运行

## 核心数据结构

### `decode.V2raySsNode` (核心节点模型)
```go
type V2raySsNode struct {
    Protocol, Add, Host, Id, Net, Path, Ps, Tls, Type string
    V, Aid, Port                                      json.Number
}
```
所有协议最终归一化为这个结构体。

### `ProxyPool` (单例)
- `sync.Once` 懒加载，`GetProxyPool()` 获取
- 管理 `map[int]*V2rayServer` (PID → 进程), `ProxyNodes` (节点列表), `speedMap` (域名→节点)
- 全局读写锁 `sync.Mutex` + `IsLock` 标志
- `GetAvailableNodes(maxDelay)`: 获取测速合格且运行中的节点（隧道代理用）

### `TunnelPool` (隧道代理池)
- `NewTunnelPool(cfg)`: 创建实例，复用 `GetProxyPool()` 单例
- `Start()`: 启动 HTTP 代理服务（支持 CONNECT 隧道和 HTTP 转发）
- `pickNodeAddr()`: 每个请求随机选一个可用节点
- `RefreshNodes()`: 对运行中节点逐个调用 `pool.testOneNode()` 主动测速，结果写回 `speedMap`，按延迟筛选后更新 `nodeList`
- `refreshNodesLoop()`: 后台定时调用 `RefreshNodes()`，间隔由 `RefreshInterval` 控制
- `globalTunnelPool`: 全局实例，由 `grpc_server.go` 中的 `InitTunnelPool/StartTunnelPool/StopTunnelPool` 管理

### `SysProxyType` (系统代理类型枚举)
- `SysProxyNone(0)`: 无系统代理
- `SysProxyNode(1)`: 固定节点代理，`SetSysProxy(1, nodeIdx)` 激活指定节点为系统代理
- `SysProxyTunnel(2)`: 隧道代理，`SetSysProxy(2, -1)` 将系统代理指向隧道端口
- `SetSysProxy()` 负责先取消旧代理再切换，自动处理 Windows 系统代理设置（`win_proxy.go:SetProxy`）

## 关键配置（`.env` 文件）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| VP_V2RAY_PATH | bin/v2ray.exe | v2ray 可执行文件路径 |
| VP_GRPC_PORT | 50051 | gRPC 控制端口 |
| VP_WEB_SERVER_PORT | 8087 | WebUI 端口，0=禁用 |
| VP_V2RAY_API_PORT | 15492 | v2ray API 端口 |
| VP_HTTP_PROXY | http://127.0.0.1:30000 | 系统代理，协议支持 http/socks |
| VP_TEST_URL | https://www.google.com/ | 测速目标 URL |
| VP_SUBSCRIBE_URL | "" | 订阅地址 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iotames/v2raypool](https://github.com/iotames/v2raypool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
