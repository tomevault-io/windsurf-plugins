---
trigger: always_on
description: WxEcho 项目的 Claude Code Agent 工作规范与上下文文档。
---

# AGENTS.md

WxEcho 项目的 Claude Code Agent 工作规范与上下文文档。

---

## 项目介绍

**WxEcho** — macOS 微信聊天记录导出工具。

从运行中的微信进程提取数据库密钥（SQLCipher 4 / AES-256-CBC），解密本地 WCDB SQLite 数据库，导出为 TXT / CSV / JSON 格式。长期愿景是做成一个**聊天数据产品**，陆续支持聊天统计、年度报告等情感化功能。

**核心价值主张**：回声，你的聊天记录完整回响。
**设计语言**：微信浅绿色（`#07c160`）+ Mac 高级感，温暖而有节制。

### 技术栈

| 层次 | 技术 |
|------|------|
| CLI 封装 | TypeScript + Commander.js + esbuild |
| 解密核心 | TypeScript + Node.js crypto（无 Python 依赖） |
| 导出核心 | Python 3（sqlite3 标准库，无外部依赖） |
| 内存扫描 | C + Mach VM API（`find_all_keys_macos`） |
| 数据库 | SQLite / WCDB / SQLCipher 4 |
| 展示端 | React 18 + Vite（landing page） |

### 支持版本

- macOS 11+（**Apple Silicon only**，M1/M2/M3/M4...）
- WeChat 4.x（已测试 WeChat 4.1.8.106 / 4.1.5.240）

---

## 文件目录

```
WxEcho/
├── bin/                        # CLI launcher script（npm 全局安装后为 wxecho 命令入口）
├── src/                        # TypeScript CLI 源码
│   ├── cli.ts                  # 主入口（Commander.js）
│   ├── commands/
│   │   ├── export.ts           # wxecho export — 列出/导出聊天记录
│   │   ├── decrypt.ts          # wxecho decrypt — 解密数据库
│   │   └── keys.ts             # wxecho keys — 从微信进程提取密钥
│   └── utils/
│       ├── doctor.ts            # wxecho doctor — 环境依赖检测
│       ├── decrypt_db.ts        # wxecho decrypt — SQLCipher 解密（TypeScript/Node.js crypto）
│       └── python.ts            # Python 子进程封装（用于 export）
├── py/                         # Python 核心逻辑
│   ├── config.py               # 配置加载与自动检测
│   ├── export_chat.py           # 聊天记录导出（TXT/CSV/JSON）
│   ├── key_utils.py             # 密钥文件处理
│   ├── find_all_keys_macos.c    # C 内存扫描器源码
│   ├── find_all_keys_macos      # 编译后的二进制（Apple Silicon only）
│   ├── decrypted/               # 解密后数据库输出目录（gitignored）
│   └── exported/                # 导出文件输出目录（gitignored）
├── landing/                     # React landing page
│   ├── src/
│   │   ├── App.tsx
│   │   ├── components/           # Hero / Features / Steps / Footer
│   │   └── styles/index.css     # WeChat 绿色主题，亮/暗模式
│   ├── public/
│   ├── index.html
│   ├── package.json
│   └── vite.config.ts
├── scripts/
│   ├── build-cli.ts             # esbuild CLI 打包脚本
│   └── postinstall.sh           # npm install 后自动安装 Python 依赖 + 编译 C 扫描器
├── dist/                        # 构建产物
│   ├── cli.js
│   └── landing/
├── package.json                 # npm 包配置（name: @walkerch/wxecho, version: 1.0.0）
├── tsconfig.json
└── README.md
```

### npm scripts（根目录）

```bash
npm run build        # 构建 CLI → dist/cli.js
npm run build:landing # 构建 landing page → dist/landing/
npm run build:all    # 同时构建 CLI + landing
npm run dev          # 直接用 tsx 运行 CLI（无需 build）
npm run typecheck    # TypeScript 类型检查
```

### 发布文件（npm publish）

```
dist/, bin/, py/config.py, py/export_chat.py, py/key_utils.py, py/*.c, py/find_all_keys_macos
```

> 注意：`decrypt_db.py` 已移除，解密逻辑已改写为 TypeScript。

---

## AGENT Workflow

### 工作目录约定

所有 agent 工作产物放在 `~/agents/`（即 `$HOME/agents/`），**不在本仓库内**。

```
~/agents/
├── briefs/           # 任务简报（本次会话要做什么）
├── handoffs/         # 中间态笔记、可续记的上下文
│   ├── WORKLOG.md    # 核心工作日志（必须）
│   └── runtime_logs/ # 运行时日志目录（必须）
├── reports/          # 最终报告
├── context/          # 可复用上下文文档
└── scratch/          # Agent 一次性输出（用完即弃）
```

### 任务发起流程

1. **创建 brief** → `~/agents/briefs/<task-name>.md`，写清楚目标、约束、验收标准
2. **创建 worklog 入口** → 用 `log_work.sh` 记录 planned 状态
3. **执行工作** → 结果写入 `~/agents/handoffs/` 或 `~/agents/scratch/`
4. **完成后更新 worklog** → 状态改为 done，summary 写明成果
5. **如有需要** → 将可复用知识沉淀到 `~/agents/context/`，将最终报告放入 `~/agents/reports/`

---

## 核心规范

### WORKLOG.md

**必须写**，路径：`~/agents/handoffs/WORKLOG.md`

### runtime_logs

**必须放**，路径：`~/agents/handoffs/runtime_logs/<task-name>_<timestamp>.log`

### Append-only 原则

- 只**追加**新条目，**不修改**旧条目
- 例外：发现自己最新条目有事实错误，可以纠正
- 每次开始持久工作、遇到 blocker、完成或暂停有进展时，都应写 entry

### Entry 格式（每条必含）

```
### <UTC ISO 8601 时间戳> | <actor> | <status>
- Summary: <一句话描述>
- Paths: <涉及的本仓库路径，如有>
- Commands: <执行的命令，如有>
- Artifacts: <生成的产物路径，如有>
- Blockers: <阻塞因素，如有>
- Next: <下一步计划>
```

**status 可选值**：`planned` | `in_progress` | `blocked` | `done` | `cancelled`

### 禁止记录 Secret

token、密码、key 等**只记名称，不记内容**。

---

## log_work.sh

纯 Bash 脚本，核心逻辑：解析参数 → 校验 → append 到 WORKLOG。

```bash
#!/usr/bin/env bash
set -euo pipefail

WORKLOG="${WORKLOG_PATH:-$HOME/agents/handoffs/WORKLOG.md}"
ACTOR="${WORKLOG_ACTOR:-${USER:-unknown}}"

while [[ $# -gt 0 ]]; do
  case "$1" in
    --actor)    ACTOR="$2";    shift 2 ;;
    --status)   STATUS="$2";   shift 2 ;;
    --summary)  SUMMARY="$2";  shift 2 ;;
    --paths)    PATHS="$2";    shift 2 ;;
    --commands) COMMANDS="$2"; shift 2 ;;
    --artifacts) ARTIFACTS="$2"; shift 2 ;;
    --blockers) BLOCKERS="$2"; shift 2 ;;
    --next)     NEXT_STEP="$2"; shift 2 ;;
    -h|--help)  printf 'Usage: log_work.sh [--actor A] [--status S] [--summary T] [--paths P] [--commands C] [--artifacts A] [--blockers B] [--next N]\n'; exit 0 ;;
    *)          printf 'Unknown arg: %s\n' "$1" >&2; exit 1 ;;
  esac
done

case "$STATUS" in
  planned|in_progress|blocked|done|cancelled) ;;
  *) printf 'Invalid --status\n' >&2; exit 1 ;;
esac
[[ -z "$SUMMARY" ]] && printf 'Missing --summary\n' >&2 && exit 1

timestamp="$(date -u +'%Y-%m-%dT%H:%M:%SZ')"
mkdir -p "$(dirname "$WORKLOG")" "$HOME/agents/handoffs/runtime_logs"

cat >> "$WORKLOG" <<EOF

### ${timestamp} | ${ACTOR} | ${STATUS}
- Summary: ${SUMMARY}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chang-xinhai/WxEcho](https://github.com/chang-xinhai/WxEcho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
