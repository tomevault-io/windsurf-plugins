---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 0. Codex 本地覆盖规则

如果当前运行环境是 Codex,必须先读取仓库根目录的 `CLAUDE.local.md`,并将其中的本地环境、常用命令、工具偏好作为本文件的补充规则。若 `CLAUDE.local.md` 不存在或不可读,需在回复中明确说明,再继续按本文件执行。

## 1. 仓库概览

Datasophon 是大数据 / 云原生平台自动化部署与运维管理系统,目标是在一组裸机或虚拟机上,以"控制面 + 工作面"的方式自动完成节点初始化(防火墙、JDK、Docker、K8s 基础环境、镜像仓库、MySQL/NTP 等基础设施)、大数据/云原生服务部署(Hadoop、Spark、Flink、Hive、Doris、Kafka、Zookeeper、Kubernetes 工作负载等 27+ 个内置服务)、服务的启停、配置下发、健康巡检、告警、日志聚合与集群级 DAG 编排变更管理。它通过一组声明式服务元数据(`meta/datacluster/<SERVICE>/service_ddl.json`)驱动安装策略,做到「新增一种服务 ≈ 写一份 DDL + 一个策略类」。

技术栈摘要:Java 21 + Spring Boot 3.4.5、Go 1.21、React 19、gRPC 1.68.1 / Protobuf 3.25.5、MyBatis-Plus 3.5.9、Druid、Flyway 9、fabric8 kubernetes-client。详细见 [README.md](./README.md) 与 [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)。

## 2. 模块矩阵

| 模块 | 路径 | 角色 | 运行形态 / 产物 | 端口 |
|---|---|---|---|---|
| `datasophon-api` | `datasophon-api/` | Master 主服务,集群编排核心 | Spring Boot 进程;`target/datasophon-manager-<version>.tar.gz`(assembly 内嵌前端 `dist/`) | HTTP `8080`(`/ddh`,`application.yml` 默认)、gRPC `18081` |
| `datasophon-worker` | `datasophon-worker/` | Worker 节点进程,本地执行安装/启动/停止等命令 | 每节点 1 个 `main()` 进程(`WorkerApplicationServer`),非 Spring Boot;`target/datasophon-worker.tar.gz` | gRPC `18082`、JMX exporter `8585` |
| `datasophon-grpc-api` | `datasophon-grpc-api/` | gRPC proto 契约 + 共享 stub | 纯库,无进程;stub 已 checked-in | — |
| `datasophon-common` | `datasophon-common/` | 公共库(K8s 客户端、命令模型、Nexus 客户端、序列化) | 纯库,无进程;不引入 Spring Web | — |
| `datasophon-cli-go` | `datasophon-cli-go/` | 节点初始化 CLI(Go 重写,替代旧 Java CLI) | Go 单二进制 `datasophon-cli`;`dist/datasophon-cli-{linux,darwin}-{amd64,arm64}` | — |
| `datasophon-k8s-agent` | `datasophon-k8s-agent/` | K8s 内 Agent,RSA 签名鉴权远端执行 | Spring Boot Web Pod;Helm Chart + Docker 镜像 | HTTP `12552`(可由 Helm 配 NodePort `32552`) |
| `datasophon-ui-v2` | `datasophon-ui-v2/` | **当前默认前端**(Umi Max 4 + antd 6 + ProComponents 3) | 静态资源;`npm run build`;Node ≥ 22,npm + Biome(禁 pnpm/yarn/ESLint/Prettier) | 开发服务器 `8000` |
| `datasophon-ui` | `datasophon-ui/` | 旧前端(React 19 + Vite + pnpm),已从 Maven 模块列表移除 | 仅历史参考,不再迭代 | `5180` |
| `datasophon-assembly` | `datasophon-assembly/` | Maven 顶层 assembly 打包模块 | 纯 `pom` 模块,无 Java 源码;`target/datasophon-<version>-package.tar.gz` | — |

> 顶层 `pom.xml` 当前模块列表:`datasophon-common` → `datasophon-grpc-api` → `datasophon-worker` → `datasophon-cli-go` → `datasophon-ui-v2` → `datasophon-api` → `datasophon-k8s-agent` → `datasophon-assembly`(后者 `package` 依赖前面所有 `tar.gz` 产物)。

## 3. 常用命令

所有 Maven 命令均从仓库根目录用 `./mvnw`(Maven Wrapper 3.8.4,见 `.mvn/wrapper/`)。`JAVA_HOME` 必须指向 JDK 21。

### 3.1 全量 / 后端

```bash
# 全量构建(编译 + 打包 + 内嵌前端)
./mvnw clean package -DskipTests

# 仅后端(前端不参与)
./mvnw clean package -DskipTests -pl datasophon-api -am

# 单元测试
./mvnw test

# 代码格式化(Spotless,2.27.2)
./mvnw spotless:apply

# 国内网络加速
./mvnw -Pgoogle-mirror <phase>
```

### 3.2 单模块构建(节省时间)

```bash
# 仅本模块 + 依赖联编
./mvnw -pl <module> -am <phase>

# 例:仅 worker
./mvnw -pl datasophon-worker -am clean package -DskipTests

# 仅 api(会顺带构建 datasophon-ui 并内嵌 dist)
./mvnw -pl datasophon-api -am clean package -DskipTests

# 仅 grpc stub(无 protoc,直接 compile 即可)
./mvnw -pl datasophon-grpc-api -am compile

# proto 改动后重新生成 stub
./mvnw -pl datasophon-grpc-api -am generate-sources -Pgenerate-proto
```

### 3.3 前端(`datasophon-ui-v2`,当前默认)

Node ≥ 22,包管理器固定 **npm**(`package-lock.json`),lint 用 **Biome**(无 ESLint / Prettier)。详细规则(禁改 `src/services/ant-design-pro/` 生成目录、`npx antd info` 先行等)见 [datasophon-ui-v2/CLAUDE.md](./datasophon-ui-v2/CLAUDE.md)。

```bash
cd datasophon-ui-v2
npm install
npm start            # dev + mock
npm run dev          # dev,无 mock(联调本地后端时用)
npm run build        # 生产构建
npm run lint         # Biome + tsc,提交前必须通过
npm run test         # 单元测试
npm run openapi      # 重新生成 src/services/(生成目录禁止手改)
```

> 旧版 `datasophon-ui`(pnpm + Vite)已从 Maven 模块列表移除,仅作历史参考;新前端工作一律在 `datasophon-ui-v2`。

### 3.4 CLI(`datasophon-cli-go`)

模块自带 `Makefile`,也可直接 `go build`。Go 版本要求 `1.21+`。

```bash
cd datasophon-cli-go
make build           # 当前平台二进制 → dist/datasophon-cli
make release         # 交叉编译 4 个目标到 dist/(linux/darwin × amd64/arm64)
make test            # go test ./...
make vet             # go vet ./...
go test -cover ./... # 测试 + 覆盖率

# 手工交叉编译
GOOS=linux  GOARCH=amd64 go build -o dist/datasophon-cli-linux-amd64  ./cmd/datasophon-cli
GOOS=linux  GOARCH=arm64 go build -o dist/datasophon-cli-linux-arm64  ./cmd/datasophon-cli
```

> CLI 命令名前置环境变量 `DDH_HOME`,为空直接 `Exit(1)`。子命令 `create cluster [plan|apply]` 支持 plan/apply 两阶段与断点续跑;全局 `--dry-run` 只打印不执行。

### 3.5 跑起来(默认账号 / 端口)

| 项 | 值 |
|---|---|
| 默认账号 | `admin` / `DJEutbydS@U%f7Jb` |
| API HTTP | `8080`,上下文 `/ddh`(本地 IDEA 启动即 `http://127.0.0.1:8080/ddh`) |
| Master gRPC | `18081` |
| Worker gRPC | `18082` |
| MySQL | `3306`(`application.yml` 中 `${mysql.ip:localhost}:${mysql.port:3306}` 占位) |
| Doris FE(本地可观测栈) | MySQL 协议 `9030`:`mysql -h127.0.0.1 -P9030 -uroot`(OTel 指标/日志/Trace 落 Doris) |

本地可观测栈(Doris + OTel Collector 等)启停:

```bash
docker compose -f deploy/compose/docker-compose.observability.yml up -d
docker compose -f deploy/compose/docker-compose.observability.yml down
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [88fantasy/datasophon](https://github.com/88fantasy/datasophon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
