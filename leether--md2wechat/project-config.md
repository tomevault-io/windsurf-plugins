---
trigger: always_on
description: 从 Markdown 文件出发，改写→排版→发布三位一体的公众号文章管线。触发词：推公众号、发公众号、微信草稿箱、公众号排版、推送草稿、写公众号文章。
---


# 微信公众号文章管线（改写 → 排版 → 发布）

## 前置配置

本 skill 所有路径基于环境变量 `${PIPELINE_HOME}`（仓库根目录）和 `${NODE_PATH}`（Node 可执行文件路径）。

```bash
# 在 .env 或 shell 环境中设置（必须）
export PIPELINE_HOME=/path/to/md2wechat   # 本仓库根目录
export NODE_PATH=node                                     # 或 Node 完整路径
```

> **安装方式**：`git clone` 本仓库 → 复制 `.env.example` 为 `.env` → 填入凭据 → 设置 `PIPELINE_HOME`
>
> 仓库即 skill，skill 即仓库。所有脚本、规则、文档都在一个目录下。

## 完整流程

```
圆桌报告/素材 MD
  ↓ Step 0：风格改写（khazix-writer）
公众号文章 MD
  ↓ Step 0.5：Orchestrator 自动调度（推荐）
一键执行：渲染 + preflight + AutoHeal + bundle + 日志 + self_report
  ↓ Step 1：渲染 HTML（render_wechat_editorial.mjs）【自动 preflight】
微信合规 HTML（含 CTA + 群二维码）
  ↓ Step 2：准备图片（封面图必须有，正文插图可选）
封面图 PNG（≤2MB）
  ↓ Step 2.5：自动 Bundle（scripts/bundle_wechat_article.mjs）
路径已替换为文件名
  ↓ Step 3：推送（relay 跳板机 → create_wechat_draft.mjs → 微信 API）
草稿箱
  ↓ Step 4：回检验证（Audit Log + 逐行核验）
已验证的草稿
  ↓ Step 5：完成前核查（skill-compliance-harness — 不得跳过）
✅ 确认发布
```

> **Step 0.5 是本轮新增的核心入口。** 过去需要手动执行 render → preflight → bundle → push 多个命令，现在一个 orchestrator 命令自动调度完整 pipeline，含 AutoHeal 自动修复和结构化日志。

---

## Step 0.5：Orchestrator 自动调度（推荐，一键完成）

 orchestrator.mjs 是本轮改造的核心入口。它把过去分散在 SKILL.md 中的 5 个手动步骤压缩成**一个命令**。

```bash
${NODE_PATH} ${PIPELINE_HOME}/scripts/orchestrator.mjs \
  --input article.md \
  --account XINZHE \
  --title "文章标题" \
  --author "新褶" \
  --auto-fix \
  --dry-run          # 首次建议加 --dry-run 验证流程
```

### Orchestrator 自动执行的步骤

| 步骤 | 自动执行 | 说明 |
|------|---------|------|
| 1. 渲染 | ✅ | 调用 render_wechat_editorial.mjs，含自动 preflight |
| 2. 活记忆加载 | ✅ | 渲染器启动时自动打印历史摩擦点风险提示 |
| 3. AutoHeal | ✅（--auto-fix） | digest 超长→自动截断、图片超 2MB→自动压缩、local_path→bundle 处理 |
| 4. 重试渲染 | ✅（修复后） | AutoHeal 修复后自动重跑渲染 |
| 5. Bundle | ✅ | 自动替换路径、打包、验证完整性 |
| 6. 推送命令生成 | ✅（手动模式） | 输出可直接复制执行的 ssh/scp 命令 |
| 7. 自动推送 | ✅（--auto-push） | 自动 SSH/SCP 上传并远程执行推送，无需手动复制命令 |
| 8. 结构化日志 | ✅ | 写入 `.md2wechat-pipeline.jsonl` |
| 9. 自动 self_report | ✅ | 分析日志，自动捕获新摩擦点，更新 LESSONS_LEARNED.md |

### 关键参数

| 参数 | 说明 |
|------|------|
| `--input` | 源 Markdown 文件（必需） |
| `--account` | 微信账号 key（必需，对应 .env 凭据） |
| `--title` | 文章标题（覆盖 MD 中的 H1） |
| `--author` | 作者名 |
| `--auto-fix` | **推荐**。启用 AutoHeal 自动修复 L1 问题 |
| `--auto-push` | **推荐**。自动通过 SSH/SCP 完成推送，无需手动执行命令 |
| `--dry-run` | 不实际推送，只验证完整流程 |
| `--out-dir` | 自定义 bundle 输出目录 |
| `--thumb-image` | 封面图路径 |
| `--qr` | 二维码图片路径 |
| `--skip-image-check` | 纯文本文章或紧急调试时显式跳过封面/插图 L1 检查 |
| `--no-write-lessons` | 验证时只分析 self_report，不写 LESSONS_LEARNED 或生成规则 |

### 参数契约矩阵

| 层 | 入口 | 关键参数 | 下游契约 | 证明命令 |
|----|------|----------|----------|----------|
| Render | `scripts/render_wechat_editorial.mjs` | `--preflight-cover`, `--skip-image-check`, `--no-preflight` | 自动 preflight 必须收到封面和显式逃逸意图 | `node scripts/render_wechat_editorial.mjs --input examples/sample-article.md --output /tmp/test.html --no-preflight` |
| Preflight | `harness/preflight.mjs` | `--cover`, `--skip-image-check`, `--rules` | L1/Agent 失败阻断；Observation 只报告 | `node harness/preflight.mjs --html /tmp/test.html --md examples/sample-article.md --skip-image-check --json` |
| Orchestrator | `scripts/orchestrator.mjs` | `--thumb-image`, `--skip-image-check`, `--auto-fix`, `--no-write-lessons` | render/preflight/bundle/self_report 参数必须显式转发 | `node scripts/orchestrator.mjs --input article.md --account X --thumb-image cover.png --auto-fix --dry-run --no-write-lessons` |
| Bundle | `scripts/bundle_wechat_article.mjs` | `--thumb-image`, `--env`, `--qr` | bundle 必须携带封面、lint report、`.env` 和内联图片 manifest | `node scripts/bundle_wechat_article.mjs --html /tmp/test.html --out /tmp/bundle --thumb-image cover.png --env .env` |
| Relay draft | `scripts/create_wechat_draft.mjs` | `--thumb-image`, `--crop-235-1`, `--lint-report` | 手动/自动 relay 命令必须传递封面、裁剪、标题、作者、账号 | `node harness/test-orchestrator-command-contract.mjs` |
| Self-report | `harness/self_report.mjs` | `--write-lessons`, `--no-write`, `--auto-encode` | 验证默认应可 no-write；写入必须显式 | `node harness/self_report.mjs --analyze-log .md2wechat-pipeline.jsonl --no-write` |

### 典型使用流程

```bash
# 第一次：dry-run 验证
${NODE_PATH} ${PIPELINE_HOME}/scripts/orchestrator.mjs \
  --input article.md --account XINZHE --thumb-image cover.png --auto-fix --dry-run

# 确认无问题后，--auto-push 自动完成推送（无需手动复制命令）
${NODE_PATH} ${PIPELINE_HOME}/scripts/orchestrator.mjs \
  --input article.md --account XINZHE --thumb-image cover.png --auto-fix --auto-push

# 如果不想自动推送，去掉 --auto-push，orchestrator 会输出命令供手动执行
# 如果确认为纯文本文章，可显式加 --skip-image-check；不要依赖隐式跳过
```

### 结构化日志

每次运行会在 MD 同级目录生成 `.md2wechat-pipeline.jsonl`：

```jsonl
{"t":"2026-06-07T06:42:20.123Z","step":"render","status":"healed","reason":"local_path_absence_skipped_for_bundle"}
{"t":"2026-06-07T06:42:21.456Z","step":"bundle","status":"success","script":"bundle_wechat_article.mjs"}
```

这份日志是 self_report 自动分析的输入源，也是排查问题的第一手资料。

---

## Step 0：风格改写（必须使用 khazix-writer）

**这一步不能跳过。** 圆桌报告或研究素材是专业文本，需要改写成有"活人感"的公众号文章。

### 触发 khazix-writer

调用 `Skill` 工具，`skill: "khazix-writer"`，传入原始报告内容和改写要求。

> **khazix-writer 是外部 skill 依赖**。本仓库 `references/khazix-writer/` 包含其完整写作指南和规则文件（MIT License），供参考和 lint 使用。要触发风格改写，需确保 `khazix-writer` skill 已安装。

### ⚠️ 人格覆盖（必须执行）

khazix-writer 的 SKILL.md 以「数字生命卡兹克」为人格锚点写作，尾部固定带卡兹克署名和邮箱。**调用后必须覆盖以下内容**：

1. **身份替换**：忽略"你正在以数字生命卡兹克的身份写作"，替换为你自己的公众号身份

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leether/md2wechat](https://github.com/leether/md2wechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
