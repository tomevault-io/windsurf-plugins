---
trigger: always_on
description: <!-- MONOCO_MANAGED_START -->
---

<!-- MONOCO_MANAGED_START -->
<!-- From resources -->
# Doc-Extractor: 文档标准化与渲染工具

将各种文档格式转换为标准化 WebP 页面序列的文档提取和渲染系统，适用于 VLM（视觉语言模型）消费。

## 概述

Doc-Extractor 提供了一个内容寻址的文档存储系统，支持自动格式标准化：

- **输入**: PDF、DOCX、PPTX、XLSX、图片（PNG、JPG）、压缩包（ZIP、TAR、RAR、7Z）
- **输出**: 可配置 DPI 和质量的 WebP 页面序列
- **存储**: 基于 SHA256 的内容寻址，存储在 `~/.monoco/blobs/`

## 命令

### 提取文档
```bash
monoco doc-extractor extract <文件> [选项]
```

选项：
- `--dpi, -d`: 渲染 DPI（72-300，默认：150）
- `--quality, -q`: WebP 质量（1-100，默认：85）
- `--pages, -p`: 指定渲染页面（例如："1-5,10,15-20"）

### 列出提取的文档
```bash
monoco doc-extractor list [--category <类别>] [--limit <数量>]
```

### 搜索文档
```bash
monoco doc-extractor search <查询>
```

### 显示文档详情
```bash
monoco doc-extractor show <哈希前缀>
monoco doc-extractor cat <哈希前缀>    # 显示元数据 JSON
monoco doc-extractor source <哈希前缀> # 显示源文件/压缩包信息
```

### 索引管理
```bash
monoco doc-extractor index rebuild   # 从 blobs 重建索引
monoco doc-extractor index stats     # 显示索引统计
monoco doc-extractor index clear     # 清空索引（保留 blobs）
monoco doc-extractor index path      # 显示索引文件路径
```

### 清理
```bash
monoco doc-extractor clean [--older-than <天数>] [--dry-run]
monoco doc-extractor delete <哈希前缀> [--force]
```

## 存储结构

```
~/.monoco/blobs/
├── index.yaml              # 全局元数据索引
└── {sha256_hash}/          # 内容寻址目录
    ├── meta.json           # 文档元数据
    ├── source.{ext}        # 原始文件（保留扩展名）
    ├── source.pdf          # 标准化 PDF 格式
    └── pages/
        ├── 0.webp          # 第 0 页渲染
        ├── 1.webp          # 第 1 页渲染
        └── ...
```

## Python API

```python
from monoco.features.doc_extractor import DocExtractor, ExtractConfig

extractor = DocExtractor()
config = ExtractConfig(dpi=150, quality=85)
result = await extractor.extract("/path/to/document.pdf", config)

print(f"Hash: {result.blob.hash}")
print(f"Pages: {result.page_count}")
print(f"Cached: {result.is_cached}")
```

## 核心原则

1. **内容寻址**: 文件按 SHA256 哈希存储 - 自动去重
2. **格式标准化**: 所有文档先转为 PDF，再渲染为 WebP
3. **压缩包支持**: 自动解压 ZIP 等压缩包，追踪内部文档来源
4. **缓存感知**: 提取结果被缓存；重复提取立即返回缓存结果


---

<!-- From resources -->
## Ralph Loop

> **重生后接力胜过泥泞中挣扎。**

当当前 Agent 遇到瓶颈（上下文不足、陷入局部最优、需要全新视角）时，启动继任 Agent 继续完成 Issue。

## 核心概念

- **Last Words** - 当前 Agent 留给继任者的关键信息：已完成的工作、当前状态、下一步建议
- **接力而非重启** - 继任者继承 Issue 上下文和工作环境，而不是从零开始
- **自愿让贤** - 当当前 Agent 判断自己效率下降时主动触发，而非强行坚持

## 命令

- `monoco ralph --issue {issue-id} --prompt "{last-words}"` - 直接传递遗言字符串
- `monoco ralph --issue {issue-id} --path {last-words-file}` - 从文件读取遗言（用于长内容）
- `monoco ralph --issue {issue-id}` - 让系统自动生成上下文摘要作为 Last Words

## 自动触发机制

Ralph Loop 会在以下任一条件满足时自动触发（无需人工干预）：

### 工具调用计数

| 阈值 | 行为 |
|------|------|
| **150 次** | 警告：提示上下文已使用约 75%，建议准备收尾 |
| **175 次** | 严重警告：提示上下文已使用约 87.5%，强烈建议完成当前里程碑或准备接力 |
| **200 次** | **强制接力**：自动启动 `monoco ralph`，当前 Agent 结束会话 |

### 内容长度累计

基于响应内容字符数统计（1 token ≈ 4 字符）：

| 阈值 | 行为 |
|------|------|
| **300k 字符** (~75k tokens) | 警告：建议准备收尾 |
| **350k 字符** (~87.5k tokens) | 严重警告：建议尽快完成或接力 |
| **400k 字符** (~100k tokens) | **强制接力**：自动启动 `monoco ralph` |

### 触发范围

**工具调用计数**追踪以下工具：
- `Bash`, `Write`, `Edit`, `Read`
- `Glob`, `Grep`, `Task`
- `WebFetch`, `WebSearch`

**内容长度**统计以下工具的响应：
- `Read`, `Glob`, `Grep`
- `WebFetch`, `WebSearch`

### 跳过机制

如需临时禁用自动触发（例如正在进行关键原子操作）：

```bash
export MONOCO_SKIP_RALPH=1
```

设置后，即使达到阈值也不会强制接力。

## 工作流

1. **自我评估**：当前 Agent 判断是否遇到瓶颈
   - 上下文窗口即将用尽（自动触发机制会处理）
   - 多次尝试同一问题无果
   - 感觉陷入细节而迷失全局

2. **撰写 Last Words**：总结关键信息
   - ✅ 已完成的工作和验证结果
   - ✅ 当前代码/文件状态
   - ✅ 遇到的障碍或不确定的问题
   - ✅ 建议的下一步方向

3. **执行接力**：运行 `monoco ralph` 启动继任 Agent

4. **平滑过渡**：继任 Agent 读取 Last Words 和 Issue 上下文，继续推进

## 指南

### 何时使用 Ralph

| 适合使用 | 不适合使用 |
|---------|-----------|
| 复杂重构涉及大量文件 | 简单的 bug 修复 |
| 上下文窗口不足（自动触发） | 单文件修改 |
| 多次尝试无果，需要新视角 | 已经接近完成，只剩收尾 |
| 陷入技术细节需要全局审视 | 验证性测试 |

### Last Words 最佳实践

- **简明扼要**：聚焦关键信息，而非完整历史
- **状态优先**：描述"现在在哪里"而非"怎么来的"
- **方向明确**：给继任者一个清晰的下一步建议
- **诚实记录**：不隐瞒失败尝试，避免继任者重复踩坑

### 接力后

- 当前 Agent 正常结束会话
- 继任 Agent 在独立环境中启动，继承 Issue 上下文
- 无需手动同步文件，`monoco ralph` 会自动处理


---

<!-- From resources -->
## Monoco 核心

项目管理的核心命令。遵循 **Trunk Based Development (TBD)** 模式。

- **初始化**: `monoco init` (初始化新的 Monoco 项目)
- **配置**: `monoco config get|set <key> [value]` (管理配置)
- **同步**: `monoco sync` (与 agent 环境同步)
- **卸载**: `monoco uninstall` (清理 agent 集成)

---

## ⚠️ Agent 必读: Git 工作流协议 (Trunk-Branch)

在修改任何代码前,**必须**遵循以下步骤:

### 标准流程

1. **创建 Issue**: `monoco issue create feature -t "功能标题"`
2. **🔒 启动 Branch**: `monoco issue start FEAT-XXX --branch`
   - ⚠️ **强制要求隔离**: 使用 `--branch` 或 `--worktree` 参数
   - ❌ **严禁操作 Trunk**: 禁止在 Trunk (`main`/`master`) 分支直接修改代码
3. **实现功能**: 正常编码和测试
4. **同步文件**: `monoco issue sync-files` (提交前必须运行)
5. **提交审查**: `monoco issue submit FEAT-XXX`
6. **合拢至 Trunk**: `monoco issue close FEAT-XXX --solution implemented`

### 质量门禁

- Git Hooks 会自动运行 `monoco issue lint` 和测试
- 不要使用 `git commit --no-verify` 绕过检查
- Linter 会阻止在受保护的 Trunk 分支上的直接修改

> 📖 详见 `monoco-issue` skill 获取完整工作流文档。


---

<!-- From last_word -->
# Last-Word: Simplified Knowledge Update System

## Overview

The knowledge update system has been **significantly simplified**. The complex YAML-based workflow has been replaced with a direct, manual approach using `mdp` tool.

## Architecture

```
PreSessionStop Event
        ↓
   Hook Triggered
        ↓
  Show Guidance
        ↓
Agent Uses mdp → Direct Edit to MD
```

## Hook


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IndenScale/monoco-toolkit](https://github.com/IndenScale/monoco-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
