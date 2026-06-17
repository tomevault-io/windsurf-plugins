---
trigger: always_on
description: 本文档包含了 AI Agent 使用 Leros 代码库的重要信息。
---

# Leros Agent 开发指南

本文档包含了 AI Agent 使用 Leros 代码库的重要信息。

## 构建/检查/测试 命令

### 构建命令

- `go build -o ./bundles/leros ./backend/cmd/leros/` - 构建主 Leros 后端服务（输出到 `./bundles/`）
- `make docker-build` - 构建 Docker 镜像（标签：registry.yygu.cn/insmtx/Leros:latest）
- `make docker-run` - 在本地运行 Docker 镜像
- `make run` - 以前台模式启动 docker-compose 服务
- `make run-detached` - 以分离模式（后台）启动 docker-compose 服务
- `make stop` - 停止 docker-compose 服务
- `make logs` - 查看 docker-compose 服务日志
- `make swagger` - 生成 Swagger API 文档（输出到 `docs/swagger/docs.go`）

### 测试命令

- `go test ./...` - 运行项目中所有测试
- `go test -v ./...` - 以详细输出方式运行所有测试
- `go test ./backend/path/to/package` - 运行特定包的测试
- `go test -run ^TestFunctionName$ ./backend/path` - 运行特定测试函数
- `go test -race ./...` - 运行所有检测竞态条件的测试
- `go test -cover ./...` - 运行测试并显示覆盖率信息

### 检查命令

- `go fmt ./...` - 格式化所有 Go 代码
- `go vet ./...` - 检查所有 Go 代码中的常见错误
- `golint ./...` - 检查所有 Go 代码（通过 `go install golang.org/x/lint/golint@latest` 安装）
- `gofmt -s -w .` - 简化代码并写入更改（按照现有 Makefile）
- `staticcheck ./...` - 全面的 Go 静态分析（如已安装）

## 代码风格指南

### 导入组织

- 在标准库、第三方和项目特定包之间用空行分组导入
- 仅在防止命名冲突时使用语义导入别名
- 组织成三组：stdlib，第三方，内部包

```
import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/spf13/cobra"

	"github.com/insmtx/Leros/backend/config"
)
```

### 格式约定

- 使用制表符进行缩进，而不是空格（从现有 Go 文件验证过）
- 提交前执行 `go fmt ./...`
- 尽可能保持每行少于 120 个字符
- 使用 `gofmt -s` 简化代码

### 命名约定

- 对于导出的函数/类型使用驼峰命名法（`GetUser`，`UserService`）
- 对于未导出/内部函数/类型使用小写驼峰命名法（`getUser`，`userService`）
- 使用清晰、描述性的名称；优先考虑清晰度而不是简洁性
- 在包之间对类似概念使用一致的名称
- 与系统相关的变量应引用 Leros 概念

### 类型和接口

- 在第一次使用附近定义接口
- 保持接口小，通常是一个或几个方法
- 在适用时，用 "-er" 后缀命名接口类型（例如，`Runner`，`Handler`）
- 在不需要接口时，在函数签名中明确使用具体类型
- 当传递给函数且会被修改时，倾向于返回结构体的指针

### 错误处理

- 显式处理错误；不要忽略它们
- 在适当的情况下使用具体的错误类型并包装错误
- 遵循以下模式："if err != nil { return err }"
- 对简单的静态字符串使用 `errors.New()`
- 使用 `fmt.Errorf()` 和 `%w` 动词来包装带有更多上下文的错误
- 适当时记录错误上下文

### 附加准则

- 所有公共函数必须有 GoDoc 注释
- 注释应采用英文，并解释原因而非内容
- 在整个应用程序中维护一致的日志格式
- 适当地使用 context.Context 进行取消和请求作用域值
- 遵循依赖注入模式而不是全局变量
- 使用 Cobra 进行命令行界面实现，如 main.go 文件所示

### 强制约束

- **禁止使用 `panic`**：整个项目（含库代码和业务代码）严禁使用 `panic`。错误必须通过返回 `error` 逐层传递，由顶层调用方统一处理。对于不可恢复的致命错误（如配置缺失导致无法启动），应在 `main` 函数中使用 `log.Fatal` 退出。
- **禁止使用 `map[string]interface{}` 传递数据**：严禁在函数签名、接口定义或跨层通信中使用 `map[string]interface{}` 传递业务数据。必须定义具名结构体（struct）或类型化 map（如 `map[string]string`），以保证编译时类型安全和代码可读性。若现有接口（如 `Skill` 接口）使用了 `map[string]interface{}`，需在后续迭代中重构为强类型参数。

### 提交准则

- 遵循约定式提交格式：<type>(<scope>): <subject>
- 在 Leros 项目中使用中文作为提交消息
- 类型选项包括：
  - `feat`：新功能
  - `fix`：修复错误
  - `docs`：文档更新
  - `style`：代码风格调整
  - `refactor`：重构代码
  - `test`：测试相关
  - `chore`：构建工具或辅助工具变更
- 适当时，在正文部分包含技术实现和业务逻辑的详细描述

## 分层边界

项目遵循三层架构，每层有明确的职责边界。写代码前先确认改动属于哪一层。

| 层级         | 路径                                | 允许                                                                                | 禁止                                                                       |
| ------------ | ----------------------------------- | ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **进程入口** | `backend/cmd/leros/`                | cobra 命令注册、进程生命周期（`lifecycle.Std().WaitExit()`）、信号处理、`log.Fatal` | 业务逻辑                                                                   |
| **库代码**   | `backend/internal/*`                | 业务逻辑实现，通过 `error` 向上传递失败                                             | `os.Exit()`、`lifecycle.Std()`、`log.Fatal`、`panic`、信号处理、cobra 依赖 |
| **共享类型** | `backend/types/`、`backend/config/` | 领域类型、配置结构定义                                                              | 任何业务逻辑、外部依赖                                                     |

核心原则：

- `internal/` 下的包不知道自己是运行在 server、worker 还是 CLI 中。进程如何启停是 `cmd/` 的事。
- 目录名不是职责的借口——`internal/cli` 表示"CLI 相关的库代码"，不代表它可以接管进程生命周期。
- 多层代码共享的常量/类型应下沉到最底层共享包，避免在两个包中重复定义。若重复已存在，优先合并到更内层的包，外层通过类型别名引用。

## 新增功能操作流程

实现任何新功能时，严格按以下顺序。跳过第 1 步直接写代码是本项目最常见的返工原因。

1. **搜索已有参照** — 项目内大概率已有类似实现。在动手前先回答"这个模式项目里哪里用过"：
   - 新增 cobra 命令 → `backend/cmd/leros/` 下已有命令
   - 新增 HTTP API → `backend/internal/api/handler/` 下已有 handler
   - 新增 HTTP 客户端调用 → 搜 `http.Client` 或 `http.NewRequest`
   - 新增事件发布/订阅 → 搜 `eventbus.Publish` 或 NATS 相关用法
   - 新增数据库操作 → `backend/internal/infra/db/` 下已有方法
   - 跨包共享的常量/类型 → 先检查 `events/`、`types/` 等共享包是否已定义

2. **复用结构** — 抄已有代码的骨架：import 组织方式、函数签名风格、错误处理模式。保留被广泛验证过的结构，只替换业务内容。

3. **填充逻辑** — 在复用来的骨架上写自己的代码。

4. **不要跳步** — 哪怕功能看起来很简单，也先 grep 验证。文件名和函数名可能产生误导（例如 `internal/cli` 看上去像 CLI 入口但实际上不是）。

## 项目结构

- `/backend` - 主要 Go 应用程序代码
  - `/backend/cmd/leros` - 主 Leros 后端服务入口点
  - `/backend/config` - 配置加载和类型
  - `/backend/engines` - 引擎层（native engine 与 system prompt 分层架构）
  - `/backend/gateway` - HTTP 网关包
  - `/backend/interaction` - 事件驱动交互层
    - `/backend/interaction/connectors` - 渠道连接器（GitHub 已实现；GitLab，WeWork 桩代码）
    - `/backend/internal/infra/mq` - NATS JetStream 事件总线实现
    - `/backend/interaction/gateway` - 事件网关设置
  - `/backend/internal/worker` - Worker 任务消费与调度
  - `/backend/pkg` - 提取的公共库
  - `/backend/prompts` - 系统提示词管理
  - `/backend/skills` - Skill 接口、类型和示例（三层架构 + 事件驱动 handler 模型）
  - `/backend/tools` - 工具模块
  - `/backend/types` - 核心领域类型（DigitalAssistant，Event 等）
  - `/backend/tests` - 测试代码
- `/bundles` - 构建输出目录（生成；已忽略 git）
- `/deployments/build/Dockerfile` - 容器构建配置

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [insmtx/Leros](https://github.com/insmtx/Leros) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
