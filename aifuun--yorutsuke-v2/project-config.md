---
trigger: always_on
description: Local-first AI accounting assistant for second-hand business users. Drag receipts → configurable AI processing (instant/batch/hybrid) → transaction reports.
---

# Yorutsuke v2 (夜付け)

Local-first AI accounting assistant for second-hand business users. Drag receipts → configurable AI processing (instant/batch/hybrid) → transaction reports.

> "你只管赚钱，记账交给电脑里的 AI"

## AI-First Development Principles

> **This project is designed for AI-assisted development. All code, templates, and patterns prioritize AI readability and reliability.**

### Core Principles

| Principle | Rationale |
|-----------|-----------|
| **Explicit > Abstract** | AI doesn't remember implicit conventions between sessions |
| **Clear > DRY** | AI can generate repetitive code quickly; clarity prevents errors |
| **Concrete > Generic** | Generic types weaken type inference, causing AI mistakes |
| **Simple > Clever** | AI understands straightforward patterns more reliably |

### Guidelines for AI

1. **Copy templates directly** - Don't try to abstract or generalize
2. **Follow explicit patterns** - Each pillar has one recommended way
3. **Check checklists** - Use `.prot/checklists/` before and after coding
4. **Avoid generic factories** - Use explicit functions (e.g., `parseUser` not `createParser`)

See also: `.claude/rules/debugging.md`, `.claude/rules/time-handling.md`

### Work Flow

**MVP → Issues → TODO 的协同机制**

| 文档 | 用途 |
|------|------|
| `.claude/WORKFLOW.md` | 工作流索引 & 速查表 (主入口) |
| `.claude/rules/workflow.md` | 核心原则 (三层架构) |
| `.claude/workflow/planning.md` | Two-Step Planning 详细指南 |

**三层架构**:
- **MVP 文件** (`docs/dev/MVP*.md`)：目标和验收标准（路标）
- **GitHub Issues + Issue Plans** (`.claude/plans/active/`)：详细的技术任务和实现方案（施工图）
- **MEMORY.md** (ADR index only)：长期知识和架构决策（备忘）

**Two-Step Planning**:
1. **Step 1** (40 min): MVP 分解 → 创建 Issues + 依赖图
2. **Step 2** (1-2h): Feature 详细规划 → Dev Plan + Test Cases (just-in-time)

**Templates & Planning**:

| 层级 | Template | Use Case | Command |
|------|----------|----------|---------|
| 战略 | `TEMPLATE-mvp.md` | New MVP release | `*plan` |
| 战役 | `TEMPLATE-feature-plan.md` | Before coding | `*issue pick` |
| 战役 | `TEMPLATE-github-issue.md` | Step 1 issues | `*issue new` |
| 战术 | Issue Plan files | Session work | `.claude/plans/active/#XXX.md` |

Templates location: `.claude/workflow/templates/`
Complete guide: `.claude/workflow/templates/README.md`

---

## Overview

- **Tech Stack**: Tauri 2, React 19, TypeScript, AWS CDK
- **Architecture**: AI_DEV_PROT v15
- **Current Version**: 0.1.0
- **Target Users**: Budget-conscious second-hand computer users (Mercari/Yahoo Auctions sellers)
- **Migration**: 本项目是迁移项目，原项目参考 `../yorutsuke`，参考功能和使用方法，架构完全改变

### Refer to Yorutsuke

参考原项目时的原则：

1. **先看原项目了解"做什么"** - 功能需求、边界条件、已踩过的坑
2. **再按新架构决定"怎么做"** - 遵循 Pillars、分层规范
3. **不直接复制，而是重写** - 适配 AI_DEV_PROT v15 架构

| 参考内容 | 价值 | 说明 |
|----------|------|------|
| Rust 逻辑 | ✅ 高 | 压缩参数、IPC 实现已调优 |
| 业务规则 | ✅ 高 | MD5 去重、quota 限制已验证 |
| 错误处理 | ✅ 高 | 异常场景已踩过坑 |
| 代码结构 | ⚠️ 低 | 架构完全不同，勿复制 |
| React hooks | ⚠️ 低 | 需适配 Pillar L headless |

## Core Flow

```
日間採集 (Day)          処理 (Configurable)         結果確認
──────────────────────────────────────────────────────────────────────────────────
Receipt Drop  →  Local SQLite  →  S3 Upload  →  Nova Lite OCR  →  Transaction Result
     ↓              ↓              ↓                                    ↓
  WebP圧縮      Queue管理   Instant/Batch/Hybrid                  確認/編集
```

**処理方式**: Admin で設定可能（即座処理/バッチ/ハイブリッド）

## Project Structure

| Directory | Purpose |
|-----------|---------|
| **`/app`** | Desktop app (Tauri 2 + React 19) - user-facing receipt capture & sync |
| **`/admin`** | Admin web panel (Vite + React) - operational monitoring & batch config |
| **`/infra`** | AWS CDK - two stacks (App + Admin) + Lambda functions + shared layer |
| **`/docs`** | Source of truth - architecture, design, operations, dev workflow |
| **`/.claude`** | Claude Code config - development rules, workflow templates, active plans |
| **`/.prot`** | AI_DEV_PROT v15 - pillars, checklists, patterns |

## Commands

**NPM Workspaces** - All commands run from root directory:

```bash
# Development
npm run dev:app               # Start Tauri app development (with hot reload)
npm run dev:admin             # Start Admin panel development (http://localhost:5173)

# Building
npm run build                 # Build both app and admin for production

# Deployment
npm run deploy                # Deploy infrastructure + sync environment variables

# Environment Management
npm run env:sync              # Sync CDK outputs to app/.env.local and admin/.env

# Testing
npm run test                  # Run all tests
npm run test:watch            # Watch mode (auto re-run on file change)
npm run test:coverage         # Generate coverage reports

# Workspace-specific commands (if needed)
npm run -w app tauri build    # Build Tauri app directly
npm run -w infra diff         # Preview CDK changes
npm run -w infra synth        # Synthesize CDK template
```

**Lambda development** (TypeScript workflow):
```bash
# Build Lambda TypeScript files (after code changes)
cd infra && npm run build:lambdas          # Compiles .ts → .mjs in .lambda-dist/

# Layer-only deploy (after modifying shared-layer/)
cd infra && npm run build:lambdas && \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aifuun/yorutsuke-v2](https://github.com/aifuun/yorutsuke-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
