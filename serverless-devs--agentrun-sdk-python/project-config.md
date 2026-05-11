---
trigger: always_on
description: 本文档详细介绍 AgentRun 的核心概念、架构设计和最佳实践。
---

# AgentRun 核心概念

本文档详细介绍 AgentRun 的核心概念、架构设计和最佳实践。

## 📚 目录

- [什么是 AgentRun](#什么是-agentrun)
- [核心概念](#核心概念)
- [架构设计](#架构设计)
- [部署方式](#部署方式)
- [网络配置](#网络配置)
- [最佳实践](#最佳实践)

## 什么是 AgentRun

AgentRun 是阿里云提供的 AI Agent 运行时服务，为 AI Agent 应用提供托管的运行环境。开发者无需关心底层基础设施，即可快速部署和运行各类 AI Agent 应用。

### 核心优势

- **🚀 快速部署** - 支持代码包和容器镜像两种部署方式，几分钟内完成部署
- **📈 弹性伸缩** - 自动根据负载调整资源，按需付费
- **🔒 安全可靠** - 企业级安全防护，多可用区容灾
- **🔌 易于集成** - 提供丰富的 SDK 和 API，轻松集成到现有系统
- **📊 监控运维** - 完善的日志、监控和告警体系

## 核心概念

### Agent Runtime（智能体运行时）

Agent Runtime 是 AgentRun 中的核心资源，代表一个运行中的 AI Agent 实例。每个 Agent Runtime 包含以下关键属性：

- **名称（agent_runtime_name）** - 唯一标识符，用于区分不同的 Agent
- **制品类型（artifact_type）** - 部署方式，支持 `CODE`（代码包）或 `CONTAINER`（容器镜像）
- **配置信息** - 包括代码配置、容器配置、网络配置等
- **状态（status）** - 运行时的当前状态
- **版本（version）** - 支持多版本管理

#### Agent Runtime 状态

| 状态 | 说明 |
|-----|------|
| `CREATING` | 创建中 |
| `READY` | 就绪，可正常提供服务 |
| `UPDATING` | 更新中 |
| `DELETING` | 删除中 |
| `FAILED` | 失败 |
| `DELETE_FAILED` | 删除失败 |

### Agent Runtime Endpoint（访问端点）

Endpoint 是 Agent Runtime 的对外访问入口，每个 Agent Runtime 可以创建多个 Endpoint 以支持不同的访问场景。

#### Endpoint 特性

- **公网访问** - 自动分配公网域名，支持 HTTPS
- **内网访问** - VPC 内网访问，低延迟高安全
- **路由配置** - 支持基于权重的流量分发
- **健康检查** - 自动检测 Agent 健康状态
- **协议支持** - HTTP/HTTPS/gRPC

#### Endpoint 状态

| 状态 | 说明 |
|-----|------|
| `CREATING` | 创建中 |
| `READY` | 就绪，可正常访问 |
| `UPDATING` | 更新中 |
| `DELETING` | 删除中 |
| `FAILED` | 失败 |

### Agent Runtime Version（版本）

版本管理允许您维护 Agent Runtime 的多个历史版本，支持版本回滚和灰度发布。

## 架构设计

```
┌─────────────────────────────────────────────────────────┐
│                        用户应用                          │
└───────────────────┬─────────────────────────────────────┘
                    │
                    │ SDK/API 调用
                    │
┌───────────────────▼─────────────────────────────────────┐
│                   AgentRun 控制面                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Runtime    │  │   Endpoint   │  │   Version    │  │
│  │  Management  │  │  Management  │  │  Management  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└───────────────────┬─────────────────────────────────────┘
                    │
                    │ 编排调度
                    │
┌───────────────────▼─────────────────────────────────────┐
│                   AgentRun 数据面                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │          Agent Runtime 实例池                    │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │  │
│  │  │ Agent A  │  │ Agent B  │  │ Agent C  │  ...  │  │
│  │  └──────────┘  └──────────┘  └──────────┘       │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │          负载均衡 & 路由                         │  │
│  └──────────────────────────────────────────────────┘  │
└───────────────────┬─────────────────────────────────────┘
                    │
                    │ 用户请求
                    │
┌───────────────────▼─────────────────────────────────────┐
│                    外部访问                              │
│              (公网/VPC 内网)                             │
└─────────────────────────────────────────────────────────┘
```

### 工作流程

1. **部署阶段**
   - 开发者通过 SDK 提交代码或镜像
   - AgentRun 创建 Agent Runtime 实例
   - 系统自动完成环境配置和依赖安装

2. **运行阶段**
   - Agent Runtime 进入 READY 状态
   - 创建 Endpoint 对外提供服务
   - 负载均衡器分发请求到 Agent 实例

3. **更新阶段**
   - 提交新版本代码或配置
   - 系统创建新版本实例
   - 平滑切换流量，无缝升级

4. **销毁阶段**
   - 删除 Endpoint，停止接收新请求
   - 优雅停止 Agent 实例
   - 释放相关资源

## 部署方式

AgentRun 支持两种部署方式，满足不同场景需求。

### 方式一：代码包部署（CODE）

适合快速开发和部署简单应用。

**特点：**
- 直接上传代码文件
- 支持多种编程语言（Node.js、Python、Java 等）
- 自动安装依赖
- 快速迭代

**示例：**

```python
from agentrun import agent_runtime

agent = client.create(
    agent_runtime.AgentRuntimeInput(
        agent_runtime_name="my-agent",
        code_configuration=agent_runtime.AgentRuntimeCode().from_file(
            language=agent_runtime.AgentRuntimeLanguage.NODEJS18,
            command=["node", "index.js"],
            file_path="/path/to/code",
        ),
    )
)
```

**支持的语言：**
- Node.js 14/16/18/20
- Python 3.10+
- Java 8/11/17
- Go 1.x
- .NET Core 3.1/6.0

### 方式二：容器镜像部署（CONTAINER）

适合复杂应用和生产环境。

**特点：**
- 完全自定义运行环境
- 支持任何容器化应用
- 版本管理更清晰
- 与 CI/CD 流程集成

**示例：**

```python
from agentrun import agent_runtime

agent = client.create(
    agent_runtime.AgentRuntimeInput(
        agent_runtime_name="my-agent",
        container_configuration=agent_runtime.AgentRuntimeContainer(
            image="registry.cn-hangzhou.aliyuncs.com/your-namespace/agent:latest",
            command=["python", "app.py"],
            port=8080,
        ),
    )
)
```

## 网络配置

### 网络模式

AgentRun 支持灵活的网络配置，满足不同安全和性能需求。

#### 公网模式（INTERNET）

- 自动分配公网域名
- 支持 HTTPS 加密
- 适合对外提供服务

#### VPC 模式

- 私有网络隔离
- 低延迟高带宽
- 适合内部服务调用

**配置示例：**

```python
from agentrun import agent_runtime

network_config = agent_runtime.AgentRuntimeNetworkConfig(
    network_mode=agent_runtime.AgentRuntimeNetworkMode.INTERNET,
    vpc_config=None  # 公网模式不需要 VPC 配置
)
```

### 健康检查

配置健康检查确保 Agent 正常运行：

```python
health_check = agent_runtime.AgentRuntimeHealthCheckConfig(
    enabled=True,
    path="/health",  # 健康检查路径
    initial_delay_seconds=10,  # 初始延迟
    period_seconds=30,  # 检查间隔
    timeout_seconds=5,  # 超时时间

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Serverless-Devs/agentrun-sdk-python](https://github.com/Serverless-Devs/agentrun-sdk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
