---
trigger: always_on
description: 本文件为 AI 编码代理（Claude Code、Codex、Cursor 等）在本仓库工作时的基础指引。
---

# AGENTS.md

本文件为 AI 编码代理（Claude Code、Codex、Cursor 等）在本仓库工作时的基础指引。

## 仓库概览

SuperDuck 是一个浏览器 AI 助手,由多个子项目组成:

| 目录 | 说明 | 语言/构建 |
|---|---|---|
| `chrome-crx/` | Chrome 扩展(Manifest V3),包含 side panel、content script、service worker、MCP runtime | TypeScript + Vite,使用 **Bun** |
| `chrome-native-host/` | Native messaging host + `superduck` CLI + MCP server | Go + Makefile |
| `coworkd/` | Cowork 守护进程 | Go |
| `desktop/` | 桌面端封装 | - |
| `mac-native-addon/` | macOS 原生 Node addon | Swift / C++ |
| `npm/` | npm 分发包 | - |

主要工作区: `chrome-crx`(扩展功能)与 `chrome-native-host`(CLI / 原生桥)。

## Monorepo 工作区

仓库根用两套 workspace 工具把多个子项目绑成一个 monorepo,新增子模块时请同步登记:

- **JS/TS** — 根 `package.json` 声明 `"workspaces": ["chrome-crx", "npm/packages/*"]`,Bun / npm / yarn 任一都能识别。在仓库根 `bun install` 会同时安装所有子包依赖,并把 husky / lint-staged 等开发工具集中放在根 `node_modules/`。
- **Go** — 仓库根 [`go.work`](go.work) 通过 `use` 指令把 `./chrome-native-host` 纳入工作区。在仓库根直接 `go build ./...` / `go test ./...` 就能跨模块编译,IDE / 代理也能基于此识别每个 Go 子项目的 module 边界。新增 Go 模块(例如未来真正落地 `coworkd/`)时,先 `cd <dir> && go mod init <name>`,再在 `go.work` 的 `use (...)` 块中追加路径并提交,不要让多模块项目散落在根 module 之外。
- `go.work.sum` 是本地缓存,**不**提交;`go.work` 是工作区契约,**必须**提交。

## 构建命令

### chrome-crx(Chrome 扩展)

使用 **Bun**,不要用 npm/pnpm/yarn。

```bash
cd chrome-crx
bun install          # 安装依赖
bun run build        # 生产构建 → dist/
bun run dev          # 监听模式
bun run typecheck    # tsc --noEmit
bun run lint         # eslint
bun run format       # prettier
bun run test         # vitest run (单元 + 集成)
bun run test:coverage # vitest 覆盖率 + 阈值门禁 (lines/statements/branches ≥ 90%, functions ≥ 55%)
bun run test:perf    # 运行测试并写出 JUnit XML 报告 (test-results.junit.xml)
bun run test:perf:report # 解析 JUnit 报告,打印每个 suite 耗时、慢测试 (>= 300ms) 与 Top10
# CI=1 bun run test  # 额外输出 verbose 计时 + JUnit 报告 (test-results.junit.xml),
#                    # 用于 BuildPulse / Datadog CI / GitHub test-reporter 跟踪测试耗时
```

构建产物在 `chrome-crx/dist/`,加载扩展时指向该目录。
**修改任何 `chrome-crx/src/**` 下的源码后都要立即 `bun run build` 验证。**

### chrome-native-host(Go)

```bash
cd chrome-native-host
make              # 构建 native-host / mcp-server / superduck 三个二进制到 build/
make superduck    # 只构建 CLI
make test         # 运行 go test ./...
make test-coverage # 覆盖率门禁(默认 ≥ 40%, 见 COVERAGE_PACKAGES / MIN_COVERAGE 变量)
make test-perf    # 运行测试并输出 per-test 耗时 + test-results.json,
                  # 报告慢测试 (>= SLOW_TEST_MS, 默认 500ms) 与 Top10 最慢用例
make lint         # 运行 golangci-lint(配置见 .golangci.yml)
make lint-install # 首次使用前安装 pinned 版本的 golangci-lint
```

构建完成后,`chrome-native-host/superduck` 通常是指向 `build/superduck` 的软链;如果丢失用 `ln -sf build/superduck superduck` 重建。

### Pre-commit Hooks (husky + lint-staged)

仓库根目录配置了 husky + lint-staged,首次 clone 后请在仓库根执行:

```bash
bun install      # 或 npm install — 触发 husky `prepare` 脚本,在 .git 中注册钩子
```

之后每次 `git commit` 时会自动:

- 对暂存的 `chrome-crx/src/**/*.{ts,tsx,js,json}` 跑 `prettier --write` 与 `eslint --fix`
- 对暂存的 `chrome-native-host/**/*.go` / `coworkd/**/*.go` 跑 `gofmt -w` 与 `go vet ./...`

钩子失败时提交会被中止;修复后重新 `git add` 再 `git commit`。如需绕过(不推荐),可加 `--no-verify`。

## 依赖最小发布年龄 (min-release-age)

为缓解供应链攻击,所有引入或升级到刚发布的依赖版本必须先经过冷却期:

- **政策**:任何 `package.json` / `go.mod` 中的版本必须**至少发布 3 天**后才允许进入主干。Major 升级建议延长到 7 天。
- **Dependabot**:[`.github/dependabot.yml`](.github/dependabot.yml) 的 `cooldown` 块强制 Dependabot 不在窗口内开 PR(default 3d / minor 5d / major 7d)。
- **CI 校验**:[`.github/workflows/min-release-age.yml`](.github/workflows/min-release-age.yml) 在每个修改 `package.json` / `go.mod` 的 PR 上运行 [`scripts/check-min-release-age.mjs`](scripts/check-min-release-age.mjs),向 npm/Go module proxy 查询每个新版本的发布时间并阻止 < 3 天的引入。
- **临时豁免**:确实需要紧急升级(例如 CVE 修复)时,在 PR 描述中说明,然后用 `workflow_dispatch` 重新触发该 workflow 并把 `days` 调低,或获得 reviewer 显式批准后再合并。

## API 文档自动生成

- TypeScript 侧用 [TypeDoc](https://typedoc.org) 生成 `chrome-crx/docs/api/`,本地运行 `bun run docs`(配置见 [`chrome-crx/typedoc.json`](chrome-crx/typedoc.json)),`bun run docs:check` 只校验不写盘
- Go 侧由 CI 通过 `go doc -all <pkg>` 给每个包生成纯文本快照,便于 agent 检索
- [`.github/workflows/docs.yml`](.github/workflows/docs.yml) 在每次 PR 上构建文档,合并到 `main` 后将 TypeDoc 站点 + godoc 快照发布到 GitHub Pages

## Dev Container

仓库根目录提供 [`.devcontainer/`](.devcontainer/devcontainer.json),含 Node + Bun + Go 1.25 + gh 与必要的 VS Code 扩展。在 Codespaces 或本地 Dev Containers 中打开即可获得统一环境,`postCreateCommand` 会自动运行 [`post-create.sh`](.devcontainer/post-create.sh) 完成依赖拉取。

## AGENTS.md 自验

仓库根的 [`scripts/validate-agents-md.mjs`](scripts/validate-agents-md.mjs) 会校验本文件的 `bun run <script>` 引用都存在于 `chrome-crx/package.json`、`make <target>` 引用都存在于 `chrome-native-host/Makefile`、所有相对链接都能解析到真实文件。CI 工作流 [`.github/workflows/validate-agents-md.yml`](.github/workflows/validate-agents-md.yml) 在每次涉及 `AGENTS.md` / `package.json` / `Makefile` 的 PR 上运行该脚本。本地可直接执行:

```bash
node scripts/validate-agents-md.mjs
```

## 环境变量

仓库根目录的 [`.env.example`](.env.example) 列出了常用的环境变量及默认值/示例;复制为 `.env` 后按需填写。简要分组:

| 变量 | 作用域 | 说明 |
|---|---|---|
| `SUPERDUCK_POSTHOG_KEY` | chrome-native-host | PostHog write key,留空则不发送埋点 |
| `SUPERDUCK_POSTHOG_HOST` | chrome-native-host | 自定义 PostHog 域名,默认 `https://us.i.posthog.com` |
| `SUPERDUCK_ANALYTICS_DISABLED` | chrome-native-host | `1/true` 强制关闭埋点 |
| `SUPERDUCK_SENTRY_DSN` | chrome-native-host | Sentry/GlitchTip DSN,留空则不上报错误 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superduck-ai/superduck](https://github.com/superduck-ai/superduck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
