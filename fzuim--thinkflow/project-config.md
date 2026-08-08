---
trigger: always_on
description: > 本文档是项目中所有 AI Coding Agent 的行为规范入口。
---

# AGENT.md — AI Agent 团队协作规范

> 本文档是项目中所有 AI Coding Agent 的行为规范入口。
> 每个 Agent 在开始任务时都会读取并遵循本文件。
> 如果你是人类开发者，这也是一份告诉你「Agent 会怎么干活」的说明书。

---

## 一、Git 工作流

### 1.1 分支管理

- **所有 Agent 任务必须从最新的 `main` 切出新分支**，禁止在已有分支上叠加不相关的任务
- **禁止任何 Agent 直接 push 到 `main` 或 `master` 分支**
- 分支命名规范：`agent/<task-id>-<brief-description>`

```
# 正确示例
agent/PROJ-234-refresh-token-rotation
agent/PROJ-301-migrate-postgres-schema
agent/ISSUE-456-fix-login-redirect

# 错误示例
fix-bug          # 缺少 agent/ 前缀和 task-id
agent/my-work    # task-id 不可追溯
```

### 1.2 Worktree 隔离（多 Agent 并发场景）

当多个 Agent 并行工作时，每个 Agent 必须使用独立的 git worktree：

```bash
# 为每个 agent 任务创建独立 worktree
git worktree add ../agent-task-234 -b agent/PROJ-234-refresh-token
git worktree add ../agent-task-301 -b agent/PROJ-301-pg-migration

# 查看当前所有 worktree
git worktree list

# 任务完成后清理
git worktree remove ../agent-task-234
```

**规则：**
- 一个 Agent 一个 worktree，工作区不共享
- Agent 的文件操作必须限制在分配的 worktree 路径内
- 任务完成、PR 合并后，清理 worktree

---

## 二、Commit 规范

### 2.1 Commit Message 格式

每个 commit 必须遵循 Conventional Commits 格式，并包含 Agent 专属 trailer：

```
<type>(<scope>): <summary>

<正文：描述变更的背景与动机>

Agent-Task: <原始任务描述或任务ID>
Agent-Model: <使用的模型>
Agent-Decision: <关键设计决策及理由>
Agent-Limitation: <已知局限或后续TODO>
```

**示例：**

```
feat(auth): implement JWT refresh token rotation

Add sliding-window refresh token support to reduce re-login friction
while maintaining session security.

Agent-Task: PROJ-234 - Add refresh token support to auth service
Agent-Model: claude-opus-4-6
Agent-Decision: Used 7-day sliding window over fixed expiry for better UX;
refresh tokens stored in httpOnly cookie to prevent XSS access
Agent-Limitation: Redis TTL not yet aligned with token expiry on logout
```

**查询 Agent 提交历史：**

```bash
# 列出所有包含 Agent-Task trailer 的提交
git log --format='%(trailers:key=Agent-Task,valueonly)'

# 按 trailer 过滤
git log --grep="^Agent-Task:" --all
```

### 2.2 Atomic Commit 原则

**每个 commit 只表达一个可解释、可回滚、可验证的语义变化。**

- 一个 commit = 一个逻辑变更
- 每个 commit 节点上代码必须可编译、测试可通过
- **不要把重构和功能修改混在同一个 commit**
- **不要把多个不相关模块的修改混在同一个 commit**

```
# 好的切分：每个 commit 对应一个独立关注点
feat(auth): add RefreshToken domain model and repository interface
feat(auth): implement JWT refresh token issuance in AuthService
feat(auth): expose POST /auth/refresh endpoint
test(auth): add unit tests for refresh token rotation logic

# 反例：所有改动压成一个 commit
feat(auth): implement refresh token  # 3000行 diff，无法审查
```

### 2.3 Checkpoint Commit 策略

对于预计耗时超过 15 分钟的任务，在以下关键节点进行 checkpoint commit：

1. 完成数据模型/接口定义
2. 完成核心逻辑实现
3. 完成测试编写
4. 完成文档更新

Checkpoint commit 的 message 以 `[WIP]` 开头：

```
[WIP] feat(auth): draft refresh token domain model
```

**任务完成后、开 PR 前，使用 interactive rebase 整理历史：**

```bash
git log --oneline main..HEAD                    # 查看当前分支提交
git rebase -i main                               # 交互式整理
git log --oneline main..HEAD                    # 确认最终结果
```

整理策略：
- 将 `[WIP]` checkpoint commit squash 为有意义的语义 commit
- 确保最终历史中每个 commit 都能独立理解和回滚
- 每个保留的 commit 需要包含 Agent-Task、Agent-Decision trailer
- **不要对已经推送到远程的分支做 force push（除非已确认远程分支无他人使用）**

---

## 三、PR 流程

### 3.1 开 PR

- 所有 Agent 发起的 PR 必须使用项目规定的 Agent PR 模板（`.github/pull_request_template/agent.md`）
- 确保所有 CI 检查通过后再请求 review
- **Agent 不得自行 merge 自己的 PR**，merge 动作由人工触发

### 3.2 PR 模板必填项

```
## Task Description     — 原始任务描述
## What Changed         — 核心变更摘要
## Key Design Decisions — 关键设计决策及理由
## Alternatives Considered — 考虑过但未采用的方案
## Test Coverage        — 测试覆盖情况
## Known Limitations    — 已知局限和后续TODO
## Review Guidance      — 建议 reviewer 重点关注的部分
```

---

## 四、禁止提交的内容

以下内容**绝对禁止**出现在任何 commit 中：

- API keys、tokens、passwords（必须使用环境变量）
- 构建产物（`dist/`、`build/`、`.next/`）
- 依赖目录（`node_modules/`、`__pycache__/`、`.venv/`）
- 本地配置文件（`.env`、`.env.local`、`*.local`）
- 大二进制文件（>1MB，应使用 Git LFS）
- 临时调试代码、注释掉的测试用例

---

## 五、多 Agent 并发协作规则

### 5.1 隔离原则

- 每个 Agent 使用独立的 git worktree
- 每个 Agent 在独立的分支上工作
- 避免多个 Agent 同时修改同一个公共模块（如共享类型定义、配置文件）

### 5.2 冲突处理

- 如果 Agent 需要修改的公共模块已被其他 Agent 占用，先在分支上记录依赖，等前序 Agent 合并后再 rebase
- 合并前检查语义正确性，不能仅依赖 Git 的文本级无冲突合并
- 公共接口变更必须同步更新所有消费方

### 5.3 Monorepo 特别注意

- 只运行受当前变更影响的 package 的测试：

```bash
# Nx
nx affected --target=test

# Turborepo
turbo run test --filter='[HEAD^1]'
```

- Atomic commit 的边界：一个 commit 可以跨多个 package，只要修改在逻辑上不可分割
- 修改共享 library 的同时必须同步更新消费方

---

## 六、CI 命令速查

```bash
# 运行受影响的测试（推荐 Agent 本地使用）
nx affected --target=test
turbo run test --filter='[HEAD^1]'

# 全量检查（由 CI 在 PR 合并前执行，不推荐 Agent 本地运行）
# npm run test:all
```

---

## 七、可追溯性链路

```
任务系统（Jira/Linear/TAPD）
    ↓ task-id
Git Branch / PR
    ↓ Agent-Task trailer in commit message
Agent Session Log（可选，存储在 .agent-logs/ 目录，已加入 .gitignore）
    ↓ 完整的 prompt 和 agent reasoning
代码变更
```

---

## 八、快速自检清单

Agent 在提交代码前，必须自查以下项目：

- [ ] 分支名符合 `agent/<task-id>-<description>` 格式
- [ ] 没有直接 push 到 main/master
- [ ] commit message 符合 Conventional Commits 格式
- [ ] 每个 commit 包含 Agent-Task、Agent-Decision trailer
- [ ] 每个 commit 是 atomic 的（一个逻辑变更、可编译、可测试）
- [ ] 没有提交 API key、.env、node_modules 等敏感/生成文件
- [ ] PR description 已按模板完整填写
- [ ] CI 检查已通过
- [ ] 长任务 (>15min) 已做 checkpoint commit 并在最终 rebase 整理
- [ ] 未自行 merge 自己的 PR

---

## 九、项目概览 — ThinkFlow

> 以下内容是项目本身的框架信息，供 AI Agent 了解项目上下文。

### 9.1 项目简介

**ThinkFlow**（中文名：思行）是一款 AI 增强的个人任务管理与灵感工具桌面应用。通过集成 LLM（大语言模型），将任务管理、AI 对话、专注计时、灵感生成、记忆回溯等功能融合在一个统一的界面中。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fzuim/ThinkFlow](https://github.com/Fzuim/ThinkFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
