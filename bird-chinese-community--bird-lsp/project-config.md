---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## base info

代码输出语言应该是 English, 但可以包含中文注释和文档字符串以提高可读性。

与我的对话应该使用 Chinese, 但代码注释和文档字符串可以使用 English 来提高国际化可读性。

你应该自行分析代码库结构和依赖关系，以便在修改或添加代码时能够正确地处理模块之间的关系。

MUST DO: 每次回答前都应该自行在 ./.agents/skills/INDEX.md 中查找相关技能/最佳实践，以便在需要时调用它们来完成特定任务。

MUST DO: 带有 `<!-- CI START -->` / `<!-- CI END -->` 标记的 README 区块只能由 CI 自动更新。除非用户明确要求临时手动改动，否则本地修改时应只更新源数据或同步脚本（如 registry / workflow / sync script），不要直接手改这些区块内容。

MUST DO: 永远不得将 `refer/config-examples-private` 下的任何内容/路径暴露在公共代码库中，这些内容只能在本地环境使用，且不应被提交到 Git 仓库，如需使用请数据脱敏后使用其提取的「最小可复现」片段。

MUST DO: 每个 PR 合并前都需要等待至少 120s 的时间 (建议分为 4 次，每次 30s)，以便让 CI 系统和 Auto Reviewer 有足够的时间检测到潜在的问题，如果门卫期间有 review 出现，应立即修复问题，在修复问题后应立即回复 review 并再次等待至少 120s。

如果是来自 Gemini 的 review，你修复并回复后应该再开一条评论：

```
@gemini-code-assist /gemini

Review again: Conduct an in-depth analysis of the latest PR's code logic capabilities, conditional operations, coupling, security vulnerabilities, performance issues, DRY violations, and more.

The review time MUST BE kept within 90 seconds.
```

## Project Overview

**BIRD-LSP** 是一个为 BIRD2（BIRD Internet Routing Daemon）配置文件提供 Language Server Protocol (LSP) 支持的工具链项目，包含语法高亮、诊断、格式化、代码补全等功能。

- **技术栈**: TypeScript + Tree-sitter + dprint + vscode-languageserver-node
- **架构**: Turborepo 管理的 monorepo
- **包管理器**: pnpm
- **测试框架**: Vitest

## Repository Structure

```
packages/
  @birdcc/parser/              # Tree-sitter grammar + WASM + JS adapter
  @birdcc/core/                # AST / Symbol Table / Type Checker / Cross-file resolution
  @birdcc/linter/              # Lint rules / Diagnostics (32+ rules)
  @birdcc/lsp/                 # LSP server implementation
  @birdcc/formatter/           # dprint plugin + builtin formatter
  @birdcc/dprint-plugin-bird/  # Official dprint plugin (Rust/WASM)
  @birdcc/cli/                 # birdcc CLI (lint/fmt/lsp commands)

refer/                         # Git submodules - reference materials
  BIRD-source-code/            # Official BIRD daemon C source
  BIRD-tm-language-grammar/    # Existing TextMate grammar
  BIRD2-vim-grammar/           # Vim syntax highlighting

.agents/skills/                # Claude Code skills for this project
```

## Common Commands

### Development

```bash
# Install dependencies
pnpm install

# Build all packages (depends on ^build, outputs to dist/)
pnpm build

# Run all tests (depends on build)
pnpm test

# Run tests for a specific package
pnpm --filter @birdcc/parser test

# Run single test file
pnpm vitest run packages/@birdcc/parser/src/index.test.ts

# Run tests in watch mode
pnpm vitest

# Type check all packages
pnpm typecheck

# Lint code (uses oxlint)
pnpm lint

# Format check (uses oxfmt)
pnpm format
```

### CLI Commands (after build)

CLI 入口位于 `packages/@birdcc/cli/dist/cli.js`:

```bash
# Lint BIRD2 config files
node packages/@birdcc/cli/dist/cli.js lint <file.conf> --format json

# Lint with BIRD validation
node packages/@birdcc/cli/dist/cli.js lint <file.conf> --bird

# Cross-file lint (default enabled)
node packages/@birdcc/cli/dist/cli.js lint <file.conf> --cross-file --include-max-depth 10

# Format check
node packages/@birdcc/cli/dist/cli.js fmt <file.conf> --check

# Format write
node packages/@birdcc/cli/dist/cli.js fmt <file.conf> --write

# Format with specific engine
node packages/@birdcc/cli/dist/cli.js fmt <file.conf> --write --engine dprint

# Start LSP server
node packages/@birdcc/cli/dist/cli.js lsp --stdio
```

### Turborepo Commands

```bash
# Run build for affected packages
turbo run build --affected

# Run tests with cache
turbo run test

# Force rebuild without cache
turbo run build --force
```

## Architecture

### 分层架构

```
Editors (VSCode/Neovim)
         |
         v
    @birdcc/lsp
 (diagnostics/hover/completion/definition/references/documentSymbol)
     |                      \
     v                       v
  @birdcc/linter        @birdcc/formatter
 (32+ Rules/Diagnostics)  (dprint + builtin)
     ^                       ^
     |                       |
  @birdcc/core  <-----------+
 (AST/Symbol/TypeChecker/CrossFile)
     ^
     |
  @birdcc/parser
 (tree-sitter + wasm adapter)
     |
     v
bird -p adapter（optional）
```

### 关键设计决策

1. **Parser**: Tree-sitter 负责语法解析，产出 CST/AST，支持错误恢复
2. **语义层**: `@birdcc/core` 负责符号表、类型检查、跨文件解析
3. **规则层**: `@birdcc/linter` 负责协议/安全/性能规则（32+ 条）
4. **Formatter**: dprint 插件为主，builtin 作为安全回退
5. **BIRD 集成**: 使用 `bird -p` 子进程验证，支持自定义验证命令模板
6. **跨文件**: 支持 `include` 展开、符号表合并、循环检测

### 双层语言模型

BIRD2 配置包含两个层次：

1. **配置声明层**: `protocol/template/filter/function` 结构
2. **Filter 表达式层**: 15+ 类型、控制流、运算符重载、方法调用

Tree-sitter 负责结构解析，类型语义交给 `@birdcc/core`，协议规则交给 `@birdcc/linter`。

## Git Submodules

项目依赖以下参考仓库：

```bash
# 初始化 submodules
git submodule update --init --recursive

# 更新 submodules
git submodule update --recursive --remote
```

- `refer/BIRD-source-code`: BIRD 官方源码，用于参考 parser 实现
- `refer/BIRD-tm-language-grammar`: 现有 TextMate 语法
- `refer/BIRD2-vim-grammar`: Vim 语法高亮

## Current Implementation Status

基于 TASKLIST.md 的执行进展:

- **M1** ✅: Tree-sitter grammar 已完成，支持多词短语识别（`local as`、`next hop self` 等）
- **M2** ✅: LSP 基础 + 错误恢复 + `bird -p` PoC 已完成

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bird-chinese-community/BIRD-LSP](https://github.com/bird-chinese-community/BIRD-LSP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
