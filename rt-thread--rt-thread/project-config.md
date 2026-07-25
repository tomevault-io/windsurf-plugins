---
trigger: always_on
description: RT-Thread is a real-time operating system (RTOS) for embedded devices. When working with RT-Thread code, please follow these guidelines to ensure high-quality contributions.
---

# GitHub Copilot Instructions for RT-Thread / RT-Thread GitHub Copilot 指南

## Overview / 概述

RT-Thread is a real-time operating system (RTOS) for embedded devices. When working with RT-Thread code, please follow these guidelines to ensure high-quality contributions.

RT-Thread 是一个面向嵌入式设备的实时操作系统（RTOS）。在处理 RT-Thread 代码时，请遵循以下指南以确保高质量的贡献。

**When reviewing Pull Requests (PRs), you MUST check all items in the PR Review Checklist section and provide feedback according to the PR Review Instructions. / 在审查 Pull Request (PR) 时，必须检查 PR 审查清单部分中的所有项目，并根据 PR 审查指令提供反馈。**

## Code Review Guidelines / 代码审查指南

### Language Requirements / 语言要求

When reviewing code, provide feedback in **both English and Chinese** to ensure accessibility for all contributors.

在审查代码时，请同时使用**英文和中文**提供反馈，以确保所有贡献者都能理解。

### Review Focus Areas / 审查重点领域

1. **Real-Time Performance / 实时性能**
   - Verify interrupt handling efficiency / 验证中断处理效率
2. **Memory Management / 内存管理**
   - Detect memory leaks / 检测内存泄漏
   - Verify proper memory allocation/deallocation / 验证正确的内存分配/释放
   - Check stack usage optimization / 检查栈使用优化
3. **Code Style / 代码风格**
   - Follow RT-Thread coding standards / 遵循 RT-Thread 编码标准
   - Maintain consistent naming conventions / 保持一致的命名约定
   - Ensure proper code comments (not documentation) / 确保适当的代码注释（而非文档）
4. **PR Review Checklist / PR 审查清单**
   - **PR Title Review / PR 标题审查**：
     - Check if PR title has proper prefix format / 检查 PR 标题是否有正确的前缀格式
     - Verify prefix follows pattern: `[module/vendor][subsystem]` or `[module/vendor]` in lowercase / 验证前缀遵循格式：小写的 `[模块/厂商][子系统]` 或 `[模块/厂商]`
     - Verify title describes changes based on modified files / 验证标题基于修改的文件描述变更
     - Check if title is specific enough (avoid vague terms like "fix bug", "optimize code") / 检查标题是否足够具体（避免模糊术语如"修复问题"、"代码优化"）
     - If title lacks prefix or uses incorrect format, suggest: "PR title should follow format: `[module][subsystem] Description`. Example: `[stm32][drivers] Fix UART interrupt handling issue`" / 如果标题缺少前缀或格式错误，建议："PR 标题应遵循格式：`[模块][子系统] 描述`。示例：`[stm32][drivers] Fix UART interrupt handling issue`"
   - **PR Description Review / PR 内容审查**：
     - Check if PR description provides overview of modified files / 检查 PR 描述是否提供了修改文件的总概
     - Verify description explains: What (what changes), Why (why needed), How (which files modified) / 验证描述是否说明：What（做了什么修改）、Why（为什么需要）、How（修改了哪些文件）
     - If description is missing or insufficient, suggest adding description with modified files list / 如果描述缺失或不充分，建议添加包含修改文件列表的描述
   - **PR File Modification Review / PR 修改文件审查**：
     - Check if PR contains multiple unrelated features / 检查 PR 是否包含多个不相关的特性
     - If PR mixes multiple features, suggest splitting into separate PRs / 如果 PR 混杂多个特性，建议拆分为多个 PR
     - Verify all file changes are related to the same feature/bug fix / 验证所有文件修改是否与同一功能/错误修复相关
   - **PR Commit Review / PR Commit 审查**：
     - Check commit message format (should follow PR title format) / 检查 commit 消息格式（应遵循 PR 标题格式）
     - Verify if commits are properly organized / 验证 commit 是否组织得当
     - If routine changes (style fixes, minor bugs, docs, typos) are split into multiple commits, suggest squashing them / 如果常规修改（风格修复、小错误、文档、拼写）被拆分为多个 commit，建议压缩它们
     - Important commits (major features, refactoring, critical bugs) can remain separate / 重要提交（主要功能、重构、关键错误）可以保持独立
     - If commit messages don't follow format, suggest: "Commit message should follow format: `[module][subsystem] Description`. Consider squashing routine changes into one commit." / 如果 commit 消息不符合格式，建议："Commit 消息应遵循格式：`[模块][子系统] 描述`。考虑将常规修改压缩为一个 commit。"

### PR Review Instructions / PR 审查指令

**When reviewing a PR, you MUST systematically check the following items and provide feedback for any violations / 审查 PR 时，必须系统性地检查以下项目，并对任何违规行为提供反馈：**

#### Step 1: PR Title Check / 步骤 1：PR 标题检查

1. **Check prefix format / 检查前缀格式**:
   - Extract the PR title from the PR / 从 PR 中提取 PR 标题
   - Verify if it starts with `[module][subsystem]` or `[module]` pattern in lowercase / 验证是否以小写的 `[模块][子系统]` 或 `[模块]` 格式开头
   - Check if prefix matches the modified files (e.g., if files are in `bsp/stm32/drivers/`, prefix should be `[stm32][drivers]` or similar) / 检查前缀是否与修改的文件匹配（例如，如果文件在 `bsp/stm32/drivers/`，前缀应为 `[stm32][drivers]` 或类似）
   - If prefix is missing, incorrect case (e.g., `[STM32]`), or doesn't match files, provide feedback / 如果前缀缺失、大小写错误（如 `[STM32]`）或不匹配文件，提供反馈
   - **Feedback template / 反馈模板**:
     ```
     🟡 [PR Title/PR 标题]: Missing or incorrect prefix format / 缺少或错误的前缀格式
     
     English: PR title should follow format: `[module][subsystem] Description` in lowercase. 
     Current title: `{current_title}`. 
     Based on modified files, suggested title: `{suggested_title}`.
     
     中文：PR 标题应遵循格式：小写的 `[模块][子系统] 描述`。
     当前标题：`{current_title}`。
     基于修改的文件，建议标题：`{suggested_title}`。
     ```

2. **Check title specificity / 检查标题具体性**:
   - Analyze modified files to understand what changes were made / 分析修改的文件以了解所做的更改
   - Verify if title accurately describes changes based on modified files / 验证标题是否基于修改的文件准确描述更改
   - Check for vague terms: "fix bug", "optimize code", "update", "modify", etc. / 检查模糊术语："修复问题"、"代码优化"、"更新"、"修改"等

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RT-Thread/rt-thread](https://github.com/RT-Thread/rt-thread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
