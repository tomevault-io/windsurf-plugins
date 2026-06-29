---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project Overview

PaperAgent 是一个 Go + Web UI 的 AI 论文阅读助手，已经从「单功能 Q&A」演化为「**Q&A + 每日推荐**」双系统产品。中文 UI / 中文文档。

- **Q&A 系统**：用户给一篇论文（URL / arXiv ID / 粘贴），AI 生成详尽摘要后进入多轮问答
- **每日推荐**：定时从 arXiv RSS 拉取新论文，LLM 按用户偏好打分，每天生成一批推荐并可推送到飞书

两个系统共享：API 配置、Feishu 集成、配置与提示词覆盖。

## Build & Run

```bash
# 推荐：用 justfile（开发 / 构建 / 静态分析都封装好了）
just --list                    # 查看所有 recipe
just dev                       # 开发模式：Go + Vite HMR 并发
just build                     # 完整构建（前端 + Go），产出单二进制
just arxiv2md                  # 编译独立 arxiv2md 工具
just vet                       # Go vet
just typecheck                 # 前端 tsc --noEmit

# 不使用 just 的等价命令
cd frontend && npm install && npm run build           # 前端 → internal/server/frontend-dist/
GOCACHE=/tmp/gocache-$USER go build -o paperagent .  # 内嵌前端打二进制
GOCACHE=/tmp/gocache-$USER go build -o arxiv2md ./cmd/arxiv2md/

# 首次运行
./paperagent                   # 启动 HTTP server（默认 8686）+ 系统托盘
# 或开发模式：PAPER_NO_BROWSER=1 PAPER_FOREGROUND=1  ./paperagent
# 浏览器访问 http://localhost:8686（或 5173 HMR 模式）

# 首次启动若 ~/.config/paperagent/config.yaml 不存在，
# Web UI 会自动弹出设置对话框引导用户配置 API 密钥。
```

### 二进制与端点

- 主二进制 `paperagent`：HTTP server + 系统托盘 + 飞书机器人
- 独立二进制 `arxiv2md`：`arxiv:ID` / `arxiv.org/abs/...` / `arxiv.org/pdf/...` → 干净 Markdown（HTML 优先，TeX 兜底）
- Chrome 扩展 `extension/`：在 arXiv 论文页右侧栏注入「在 PaperAgent 中打开」按钮

### 进程模型

- `main.go`：解析 `-version`、加载 config、检测 API key、建目录、`daemonize()`、起托盘
- `main_unix.go`（非 Windows）：默认调用 `setsid` 把自己 daemonize 成后台进程；`PAPER_FOREGROUND=1` 跳过（dev 模式）
- `main_windows.go`：空操作（托盘管理生命周期，不需要 fork）
- HTTP 端口：`PAPER_ADDR=:8686` 可改；启动时自动扫描 8686～8785 找可用端口
- 浏览器自动打开：默认开；`PAPER_NO_BROWSER=1` 关闭

## Testing

```bash
# 全部测试（部分会打真实 API）
go test ./... -v

# 单元测试（不打 API），CI 也只跑这批
go test ./internal/config/ ./internal/session/ ./internal/chat/ \
        ./internal/prompt/ ./internal/urlparse/ ./internal/export/ \
        ./internal/database/ ./internal/recommend/ -v

# Lint
go vet ./...
```

测试覆盖：config + crypto（含 AES 解密旧 key 兼容）、session、chat（fakeLLM 驱动 Engine+Sink 测试）、prompt、urlparse、export、database（含 SQL 池钩子）、recommend（feed / scoring / algorithm 单测 + e2e）、feishu（latex2unicode / 卡片尺寸 / 卡片模板）、api 客户端。

`internal/recommend/e2e_test.go` 是端到端冒烟：通过 `PAPER_RECOMMEND_RSS_FILE` 环境变量把 RSS XML 替换成本地文件，跑完整推荐管线，不打网络。

## Release

通过 GitHub Actions（`.github/workflows/release.yml`，`workflow_dispatch`）发布。**不要** 手动 `git tag` / `gh release create`。

CI（`.github/workflows/ci.yml`）会在 PR 上跑 `npm ci && npm run build`、`go vet ./...`、单元测试、`go build`。

### 步骤

1. **检查自上次 release 之后的 commits**

   ```bash
   git log v{last_tag}..HEAD --format="%h %s" --reverse
   ```

2. **检查实际 diff**（注意 rebase 后的 commit —— 看日期而非 hash）

   ```bash
   git diff --stat v{last_tag}..HEAD
   ```

3. **触发 release workflow**

   ```bash
   gh workflow run release.yml -f version=v1.2.0 -f release_notes='你的 release notes markdown'
   ```

   推荐**总是传** `version` 明确指定版本，避免 auto-bump 受脏 tag 影响。
   确认没有脏 tag 也可以只用 `bump`：`-f bump=patch`。
   `release_notes` 传空字符串时 action 会自动从 commits 生成。

   > ⚠️ 绝对不要手动 `gh release create` 或 `git tag` + `git push`，会留下脏 tag 干扰 auto-bump。

4. **等 action 完成**（https://github.com/happyTonakai/PaperAgent/actions）

   流水线：`prepare`（算版本号 + 编译前端，Node 堆 4GB）→ `build`（三平台：macOS arm64 / Windows amd64 / Linux amd64，macOS ad-hoc 签名）→ `release`（打 tag + 创建 Release + 上传二进制）。

## Architecture

**Tech stack**：Go 1.25.8、React 19 + TypeScript + Vite 6 + Tailwind 4、YAML 配置、JSON 持久化、SQLite（`modernc.org/sqlite`，纯 Go 零 CGO）、`fyne.io/systray`、`larksuite/oapi-sdk-go/v3`、KaTeX / highlight.js / react-markdown。

**核心设计原则（Q&A 系统）**：论文全文始终在 LLM 上下文（L1），多轮 Q&A 用锚点 + 动态截断：上下文窗口从 `TruncationAnchor` 开始累计 token，超过 `max_input_tokens`（默认 30000）后硬截断到 `min_recent_rounds`（默认 2）。截断后前缀稳定，KV cache 友好。初始摘要**不进入**后续对话上下文。`/btw` 旁听消息（`SkipContext: true`）从上下文窗口排除。

**双系统架构**：

```
            ┌──────────────────┐
            │   Web UI (React) │
            └────────┬─────────┘
                     │ HTTP / SSE
            ┌────────▼─────────┐
            │   HTTP Server    │  internal/server/
            │  (embed.FS SPA)  │  + handlers_recommend.go
            └────┬──────┬──────┘
                 │      │
   ┌─────────────┘      └──────────────┐
   ▼                                  ▼
┌──────────────┐               ┌──────────────┐
│  Q&A System  │               │  Recommend   │
│  session/    │               │  recommend/  │
│  api/        │               │  database/   │
│  chat/       │               │  scheduler/  │
│  prompt/     │               │              │
│  urlparse/   │               │              │
└──────┬───────┘               └──────┬───────┘
       │                              │
       └──────────┬───────────────────┘
                  ▼
          ┌──────────────┐
          │  config/     │
          │  export/     │
          │  feishu/     │
          │  systray/    │
          └──────────────┘
```

### Q&A 二阶段状态机

- **INIT**：`system.txt` + 论文全文 + `heavy.txt` 任务提示 → API → SSE 流式输出 Markdown 摘要；title 从 arXiv HTML 解析
- **CHAT**：`chat.Engine` 构建消息（`system.txt` + 论文全文 + `light.txt` + 动态上下文）→ LLM 可调用工具（`fetch_arxiv` 获取另一篇论文做交叉对比、`get_references` 查看参考文献）→ 流式回复经 `sseSink` / `cardSink` 输出


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [happyTonakai/PaperAgent](https://github.com/happyTonakai/PaperAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
