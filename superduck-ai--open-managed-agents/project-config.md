---
trigger: always_on
description: - 在仓库根目录使用 `just restart-server` 重启本地后端服务，地址为 `127.0.0.1:38080`。
---

# Open Managed Agents

## 本地重启脚本

- 在仓库根目录使用 `just restart-server` 重启本地后端服务，地址为 `127.0.0.1:38080`。
- `just restart-server` 会调用 `./scripts/restart-server.sh`，杀掉所有监听 `PORT`（默认 `38080`）的进程，等待端口释放，必要时升级为 `kill -9`，然后以前台方式执行 `go run .`；监听地址由 `config/config.yaml` 的 `server.addr` 决定。
- 仅在 `server.addr` 已改为其他端口时，才使用 `PORT=... just restart-server` 指定需要释放的对应端口。
- 如果修改了 `web/` 下的前端代码，在使用浏览器或 SuperDuck 验证前，也要从仓库根目录执行 `just restart-web` 重启前端开发服务器。该命令会调用 `./scripts/restart-web.sh`，只停止当前仓库路径启动的 Vite 监听进程；如果目标端口被其他路径的进程占用，则保留该进程并自动选择后续可用端口以前台方式启动前端。

## GitHub PR 提交身份

- 本仓库的 Pull Request 必须通过本机已认证的 `gh` CLI 创建；禁止使用 Codex GitHub Connector 或其 GitHub App 创建 PR。
- 创建 PR 前必须依次运行 `gh auth status` 和 `gh api user --jq .login`，记录当前登录账号。任意已认证账号均可使用；仅当用户明确指定了账号且当前登录账号不匹配时，才应停止并请求用户处理。
- 分支、提交和推送仍使用本地 `git`；分支推送成功后，使用 `gh pr create --draft ...` 创建 Draft PR，并通过 `gh pr view --json author,url,isDraft` 确认 PR 作者与创建前记录的登录账号一致且状态为 Draft。

## 提交前质量门禁

- 首次 clone 仓库或发现 hook 尚未安装时运行 `just hooks-install`，为当前 Git 仓库安装受管的 pre-commit hook；同一 clone 下的 worktree 共用该 hook。缺少 `pre-commit` 时，脚本会优先通过 `uv` 安装固定版本。
- hook 对暂存文件执行通用文件卫生检查，对 Go 文件执行 `gofmt`、对应 package 的 golangci-lint、不可达声明检测、重复代码检测和生产代码复杂度检查，并用项目固定版本的 Prettier 格式化前端文件，同时检查 TypeScript 重复代码、命名与复杂度。
- 使用 `just hooks-run` 对全部跟踪文件复跑相同检查；不要使用 `SKIP` 绕过失败项，除非用户明确批准并记录原因。
- 使用 `just large-files` 通过仓库固定版本的 `check-added-large-files --enforce-all` 检查全部受跟踪文件，统一上限为 1 MiB。嵌入式目录快照 `internal/platformapi/directory_servers.json` 必须保存为紧凑 JSON。不要通过改名、忽略路径或提高预算绕过失败；有意引入大二进制文件时，应单独评审 Git LFS 策略。

## 死代码检测

- 修改 Go 代码后运行 `just dead-code`。`.golangci-dead-code.yml` 使用 `unused` 分析器检查生产代码和测试中的不可达包级函数、方法、变量、常量与类型。
- pre-commit 和 `.github/workflows/dead-code.yml` 使用同一配置；不要通过 `nolint`、全局排除、伪造引用或保留无调用路径的兼容包装来绕过失败。确认不再可达的声明及其专属辅助链应直接删除。

## 重复代码预算

- 修改 Go 或 TypeScript/TSX 生产代码后运行 `just duplicates`。项目固定的 jscpd 以 strict token 模式检测至少 12 行且至少 70 token 的复制代码；Go 与前端分别执行，避免一个应用较低的比例掩盖另一个应用的增长。
- `.jscpd.json` 将 Go 生产代码重复率限制为 3.75%，`web/.jscpd.json` 将前端生产代码重复率限制为 1.1%。测试、suite 和生成文件不计入生产预算；不要通过扩大 ignore、提高百分比、提高最小行数/token 数或拆成近似副本绕过门禁。
- pre-commit 和 `.github/workflows/duplicate-code.yml` 使用 `scripts/check-duplicates.sh` 执行相同门禁。预算失败时优先抽取领域辅助函数、共享数据映射或展示组件；只有确实共享同一语义与演进方向的代码才应合并。

## 复杂度预算

- 修改 Go 或 TypeScript/TSX 生产代码后运行 `just complexity`。Go 使用 `cyclop`，单函数最大圈复杂度为 30；测试文件不计入生产代码复杂度指标。
- 前端使用 ESLint modified cyclomatic complexity，新代码上限为 20。`web/eslint.complexity.config.js` 中列出的历史热点以当前测量上限作为 ratchet，修改这些文件时不得提高预算，并应优先通过拆分纯函数、数据映射或展示组件降低预算。
- 不要通过 `nolint`、ESLint disable 注释、忽略新增生产文件或提高复杂度阈值来绕过失败。确需调整预算时，必须同时说明无法拆分的边界原因，并更新 `docs/design/development-complexity-guardrails.md`。
- pre-commit 和 `.github/workflows/complexity.yml` 都调用仓库固定的复杂度配置；本地验收入口为 `just complexity`。

## 命名规范

- Go package 名使用简短的小写单词；导出类型、函数和方法使用 PascalCase，未导出标识符使用 mixedCaps。缩写保持 Go 惯例并在同一标识符中一致，例如 `API`、`HTTP`、`ID`、`URL`、`UUID`；接收器名应简短且在同一类型的方法中一致。
- TypeScript/React 的类型、接口、类和组件使用 PascalCase；普通变量、函数和参数使用 camelCase；模块级常量可使用 UPPER_CASE；泛型类型参数使用 PascalCase。以 PascalCase 命名的函数参数只用于组件或构造器引用等可调用类型。
- Anthropic/API、数据库和第三方 payload 的字段名属于外部合同，可在边界 DTO、对象属性和解构中保留 `snake_case`；进入内部变量或业务模型后应映射为上述语言惯例，不要把例外扩散到业务标识符。
- Go 命名由 `.golangci.yml` 中 `revive/var-naming` 强制；前端命名由 `bun run lint:naming` 强制，并在 pre-commit 与 `.github/workflows/web-naming.yml` 中执行。

## TrimSpace 使用规范

- `strings.TrimSpace` 和 `bytes.TrimSpace` 只允许用在确实可能携带首尾空白的外部输入边界，例如用户提交的请求字段、CLI 参数、环境变量、配置文件值和第三方 payload；这类值应在进入内部模型前修剪一次。
- 不要对内部生成或已校验的值做防御性修剪：常量、代码拼接的标识符、UUID、enum、数据库行扫描结果，以及上游已经修剪过的值都不需要再 TrimSpace。
- 同一条数据流上不得层层重复 TrimSpace；修剪责任属于最早接触不可信输入的 HTTP/resource/service 边界，下游代码应假设值已清洁。
- 不要用 TrimSpace 掩盖 bug：如果内部 ID、路径或协议字段出现意外空白，应在产生它的位置修复，而不是在每个消费点静默吞掉。

## JSON 与 schema 边界

- `json.RawMessage` 只用于数据库 JSON/JSONB、HTTP/第三方 payload、延迟解析和未知字段透传等序列化边界。业务逻辑一旦需要读取其中字段，应在边界附近解析为命名 schema/DTO，再映射为内部领域类型。
- 不要让 `json.RawMessage`、`map[string]any` 或 `[]any` 作为内部业务模型跨 package 扩散，也不要用它们规避已知字段的 schema 定义。
- 需要保留未知 JSON 字段时，可以在边界使用 `map[string]json.RawMessage` 作为 envelope，但已知字段仍必须通过命名 schema 解析和校验。
- DB 层可以返回原始 JSONB 值，但不承担 HTTP/DTO 解析或领域策略；调用方应在 resource/service/policy 边界尽早完成结构化转换。

## Go 日志规范

- 生产代码统一使用标准库 `log/slog` 作为日志 API；不要直接使用标准库 `log`、`fmt.Printf` 式日志、logrus、zap、zerolog 或自建 printf 包装器。面向终端用户的 CLI 输出不属于运行日志，可以继续写入 stdout/stderr。
- logger 和 handler 统一在可执行程序的组装层通过 `internal/logging` 创建，并在启动早期调用 `slog.SetDefault`。根 logger 通过 `ServerDeps`、资源构造函数或 worker 构造函数显式注入；组装层用 `logger.With("component", "...")` 创建组件 logger，业务方法使用自身持有的 logger。`config.Config` 只承载可序列化的业务/部署数据，不得包含 `*slog.Logger` 或其他运行时依赖。构造边界使用 `logging.LoggerOrDefault` 统一兼容 nil logger，组件内部不得重复读取 `slog.Default()`；生产组装必须显式传入 logger。稳定的 DB、配置和 logger 依赖应由 Handler、Service、Enqueuer 或 Worker 持有，不要在每次业务调用中重复透传；纯解析、转换和 I/O helper 优先返回结果与 error，由拥有方决定是否记录。不要为了 logger 注入把静态 logger 塞入请求 context；只有基于 request ID、trace 等请求域数据派生的 request-scoped logger 才适合随请求传递。业务包不要各自创建 handler，也不要在启动完成后修改全局默认 logger。
- 日志采用结构化字段：消息使用稳定、简短的事件描述，动态值放入属性；新增属性名使用 `snake_case`。错误统一放在 `error` 字段，资源标识使用 `request_id`、`organization_id`、`workspace_id`、`session_id` 等明确字段，不要用 `fmt.Sprintf` 或格式化占位符拼接日志。
- 已持有 `context.Context` 的请求、worker 和后台任务路径优先使用 `DebugContext`、`InfoContext`、`WarnContext`、`ErrorContext`，以便 handler 关联请求 ID、trace 和调用域字段。
- 级别语义保持一致：`Debug` 用于默认关闭的诊断细节，`Info` 用于正常生命周期与重要状态变化，`Warn` 用于可恢复降级、预期拒绝或需要关注的异常输入，`Error` 用于操作未能完成的非预期故障。同一错误只在负责最终处理或补充关键边界信息的层记录一次。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superduck-ai/open-managed-agents](https://github.com/superduck-ai/open-managed-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
