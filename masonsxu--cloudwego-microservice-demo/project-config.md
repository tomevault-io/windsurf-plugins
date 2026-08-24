---
trigger: always_on
description: AI 辅助开发规范文件，为 AI 编程助手提供项目上下文和开发约束。
---

# AGENTS.md

AI 辅助开发规范文件，为 AI 编程助手提供项目上下文和开发约束。

## 项目概述

基于 Go 1.24+ 的微服务项目，采用 CloudWeGo 技术栈和 Go Workspace 管理两个独立模块：

- `gateway/`：HTTP 网关服务（Hertz，端口 8080）- 鉴权、权限检查、HTTP→RPC 协议转换
- `rpc/identity_srv/`：身份认证 RPC 服务（Kitex，端口 8891）- 用户、组织、角色、权限、菜单管理

## 常用命令

### 基础设施和服务启动（podman kube play 全栈）

> 一条命令启动**全部基础设施 + 两个 Go 服务**（共 7 个 Pod）。
> 项目仅维护这一种部署方式，无 docker-compose 或本地脚本启动选项。

```bash
# 1. 首次使用：复制示例文件，修改其中的 CHANGE_ME_* 占位密码
cp docker/pod.yml.example docker/pod.yml
vim docker/pod.yml   # 修改 Secret 中四个 CHANGE_ME_* 值（Postgres/RustFS/Redis/JWT）

# 2. 构建本地镜像（identity-srv:latest 和 gateway:latest）
./scripts/build-images.sh                  # 全部构建
./scripts/build-images.sh identity         # 只构建 identity-srv
./scripts/build-images.sh gateway          # 只构建 gateway

# 3. 启动（5 基础设施 + identity-srv + gateway，共 7 个 Pod）
podman kube play docker/pod.yml

# 4. 查看
podman pod ps          # 查看所有 Pod
podman ps              # 查看所有容器
podman volume ls       # 查看 PVC 持久卷
podman logs -f gateway-pod-gateway            # 查看 gateway 日志
podman logs -f identity-srv-pod-identity-srv  # 查看 identity_srv 日志

# 5. 修改代码后重新部署
./scripts/build-images.sh gateway
podman kube play --down docker/pod.yml
podman kube play docker/pod.yml

# 6. 销毁（保留 PVC 数据卷）
podman kube play --down docker/pod.yml

# 7. 销毁并清空数据卷
podman kube play --down --force docker/pod.yml
```

**资源拆分说明**：
- `Secret/cloudwego-secrets`：密码、密钥（Postgres/RustFS/Redis/JWT）
- `ConfigMap/cloudwego-app-env`：容器化场景必需的环境变量（服务发现地址、DB 主机等）
- 其他 .env 配置项依赖代码内 `defaults.go`，需自定义时可在对应 Deployment 的 `env:` 中追加

**敏感文件管理**：`docker/pod.yml` 已加入 `.gitignore`，仅 `pod.yml.example` 入库。修改 example 时不要写入真实密码。

**修改 Secret 后注意**：Postgres 密码只在 PVC 数据目录为空时初始化生效。修改 `POSTGRES_PASSWORD` 后，必须 `podman kube play --down --force` 清空 PVC 重建，否则会因密码不一致导致 identity_srv 启动失败。

### 测试

```bash
# 运行全部测试
go test ./... -v

# 运行单个模块的测试
cd rpc/identity_srv && go test ./... -v
cd gateway && go test ./... -v

# 运行单个测试函数
go test ./biz/logic/user -run TestUserLogic_CreateUser -v -count=1

# 测试覆盖率
go test ./... -coverprofile=coverage.out
go tool cover -func=coverage.out | grep total
```

## AI 行为约束

### 🚫 禁止自动启动服务

**规则**：AI **严禁**自动启动前端开发服务器（`npm run dev`）或任何长期运行的服务。

**原因**：
- 开发服务器会占用端口（5173、5174、5175等），多次启动会导致端口冲突
- 用户可能已经在运行其他服务的实例
- 启动服务是用户的主动操作，不应由AI自动执行

**正确做法**：
- 当需要测试前端页面时，AI应该**提示用户**手动在终端执行启动命令
- AI只负责代码生成、文件编辑、配置修改等工作
- 示例提示："前端页面已实现完成，请在终端手动运行 `cd web && npm run dev` 启动开发服务器"

**适用范围**：
- ✅ 可以执行：代码生成、编译测试、lint检查、数据库迁移等一次性任务
- ❌ 禁止执行：`npm run dev`、`podman kube play docker/pod.yml` 等长期运行的服务

### 🔒 代码一致性规范（防止 CI 测试失败）

以下规则来源于历次 CI 失败的复盘，旨在从源头杜绝"本地不报错、CI 必定失败"的问题。

#### 规则 1：Model 字段变更必须同步 DAL 层原生 SQL

修改 `models/` 中 GORM struct 的字段名或 `gorm:"column:xxx"` tag 后，**必须全局搜索旧列名**，同步更新所有 DAL 层手写 SQL 查询。

```bash
# 示例：将某列名修改后，搜索旧列名确认所有引用已更新
grep -r "old_column_name" rpc/identity_srv/
```

#### 规则 2：测试代码必须匹配函数签名

编写或修改测试时，**必须先确认被测函数的完整签名**（参数类型、返回值类型），断言中的类型必须精确匹配。

```go
// ❌ 错误：FindWithConditions 返回 *PageResult，不是 int64
users, total, err := s.repo.FindWithConditions(ctx, conditions)
assert.GreaterOrEqual(s.T(), total, int64(5))

// ✅ 正确：使用 pageResult.Total (int32)
users, pageResult, err := s.repo.FindWithConditions(ctx, conditions)
assert.GreaterOrEqual(s.T(), pageResult.Total, int32(5))
```

#### 规则 3：Test Suite Struct 必须包含所有引用字段

`SetupSuite`/`TearDownSuite` 中使用的字段必须在 struct 中显式声明，否则编译失败。**添加测试辅助字段时必须同步更新 struct 定义。**

```go
// ❌ 错误：SetupSuite 赋值了 s.sqlDB 和 s.container，但 struct 中没有定义
type MySuite struct {
    suite.Suite
    db *gorm.DB
}

// ✅ 正确：所有 Setup/TearDown 中使用的字段都有定义
type MySuite struct {
    suite.Suite
    db        *gorm.DB
    sqlDB     *sql.DB
    container testcontainers.Container
}
```

#### 规则 4：CI 配置变更必须验证后合并

修改 `.github/workflows/ci.yml` 中的以下内容时，**必须先在分支上推送并确认 CI 通过后再合并到 main**：

- 容器镜像版本（如 etcd、PostgreSQL、Redis）
- 健康检查方式（`--health-cmd`、`options`）
- 环境变量（`ETCD_ADVERTISE_CLIENT_URLS` 等）
- 测试执行命令（`go test` 参数）

**反面教材**：将 etcd 从 v3.5.21（curl 健康检查）改为 v3.5.16（etcdctl 健康检查），导致容器初始化持续失败，整个 Integration 测试 job 无法运行。

### 格式化与 Lint

```bash
# 自动格式化（gofumpt + golines + gci）
golangci-lint fmt

# 代码检查（Lint）
golangci-lint run

# 修复导入排序（独立使用）
cd rpc/identity_srv && gci write .
cd gateway && gci write .
```

### 📋 提交前检查规范（避免 CI 失败）

**问题**：CI 中 `golangci-lint run` 只检查不修复，导致 push 后 lint 失败。

**解决方案**：每次 commit 前必须执行以下步骤：

```bash
# 1. 确保本地 golangci-lint 版本与 CI 一致（v2.4.0）
golangci-lint version

# 2. 自动修复格式问题
golangci-lint fmt

# 3. 验证是否还有问题（需手动修复 unused 代码等）
golangci-lint run

# 4. 确保测试通过
go test ./... -v
```

**安装 pre-commit hook**（自动检查）：
```bash
ln -sf ../../scripts/git-hooks/pre-commit .git/hooks/pre-commit
```

**常见 CI 失败原因**：
- `gci` 导入顺序问题 → `golangci-lint fmt` 自动修复
- `wsl_v5` 空行问题 → `golangci-lint fmt` 自动修复
- `unused` 函数/变量 → 需手动删除代码
- 本地版本与 CI 不一致 → 重新安装 v2.4.0

**关键原则：本地检查 = CI 检查**：
- pre-commit hook 必须使用 `golangci-lint run`（全量检查），**禁止** `--new-from-rev=HEAD` 等增量参数
- 增量检查会导致"本地通过但 CI 失败"的盲区（如：commit A 引入错误但 hook 未拦截，后续 commit 只要不改同一文件就永远检测不到）

### 代码生成

```bash
# Kitex RPC 代码生成（修改 IDL 后必须运行）
cd rpc/identity_srv && ./script/gen_kitex_code.sh

# Hertz HTTP 代码生成（修改 IDL 后必须运行）
cd gateway && ./script/gen_hertz_code.sh

# Wire 依赖注入生成（修改 provider 后必须运行）
cd rpc/identity_srv/wire && wire
cd gateway/internal/wire && wire
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masonsxu/cloudwego-microservice-demo](https://github.com/masonsxu/cloudwego-microservice-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
