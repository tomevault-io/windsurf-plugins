---
trigger: always_on
description: > 本文件是 AI Agent 和开发者的协作指南，记录了项目的核心约定、架构决策和开发原则。
---

# AGENTS.md - OpenAgent GitHub Bridge

> 本文件是 AI Agent 和开发者的协作指南，记录了项目的核心约定、架构决策和开发原则。

---

## 项目概述

**OpenAgent GitHub Bridge** 是一个 Go 后端服务，作为 GitHub Webhooks 和 AI Agent（如 OpenCode）之间的桥梁。它监听 GitHub 事件，根据配置触发 AI Agent 执行任务（如自动修复 Issue、Review PR）。

### 核心设计理念

```
GitHub Webhook → Bridge (验证/解析/下发) → AI Agent (执行/创建PR)
                      ↓
              Fire-and-Forget
              (下发即返回，不等待结果)
```

---

## 核心约定

### 1. Fire-and-Forget 模式

**Bridge 只负责下发任务，不等待 Agent 响应。**

- Webhook 收到请求 → 验证签名 → 解析事件 → 放入队列 → **立即返回 202**
- Worker 从队列取出 → 构建 Prompt → 调用 Agent API → **立即返回**（不等待执行结果）
- Agent 自己负责：执行任务、与 GitHub 交互、创建 PR、发评论

```go
// 正确 ✓ - Fire-and-forget
func (a *OpenCodeAdapter) DispatchTask(ctx context.Context, req *TaskRequest) (*TaskResponse, error) {
    // 调用 /session/:id/prompt_async，返回 204 No Content
    // 不等待 Agent 执行完成
}

// 错误 ✗ - 同步等待
func (a *OpenCodeAdapter) DispatchTask(ctx context.Context, req *TaskRequest) (*TaskResponse, error) {
    // 等待 Agent 返回完整响应
    // 这会阻塞 Worker
}
```

### 2. 配置开关原则

**只有实际触发动作的事件才需要 enable 配置。**

```yaml
# 正确 ✓ - 有实际动作的事件
features:
  ai_fix:
    enabled: true  # Issue 被打上标签 → 触发自动修复
  pr_review:
    enabled: false              # PR 创建 → 触发自动 review
    label_trigger_enabled: true # PR 被打标签 → 触发 review

# 错误 ✗ - 没有实际动作的事件不需要配置
features:
  issue_opened:
    enabled: true  # 如果没有对应动作，不要添加这个配置
```

**新增功能时的检查清单：**
- [ ] 这个事件监听后会触发什么动作？
- [ ] 如果有动作，添加对应的 `enabled` 配置
- [ ] 如果只是数据收集/日志，不需要配置

### 3. Git Workspace 隔离

**每个 Issue/PR 的工作 Session 应该在独立的 Git Workspace 中。**

- 下发任务前，Bridge 会指示 Agent 创建独立 workspace
- Workspace 分支命名：`issue-{number}` 或 `pr-{number}`
- 不同 Session 不会相互干扰

```go
// 创建 workspace 的流程
1. Bridge 调用 agent 侧 `workspace-manager` companion service
2. companion service 以独立 clone 方式创建或复用 workspace，并返回 `worktreePath`
3. Bridge 创建正式 OpenCode session，绑定到该目录
4. Bridge 下发任务到该 workspace session
```

### 4. Session 管理与复用

**以 `{owner}/{repo}/{type}/{number}` 为唯一 Key 管理 Session，同一 Issue/PR 复用 Agent Session。**

```go
// Session Key 格式
type SessionKey struct {
    Owner  string  // e.g., "openagent"
    Repo   string  // e.g., "github-bridge"
    Type   string  // "issue" or "pr"
    Number int     // e.g., 123
}

// 同一 Issue/PR 的所有交互复用同一 Session
key := SessionKey{Owner: "openagent", Repo: "bridge", Type: "issue", Number: 42}
// Key: "openagent/bridge/issue/42"

// Session 复用流程
1. 首次交互：创建新 Session，创建 Workspace，保存 Agent Session ID
2. 后续交互：复用已有 Agent Session ID，不重新创建 Workspace
3. Session 过期后（默认 24h）：重新创建
```

### 5. API 参考注释

**所有外部 API 调用必须在注释中标明参考来源。**

```go
// 正确 ✓
// VerifySignature validates the webhook payload signature using HMAC-SHA256.
// Reference: https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries
func VerifySignature(payload []byte, signature, secret string) bool {

// IssueEvent represents an issue event payload.
// Reference: https://docs.github.com/en/webhooks/webhook-events-and-payloads#issues
type IssueEvent struct {

// DispatchTask sends a task to OpenCode server.
// Reference: https://open-code.ai/docs/en/server#post-session-id-prompt_async
func (a *OpenCodeAdapter) DispatchTask(ctx context.Context, req *TaskRequest) error {
```

**参考文档来源：**
- GitHub Webhooks: https://docs.github.com/en/webhooks
- GitHub API: https://docs.github.com/en/rest
- OpenCode Server: https://open-code.ai/docs/en/server
- OpenCode SDK: https://opencode.ai/docs/sdk

### 6. 多仓库支持

**OpenCode 必须在仓库目录中启动，每个仓库需要独立的 OpenCode 实例。**

```yaml
# 单仓库模式 - 只配置默认 OpenCode
opencode:
  host: "http://localhost:4096"

# 多仓库模式 - 为每个仓库配置独立的 OpenCode
repositories:
  "owner1/repo1":
    opencode_host: "http://localhost:4096"
    workspace_manager_host: "http://localhost:4081"
  "owner2/repo2":
    opencode_host: "http://localhost:4097"
    workspace_manager_host: "http://localhost:4082"
```

**启动 OpenCode 的正确方式：**
```bash
# 必须在仓库目录中启动
cd ~/repos/owner1/repo1 && opencode server --port 4096
cd ~/repos/owner2/repo2 && opencode server --port 4097
```

**路由逻辑：**
```go
// MultiRepoOpenCodeAdapter 根据 repo 路由到对应的 OpenCode 实例
func (m *MultiRepoOpenCodeAdapter) DispatchTask(ctx context.Context, task TaskContext) {
    adapter := m.getOrCreateAdapter(task.RepoOwner, task.RepoName)
    return adapter.DispatchTask(ctx, task)
}
```

### 7. 鉴权配置

**支持可配置鉴权，未配置时作为无鉴权处理。**

```go
// OpenCode HTTP Basic Auth
type OpenCodeConfig struct {
    Host     string // Server URL
    Username string // Default: "opencode"
    Password string // Empty = no auth
}

// 鉴权逻辑
func (a *OpenCodeAdapter) setAuth(req *http.Request) {
    if a.config.Password != "" {
        req.SetBasicAuth(a.config.Username, a.config.Password)
    }
    // Password 为空时，不设置 Auth header
}
```

---

## 目录结构

```
openagent-github-bridge/
├── cmd/server/main.go           # 应用入口
├── internal/
│   ├── config/config.go         # 配置管理（含多仓库映射）
│   ├── agent/
│   │   ├── interface.go         # Agent 接口定义
│   │   ├── opencode.go          # OpenCode 适配器（单实例）
│   │   └── multi_repo.go        # 多仓库路由适配器
│   ├── github/
│   │   ├── client.go            # GitHub API 客户端
│   │   └── webhook.go           # Webhook 签名验证
│   ├── handler/webhook.go       # HTTP 处理器
│   ├── queue/task.go            # 异步任务队列
│   ├── session/manager.go       # Session 管理器
│   └── service/bridge.go        # 核心业务逻辑
├── config/config.yaml           # 配置文件

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spongehah/Openagent-Github-Bridge](https://github.com/spongehah/Openagent-Github-Bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
