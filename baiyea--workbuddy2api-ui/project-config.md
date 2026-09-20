---
trigger: always_on
description: 本文面向在本仓库工作的 AI 编程助手，维护架构、开发、验证、上游更新与镜像发布约定。`README.md` 面向使用者，主打 **将 WorkBuddy / CodeBuddy 反向代理为 OpenAI 兼容 API**；Web 控制台是配套能力，不要把 README 写成开发手册。
---

# AI 开发指南

本文面向在本仓库工作的 AI 编程助手，维护架构、开发、验证、上游更新与镜像发布约定。`README.md` 面向使用者，主打 **将 WorkBuddy / CodeBuddy 反向代理为 OpenAI 兼容 API**；Web 控制台是配套能力，不要把 README 写成开发手册。

## 优先遵守

- 当前处于开发阶段，以当前设计为准，不为历史版本新增兼容层、迁移分支或双轨配置。
- 不直接修改 `upstream/`。新增 core 能力放 `extensions/`，修改上游既有文件通过 `patches/`，控制台直接改 `console/`。
- 生产部署保持 `docker compose up -d` 一条命令启动，不新增宿主启动脚本、独立 init 服务或服务器端构建依赖。
- 生产 Compose 的 `environment` 默认只有 `TZ` 和注释形式的可选密钥；固定内部参数内置在镜像。不重新引入 `.env` 插值或必填密钥。
- 保留相对目录挂载、源码构建入口和 `linux/amd64` 发布架构，不新增 Docker 健康检查。
- 修改前检查 `git status --short`，保留用户已有改动；不能为通过验证而重置工作区、清空数据或覆盖真实配置。
- 更新上游、发布镜像、推送 Git、重启实际服务分别需要对应授权。发布镜像不等于部署授权。

## 架构

```text
浏览器 / OpenAI 或 Anthropic 文本客户端
            │ :7863
            ▼
console（独立 Go 服务，内嵌 HTML/CSS/JavaScript）
  ├─ /admin/*：管理会话、CSRF、网页操作
  │                 └─ 桥接密钥 → core /internal/v1/*
  └─ /v1/*、/status、/healthz：公共接口代理 → core
                                                    │
                                  同一个账号池、上游客户端、调度器
                                                    │
                                          WorkBuddy / CodeBuddy
```

- **core**：固定上游加扩展和补丁，负责账号池、公共 API、OAuth、任务执行及持久化。不得为控制台另建账号池或调度器。
- **console**：负责网页、管理认证和代理；不加载账号凭据，不直接执行签到等业务。
- 仅 console 发布宿主端口。core 只在 Compose 网络内访问；console 仅只读挂载密钥目录，不挂载账号和状态目录。
- 当前 README 重点介绍 `/v1/models`、`/v1/chat/completions` 与流式调用；不得把“OpenAI 兼容”宣传为完整覆盖所有 OpenAI API 或客户端功能。
- `/v1/messages` 由 core 的 Anthropic 文本适配器在进程内复用原 OpenAI Handler；不另建账号池、HTTP 回环或协议代理服务。只在启用 core 桥接的分支包装，未启用桥接的源码模式保持原 Handler。

## 目录与代码入口

| 路径 | 职责 |
| --- | --- |
| `upstream/`、`upstream.lock` | 上游普通源码快照；锁文件记录来源、commit 和源码摘要。 |
| `extensions/cmd/server/extension.go` | core 初始化、密钥、桥接和任务生命周期接线。 |
| `extensions/internal/bridge/` | 内部管理接口，复用公共能力。 |
| `extensions/internal/anthropic/` | Messages 文本请求、普通响应与增量 SSE 适配，传递取消与真实用量。 |
| `extensions/internal/oauth/`、`extensions/internal/pool/` | 授权流程、账号热加载等扩展。 |
| `extensions/internal/scheduler/`、`extensions/internal/taskrun/` | 任务目录、执行观察、单运行器和持久历史。 |
| `extensions/scripts/` | Python 任务结果事件与测试。 |
| `patches/series`、`patches/README.md` | 补丁顺序、修改原因、验证方式和移除条件。 |
| `console/main.go`、`console/server.go`、`console/proxy.go` | 配置、管理会话、路由与代理。 |
| `console/web/`、`console/web_test.cjs` | 无前端框架的页面资源及 Node 测试；通过 Go embed 打包。 |
| `deploy/` | Dockerfile、core 镜像启动入口、配置和隔离验收工具。 |
| `scripts/overlay.py` | 源码物化、补丁标识和上游更新。 |
| `scripts/check.sh`、`scripts/acceptance.sh` | 本地检查和真实容器隔离验收。 |
| `scripts/release.sh`、`scripts/release.py` | 时间戳镜像发布入口与实现。 |
| `docs/superpowers/verification/` | 验证记录、界面截图；模拟素材不代表真实上游结果。 |

## 上游与补丁机制

唯一上游为 `https://github.com/Sliverkiss/workbuddy2api`。实际版本以 `upstream.lock` 为准，不在多份文档中重复写死 commit。

`scripts/overlay.py prepare` 的步骤：校验源码摘要 → 复制快照到新目录 → 复制扩展文件 → 按 `patches/series` 执行 `git apply --check` 并应用补丁。不会改写 `upstream/`。

1. 新文件和测试放到 `extensions/` 的对应相对路径；不能通过扩展覆盖已有上游文件。
2. 需要改既有上游文件时，在物化目录验证后维护补丁，同时更新补丁说明和必要测试。
3. `.build/` 是临时产物。不能只改物化代码而不回写扩展或补丁。
4. `console/` 是独立 Go 模块，不向上游公共 Handler 注入网页管理实现。
5. 上游提供等价行为且相关回归测试无需补丁也通过时，才移除对应补丁。

### 更新上游

仅在明确授权后执行；构建相关目录须无未提交改动，显式指定 commit 或 tag：

```bash
python3 scripts/overlay.py update --ref COMMIT_OR_TAG
git diff -- upstream upstream.lock
git diff -- patches extensions deploy console scripts
```

脚本在候选目录验证补丁、扩展、测试及隔离容器验收；成功才更新快照和锁文件。失败保留诊断候选，不自动提交、推送、发布或重启服务。不要绕过摘要检查、强行应用失败补丁或手改锁文件掩盖不一致。

## 开发与测试

开发机需要 Git、Bash、Python 3、Go、Node.js、curl、Docker Compose 和 Buildx。Go 最低版本以各模块 `go.mod` 为准，镜像使用 Go 1.23；竞态测试需要支持 CGO 的本机编译环境。部署成品镜像的服务器不需要这些开发工具。

以下命令从仓库根目录执行。根目录不是 Go 模块，不要直接在根目录运行 `go test ./...`。

```bash
python3 -m unittest discover -s scripts -p 'test_*.py' -v
python3 -m unittest discover -s deploy -p 'test_*.py' -v
bash scripts/check.sh
docker compose --env-file /dev/null -f docker-compose.yml config --quiet
git diff --check
```

`check.sh` 在新临时目录物化 core，运行 Go 测试、vet、关键包竞态测试、console 竞态测试、Node 页面测试和 Python 任务测试。

定点调试时使用尚不存在的输出路径：

```bash
python3 scripts/overlay.py prepare --output .build/core-review
go -C .build/core-review test ./internal/bridge ./internal/taskrun
go -C console test -race ./...
node --test console/web_test.cjs
```

修改后重新物化，旧物化目录不会自动同步；重复运行使用新的输出路径，不删除不明来源的目录。

涉及镜像、Compose、启动参数、密钥或持久化时，还需运行 `bash scripts/acceptance.sh`。它在隔离项目、端口和存储中验证直接启动、YAML 密钥覆盖与取消覆盖、日志、时区、数据保留以及 mock API/任务流程。禁止将测试指向真实凭据或数据；模拟验收不证明真实上游授权成功或奖励到账。

### Anthropic 文本验证

公共 `/v1/messages` 使用同一个模型 API Key（`x-api-key`）和固定 `anthropic-version: 2023-06-01`。`New(next, apiKey, maxBodyBytes)` 仅截获该路径，其他 OpenAI 路径不变；未知用量保持 `null`，不得伪造零或宣称完整 Claude Code 兼容。

网页 `POST /admin/messages` 沿用管理会话、同源、CSRF 和退出取消，代理至带 owner 的 `/internal/v1/messages`。请求 envelope 为 `{conversation_id, request}`；ID 限 `[A-Za-z0-9_-]{1,128}`，request 为完整 Anthropic 文本请求，不加私有字段。bridge 有界读取并验证 envelope，通过 `WithConversation` 注入可信上下文，由 adapter 写入 OpenAI `conversationId`；不接受公共私有头伪造会话。体积限制沿用 `server.max_body_mb`，包含管理 envelope。

SDK 仅用于隔离测试，不进入镜像或生产依赖。使用 Python 3.12 临时虚拟环境安装 `anthropic==0.67.0 httpx==0.28.1` 后执行：

```bash
# 内存契约：未知/晚到用量、raw events、text_stream、最终消息聚合
/path/to/venv/bin/python scripts/check_anthropic_sdk.py
# 实际访问验收脚本创建的隔离 mock 网关；执行后沿原机制清理
WB2A_SDK_PYTHON=/path/to/venv/bin/python bash scripts/acceptance.sh
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baiyea/workbuddy2api-ui](https://github.com/baiyea/workbuddy2api-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
