---
trigger: always_on
description: 这是一个基于 go-starter 框架开发的 MPC（多方安全计算）基础设施系统，基于阈值签名技术（TSS - Threshold Signature Scheme），为 2B 和 2C 产品提供安全、可靠的密钥管理和签名服务。系统采用分布式架构，支持密钥分片、阈值签名、密钥轮换等核心功能。
---

# MPC 项目开发规范

## 项目概述

这是一个基于 go-starter 框架开发的 MPC（多方安全计算）基础设施系统，基于阈值签名技术（TSS - Threshold Signature Scheme），为 2B 和 2C 产品提供安全、可靠的密钥管理和签名服务。系统采用分布式架构，支持密钥分片、阈值签名、密钥轮换等核心功能。

**核心设计原则**：
1. **去中心化安全**：密钥分片分布式存储，无单点故障，密钥永不完整存在
2. **协议安全**：基于成熟的密码学协议（GG18、GG20、FROST）
3. **阈值容错**：支持 M-of-N 阈值配置，只要达到阈值即可签名
4. **分布式架构**：Coordinator 和 Participant 节点分离，节点间对等通信
5. **审计追踪**：所有操作记录不可篡改的审计日志
6. **零信任架构**：不信任任何请求，所有请求都需要验证
7. **高性能**：低延迟签名（< 200ms 目标），高吞吐量（1000+ 签名/秒）

## 架构规范

### 分层架构

严格遵循 go-starter 的分层架构：

```
API Layer (internal/api/handlers/)
  ↓
Infra Layer (internal/infra/) - 业务服务层
  ├── coordinator/      # Coordinator 服务（协调签名流程）
  ├── key/              # 密钥服务（DKG、密钥轮换）
  ├── signing/          # 签名服务
  ├── backup/           # 备份恢复服务
  ├── session/          # 会话管理
  └── discovery/        # 服务发现
  ↓
MPC Core Layer (internal/mpc/) - 核心逻辑层
  ├── protocol/         # 协议引擎（GG18/GG20/FROST）
  ├── node/             # 节点管理逻辑
  ├── chain/            # 链交互逻辑
  └── grpc/             # MPC 内部通信
  ↓
Model Layer (internal/models/) - SQLBoiler 生成
  ↓
Persistence Layer (internal/infra/storage/)
  ├── Key Share Storage (本地/S3/Vault)
  └── Metadata Store (PostgreSQL/Redis)
```

### Wire 依赖注入

**必须遵循 Wire 规范**：

1. **Provider 函数**：所有服务必须提供 Provider 函数
   ```go
   // ✅ 正确
   func NewKeyService(db *sql.DB, protocol *protocol.Engine) (*Service, error) {
       return &Service{db: db, protocol: protocol}, nil
   }
   ```

2. **Wire 配置**：在 `internal/api/wire.go` 中注册所有 Provider
   ```go
   var mpcServiceSet = wire.NewSet(
       NewMetadataStore,
       NewRedisClient,
       NewSessionStore,
       NewKeyShareStorage,
       NewNodeManager,
       NewProtocolEngine,
       NewDKGServiceProvider,
       NewKeyServiceProvider,
       NewSigningServiceProvider,
       NewCoordinatorServiceProvider,
       // ...
   )
   ```

3. **Server 结构体**：在 `internal/api/server.go` 中声明服务字段
   ```go
   type Server struct {
       // ...
       Key             *key.Service            `wire:"-"`
       Signing         *signing.Service        `wire:"-"`
       Coordinator     *coordinator.Service    `wire:"-"`
       Backup          *backup.Service         `wire:"-"`
       // ...
   }
   ```

## 目录结构规范

### 目录结构概览

```
internal/
├── api/
│   ├── handlers/         # HTTP 接口处理
│   │   ├── infra/        # 基础设施相关接口
│   │   │   ├── keys/     # 密钥管理接口
│   │   │   ├── signing/  # 签名接口
│   │   │   ├── backup/   # 备份接口
│   │   │   └── nodes/    # 节点接口
│   │   └── auth/         # 认证接口
│   ├── router/           # Echo 路由配置
│   ├── server.go         # Server 定义
│   └── wire.go           # Wire 依赖注入配置
├── infra/                # 基础设施服务层
│   ├── coordinator/      # 协调服务
│   ├── key/              # 密钥服务
│   ├── signing/          # 签名服务
│   ├── backup/           # 备份服务
│   ├── session/          # 会话管理
│   ├── discovery/        # 服务发现
│   └── storage/          # 存储实现
├── mpc/                  # MPC 核心逻辑层
│   ├── protocol/         # 密码学协议 (GG18, GG20, FROST)
│   ├── node/             # 节点逻辑
│   ├── chain/            # 链适配器
│   └── grpc/             # 内部 gRPC 通信
├── models/               # SQLBoiler 生成的模型
└── test/                 # 测试辅助工具
```

### 关键目录详解

#### MPC Core (`internal/mpc/`)
```
internal/mpc/
├── protocol/
│   ├── engine.go         # 协议引擎入口
│   ├── gg18.go           # GG18 协议实现
│   ├── gg20.go           # GG20 协议实现
│   ├── frost.go          # FROST 协议实现
│   └── types.go
├── node/
│   ├── manager.go        # 节点管理器
│   ├── registry.go       # 节点注册表
│   └── discovery.go      # 节点发现逻辑
└── chain/                # 链交互适配器
    ├── bitcoin.go
    ├── ethereum.go
    └── solana.go
```

#### Infra Services (`internal/infra/`)
```
internal/infra/
├── key/
│   ├── service.go        # KeyService 实现
│   ├── dkg.go            # DKG 流程控制
│   └── derivation.go     # 密钥派生
├── signing/
│   ├── service.go        # SigningService 实现
│   └── service_2of3_test.go
├── coordinator/
│   └── service.go        # 协调器服务
└── backup/
    ├── service.go        # 备份服务
    ├── sss.go            # Shamir Secret Sharing
    └── recovery.go       # 恢复逻辑
```

#### API Handlers (`internal/api/handlers/`)
```
internal/api/handlers/
├── infra/
│   ├── keys/
│   │   ├── post_create_key.go
│   │   ├── post_derive_key.go
│   │   └── get_key.go
│   ├── signing/
│   │   ├── post_sign.go
│   │   └── post_batch_sign.go
│   └── nodes/
│       ├── post_register_node.go
│       └── post_heartbeat.go
└── auth/
    ├── post_login.go
    └── post_register.go
```

**命名规范**：
- Handler 文件：`{method}_{route_name}.go`（如 `post_create_wallet.go`）
- Handler 函数：`{method}{RouteName}Handler`（如 `postCreateWalletHandler`）
- Route 函数：`{Method}{RouteName}Route`（如 `PostCreateWalletRoute`）

## 编码规范

### Go 代码规范

1. **遵循 Go 标准**：使用 `gofmt`、`golint`、`golangci-lint`
2. **错误处理**：始终检查错误，使用 `errors.Wrap` 或 `fmt.Errorf` 添加上下文
3. **上下文传递**：所有服务方法第一个参数必须是 `context.Context`
4. **接口定义**：接口定义在服务包内，使用小写接口名（如 `service`），实现使用大写（如 `Service`）

### 命名规范

**包名**：
- 使用小写、简短、有意义的名称
- 避免下划线和混合大小写
- 示例：`kms`, `key`, `encryption`, `policy`, `audit`, `hsm`

**类型名**：
- 使用驼峰命名，首字母大写（导出）
- 接口名通常以 `er` 结尾（如 `Adapter`, `Manager`, `Engine`）
- 示例：`KeyService`, `EncryptionService`, `PolicyEngine`, `HSMAdapter`

**函数名**：
- Provider 函数：`New{Type}`（如 `NewWalletService`）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kashguard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
