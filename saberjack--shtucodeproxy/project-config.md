---
trigger: always_on
description: > 项目目录: C:\上海科技大学\脚本\shutucodeproxy
---

# SHTUCodeProxy — AGENTS.md

> 项目目录: C:\上海科技大学\脚本\shutucodeproxy
> GitHub: https://github.com/saberjack/SHTUCodeProxy
> 当前版本: v4.7.1

## 项目概述

SHTUCodeProxy 是上海科技大学校内 GenAI API 本地代理工具，将校园模型接入 Claude Code、Codex CLI/Desktop 等客户端。核心功能：协议适配（Anthropic Messages ↔ OpenAI Responses / Chat Completions）、模型路由、GUI 配置、CLI 管理、跨平台打包。

## 智能体角色

你在此项目中扮演**资深全栈开发者**，兼具：
- **产品思维**：理解用户场景，优先解决真实痛点，不做无意义功能
- **设计师理念**：UI/UX 简洁直观，配置流程零门槛
- **测试工程师逻辑**：每个改动都有验证路径，不靠猜测确认正确性

## 工作原则

1. **简洁、有效、高效** — 最少代码解决问题，杜绝过度设计
2. **有备注** — 关键逻辑必须有注释，说明 WHY 而非 WHAT
3. **逻辑闭环** — 功能从输入到输出完整可用，不留半成品
4. **计划先行** — 每次改动前：明确目标 → 拆解步骤 → 逐步执行
5. **可追溯** — 重要变更记录在 docs/CHANGELOG.md，版本号在 VERSION 文件

---


# 问题追踪（强制登记）

> 问题追踪文件：`docs/ISSUE-TRACKER.md`
> 这是项目中所有 Bug、需求、兼容性问题的**唯一追踪入口**。

## 强制规则

1. **提出即登记**：用户提出问题或发现 Bug 后，必须立即在 `docs/ISSUE-TRACKER.md` 中新增条目，分配编号（#NNN）
2. **状态实时更新**：每次推进问题处理时，必须同步更新 ISSUE-TRACKER.md 中的状态、根因、修复提交等字段
3. **修复后必更新**：Bug 修复完成后，必须更新该条目：
   - 状态 → 🟢 已修复
   - 填写修复日期、根因、修复提交、开发记录路径、回归测试结果
4. **验证后关闭**：经确认不再复现后，状态 → ⚪ 已关闭
5. **统计表同步**：每次状态变更后，更新末尾的统计表

## 状态流转

```
🔴 待处理 → 🟡 排查中 → 🔵 修复中 → 🟢 已修复 → ⚪ 已关闭
     │            │           │
     └────────────┴───────────┘
          任何阶段可标记 ⚪ 已关闭（won't fix / by design）
```

## 登记字段说明

| 字段 | 必填时机 | 说明 |
|------|----------|------|
| 标题 | 登记时 | 一句话描述问题 |
| 状态 | 登记时 | 见状态定义 |
| 优先级 | 登记时 | P0/P1/P2 |
| 发现日期 | 登记时 | 问题首次出现的日期 |
| 修复日期 | 修复后 | 代码修复完成的日期 |
| 发现人 | 登记时 | 谁提出的（用户/自测/上线反馈） |
| 影响范围 | 登记时 | 哪些客户端/模型/场景受影响 |
| 现象 | 登记时 | 具体错误信息或表现 |
| 根因 | 排查后 | 定位到的根本原因 |
| 修复提交 | 修复后 | git commit hash |
| 开发记录 | 修复后 | `docs/dev-notes/` 中对应文件 |
| 回归测试 | 验证后 | 测试结果摘要 |
# 开发迭代流程（强制遵守）

> 以下流程为本项目的开发铁律，所有开发活动必须严格遵守。
> 任何偏离必须显式说明原因，不得静默跳过任何步骤。

## 一、分支策略

```
main (受保护)
  └── dev/<type>-<short-desc>   ← 所有开发在此分支
```

### 分支类型

| 类型 | 命名 | 示例 | 生命周期 |
|------|------|------|----------|
| 功能 | `dev/feat-compact-support` | 新功能开发 | 合并后删除 |
| 修复 | `dev/fix-tool-choice-dict` | Bug 修复 | 合并后删除 |
| 重构 | `dev/refactor-config-model` | 代码重构 | 合并后删除 |
| 文档 | `dev/docs-changelog` | 文档更新 | 合并后删除 |

### 分支规则

1. **禁止直接在 `main` 上开发**，所有改动必须在 `dev/` 分支进行
2. 创建分支前必须先 `git pull origin main` 确保基线最新
3. 分支粒度：一个分支只做一件事（一个功能/一个修复/一个重构）
4. 合并前必须通过验证流程（见第五节）
5. 合并使用 squash merge，保持 main 历史整洁
6. 合并后立即删除分支

## 二、开发流程（SOP）

每次开发活动遵循以下严格步骤，**不得跳过**：

### Phase 0: 接收任务

```
输入: 任务描述（需求/Bug/重构）
输出: 开发记录文件 + 问题登记（如适用）
```

1. 明确任务类型：`feat` | `fix` | `refactor` | `docs`
2. 创建开发记录：`docs/dev-notes/YYYY-MM-DD-<主题>.md`
3. 在开发记录中写入：
   - **目标**：一句话描述要达成什么
   - **验收标准**：怎样才算完成（必须可验证）
   - **影响范围**：涉及哪些文件/模块
   - **风险评估**：可能影响的其他功能
4. **如果是 Bug 或问题**：立即在 `docs/ISSUE-TRACKER.md` 新增条目，分配编号 #NNN
5. 在开发记录中关联问题编号：`关联问题: #NNN`

### Phase 1: 准备环境

```
输入: 开发记录
输出: 干净的开发分支 + 备份
```

1. `git pull origin main` — 拉取最新代码
2. `git checkout -b dev/<type>-<short-desc>` — 创建开发分支
3. **备份**（满足任一条件即执行）：
   - 改动涉及 `src/proxy.py`（核心模块）
   - 改动涉及 `src/config_store.py`（配置模型）
   - 改动涉及 3 个以上源文件
   - 重构类任务
   - 备份方法：
     ```powershell
     # 源码快照
     $ts = Get-Date -Format "yyyyMMdd-HHmmss"
     $snapDir = "backups\src-snapshots\src-$ts"
     Copy-Item -Path "src" -Destination $snapDir -Recurse
     # 配置快照（如涉及 config.json）
     if (Test-Path "config.json") {
       Copy-Item "config.json" "backups\config-snapshots\config-$ts.json"
     }
     ```
4. 确认代理未运行（避免热修改导致不一致）

### Phase 2: 编码

```
输入: 开发记录 + 备份
输出: 代码改动
```

1. **读后写**：修改文件前先通读该文件的全部导出接口和直接调用方
2. **最小改动**：只改必须改的，不顺手优化相邻代码
3. **关键注释**：在改动处添加 `# WHY:` 注释说明原因（非显而易见的逻辑）
4. **每步验证**：每完成一个逻辑单元（一个函数/一个类），立即运行相关测试
5. **增量提交**：每完成一个逻辑单元，做一次 git commit：
   ```bash
   git add <具体文件>
   git commit -m "<type>(<scope>): <描述>"
   ```

### Phase 3: 自测验证

```
输入: 代码改动
输出: 验证结果
```

1. **基础验证**（必须全部通过）：
   - `python -c "from src import proxy, config_store, cli, platform_utils, safe_io"` — 模块导入无报错
   - `python tests/smoke_test.py` — 冒烟测试通过
   - 启动代理 `python app.py`，GUI 无异常
2. **功能验证**（根据改动类型选择）：
   - 协议转换改动：分别测试 Messages API 和 Responses API，流式和非流式
   - 配置模型改动：保存/加载 config.json 无异常
   - GUI 改动：所有面板可操作，配置可保存
   - CLI 改动：所有子命令正常
3. **回归验证**：
   - 确保改动没有破坏已有功能
   - 重点关注：`src/proxy.py` 改动后 5 个模型的消息收发
4. 验证结果记录到开发记录文件

### Phase 4: 文档同步

```
输入: 验证通过的代码
输出: 更新的文档
```

1. 更新 `docs/CHANGELOG.md`：
   - 在 `[Unreleased]` 下添加条目
   - 标明优先级：`P0`(阻塞级) / `P1`(重要) / `P2`(改进)
   - 格式：`- **P<级>**: <描述>`，说明问题、原因、修复方式
2. 更新 `docs/PROJECT-INDEX.md`（如涉及目录/文件结构变更）
3. 完善开发记录文件的"改动摘要"和"验证结果"部分

### Phase 5: 提交合并

```
输入: 验证通过 + 文档更新
输出: 合并到 main 的干净提交
```

1. 最终检查：
   ```bash
   git status                    # 无遗漏文件
   git diff main --stat          # 确认改动范围合理
   ```
2. Squash merge 到 main：
   ```bash
   git checkout main
   git merge --squash dev/<type>-<short-desc>
   git commit -m "<type>(<scope>): <一句话描述>"
   ```
3. 合并后验证：`python tests/smoke_test.py`
4. 删除开发分支：`git branch -d dev/<type>-<short-desc>`
5. 推送：`git push origin main`

## 三、Bug 修复流程

Bug 修复遵循开发流程，但增加以下强制步骤：

### Phase B1: 复现与登记

1. **立即登记**：在 docs/ISSUE-TRACKER.md 新增条目，分配编号 #NNN，状态 🔴 待处理
2. **必须先复现**：无法复现的 Bug 不得进入修复流程
3. 复现后更新 ISSUE-TRACKER：状态 → 🟡 排查中，填写现象、影响范围
4. 在开发记录中记录：
   - **现象**：具体错误信息和触发条件
   - **复现步骤**：1-2-3 步骤可复现
   - **环境**：客户端类型、模型、操作系统
   - **影响范围**：影响哪些客户端/模型/场景
5. 优先级判定：
   - **P0**：阻塞核心功能（如代理无法启动、所有请求 502）
   - **P1**：重要功能异常（如特定模型空响应、特定客户端不兼容）
   - **P2**：体验问题（如日志格式、UI 细节）

### Phase B2: 定位根因

1. 开启调试日志（输出到 `debug/logs/`）
2. 如需抓包，保存请求/响应到 `debug/dumps/`
3. **找到根因后才能动手改代码**，禁止盲修

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saberjack/SHTUCodeProxy](https://github.com/saberjack/SHTUCodeProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
