---
trigger: always_on
description: This is a TypeScript ESM CLI/TUI project. Main source lives in `src/`.
---

# Repository Guidelines

## Project Structure & Module Organization

This is a TypeScript ESM CLI/TUI project. Main source lives in `src/`.

- `src/main.ts`, `src/index.ts`: CLI entry and harness setup.
- `src/tui/`: interactive terminal UI, commands, components, state.
- `src/agents/`: prompts, model routing, dynamic tools, workspace setup, subagents.
- `src/tools/`: model-facing tools; tests live in `src/tools/__tests__/`.
- `src/security/`: pentest context, findings, reports, and shared helpers.
- `src/skills/`: built-in skills and workflows. Skill folders must contain `SKILL.md`.
- `src/auth/`, `src/mcp/`, `src/hooks/`, `src/lsp/`: provider auth, MCP, hooks, and language server support.

Project docs include `README.md`, `README.en.md`, `CONTRIBUTING.md`, `DEVELOPMENT.md`, and `SECURITY.md`.

## Build, Test, and Development Commands

- `pnpm install`: install dependencies.
- `pnpm cli`: run the CLI from source with `tsx src/main.ts`.
- `pnpm check`: run TypeScript type checking.
- `pnpm test`: run Vitest in watch mode.
- `pnpm test:run`: run the full Vitest suite once.
- `pnpm vitest run <path>`: run focused tests, for example `pnpm vitest run src/tools/__tests__/jwt-analyze.test.ts`.
- `pnpm lint`: run ESLint.
- `pnpm build`: build with `tsup` and copy built-in skills.
- `pnpm pack:check`: dry-run npm packaging.

## Coding Style & Naming Conventions

Use TypeScript, ESM imports, and existing local patterns. Keep modules focused and avoid unrelated refactors. Tool files use kebab-case filenames such as `jwt-analyze.ts`, while tool IDs use snake_case such as `jwt_analyze`. Define tool input schemas with Zod near the tool. Prefer ASCII unless editing a file that already uses non-ASCII content.

## Testing Guidelines

Vitest is the test framework. Place tests beside the owning area in `__tests__` directories and name files `*.test.ts`. Run the narrowest relevant test during development, then broaden with `pnpm check` and `pnpm test:run` before a PR. For tools, update `src/agents/__tests__/tools.test.ts` when registration or mode exposure changes.

## Commit & Pull Request Guidelines

Recent history uses short imperative subjects, sometimes Conventional Commit prefixes: `fix: detect prerelease npm updates`, `chore: publish under mingyilab npm scope`, `feat: add pentest mode and rebrand cli`. Keep commits focused.

PRs should include what changed, why, tests run, and security implications. Update docs for user-facing behavior, public APIs, workflow changes, or contributor process changes.

## AI Coding 变更说明规范

本规范用于 AI 提交代码、Code Review 修复、MR/PR 描述和自动化审查记录。目标是把 AI 生成的自然语言变更描述沉淀为可审查、可追踪、可自动处理的标准记录。

### 适用范围

- AI 直接生成或修改代码后的提交说明。
- AI 根据 Code Review 意见进行修复后的回复记录。
- MR/PR 描述中的变更摘要、影响面和验证结果。
- 自动化审查工具输出的结构化审查记录。

### 核心原则

1. 可审查：每条记录必须说明变更意图、影响范围、风险和验证方式。
2. 可追踪：每条记录必须关联需求、缺陷、审查意见、任务或上下文来源。
3. 可处理：字段名称、枚举值和列表结构应稳定，便于脚本、CI、审查机器人解析。
4. 可复核：AI 不能把未执行的验证写成已通过；不确定结论必须显式标记。
5. 最小充分：描述围绕本次变更，不混入无关重构、泛化说明或无法验证的价值判断。

### 记录格式

标准记录由 YAML 元信息块和 Markdown 说明块组成。YAML 用于机器解析，Markdown 用于人工审查。

```markdown
---
record_type: change
source: ai
change_id: AI-CHANGE-YYYYMMDD-001
related_refs:
  - type: issue
    id: CINSIGHT-123
title: 修复状态监测任务重试计数异常
summary: 修复 NATS 状态监测任务在重试后重复累计失败次数的问题。
scope:
  modules:
    - internal/worker/status
  files:
    - internal/worker/status/processor.go
change_type:
  - bugfix
risk_level: medium
compatibility: backward-compatible
data_impact: none
security_impact: none
tests:
  - command: go test ./internal/worker/status
    result: passed
review_status: ready
created_by: ai
created_at: 2026-06-08T10:00:00+08:00
---

## 变更背景

...

## 变更内容

...

## 影响范围

...

## 验证结果

...

## 风险与回滚

...
```

### 字段定义

| 字段 | 必填 | 类型 | 说明 |
| --- | --- | --- | --- |
| `record_type` | 是 | enum | 记录类型，见枚举定义。 |
| `source` | 是 | enum | 记录来源，AI 生成固定为 `ai`。 |
| `change_id` | 是 | string | 本次记录唯一标识。提交前未知可用 `pending`，合并前必须替换。 |
| `related_refs` | 是 | list | 关联需求、缺陷、审查意见、PR、提交或外部上下文。无明确来源时写 `type: context`。 |
| `title` | 是 | string | 一句话说明本次变更，不超过 80 个中文字符。 |
| `summary` | 是 | string | 概述变更目的和结果，不超过 200 个中文字符。 |
| `scope.modules` | 是 | list | 受影响模块或目录。 |
| `scope.files` | 是 | list | 关键文件列表，不要求列出生成文件和锁文件。 |
| `change_type` | 是 | list | 变更类型，见枚举定义。 |
| `risk_level` | 是 | enum | 风险等级，见枚举定义。 |
| `compatibility` | 是 | enum | 兼容性影响。 |
| `data_impact` | 是 | enum | 数据影响。 |
| `security_impact` | 是 | enum | 安全影响。 |
| `tests` | 是 | list | 已执行、未执行或不可执行的验证记录。 |
| `review_status` | 是 | enum | 当前审查状态。 |
| `created_by` | 是 | string | 记录创建者，AI 生成固定为 `ai` 或具体 AI 工具名。 |
| `created_at` | 是 | datetime | ISO 8601 时间，包含时区。 |

### 枚举定义

`record_type`：

- `change`：普通代码变更记录。
- `review-fix`：Code Review 修复记录。
- `merge-request`：MR/PR 描述记录。
- `auto-review`：自动化审查记录。

`change_type`：

- `feature`：新增能力。
- `bugfix`：缺陷修复。
- `refactor`：不改变外部行为的重构。
- `test`：测试补充或调整。
- `docs`：文档变更。
- `chore`：构建、脚本、配置等工程事务。
- `security`：安全相关变更。
- `migration`：数据库、数据或状态迁移。
- `api-change`：API 合同或接口行为变化。

`risk_level`：

- `low`：局部变更，回滚简单，无数据或兼容性影响。
- `medium`：影响核心流程、异步任务、外部接口或配置行为。
- `high`：涉及权限、安全、数据迁移、批量任务、不可逆操作或跨模块行为。

`compatibility`：

- `backward-compatible`：向后兼容。
- `breaking-change`：存在破坏性变更。
- `not-applicable`：不适用。
- `unknown`：无法确认，必须在 Markdown 中说明原因。

`data_impact`：

- `none`：无数据影响。
- `read-only`：只读访问变化。
- `schema-change`：表结构或索引变化。
- `data-migration`：存量数据迁移。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MingyiSecLab/Mingyi-Atlas](https://github.com/MingyiSecLab/Mingyi-Atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
