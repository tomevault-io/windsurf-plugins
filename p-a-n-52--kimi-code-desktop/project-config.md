---
trigger: always_on
description: 本仓库是 Kimi Code CLI 的独立 Windows 桌面外壳，不是 CLI 源码树。React/Tauri 负责桌面体验和进程编排；AI 会话、模型、工具及运行时行为由用户安装的 Kimi Code CLI 通过 `kimi acp` 提供。配置、MCP、会话文件和 Git 信息由 Rust 本地辅助模块读写 `~/.kimi-code` 与当前工作区。
---

# Kimi Code Desktop 代理指南 / Agent Guide

## 项目定位 / Project Role

本仓库是 Kimi Code CLI 的独立 Windows 桌面外壳，不是 CLI 源码树。React/Tauri 负责桌面体验和进程编排；AI 会话、模型、工具及运行时行为由用户安装的 Kimi Code CLI 通过 `kimi acp` 提供。配置、MCP、会话文件和 Git 信息由 Rust 本地辅助模块读写 `~/.kimi-code` 与当前工作区。

English: This repository is an independent Windows desktop shell for Kimi Code CLI, not the CLI source tree. React/Tauri owns the desktop UX and process orchestration; the user-installed Kimi Code CLI provides sessions, models, tools, and runtime behavior through `kimi acp`. Rust helpers handle config, MCP, session files, and Git data in `~/.kimi-code` and the active workspace.

当前事实来源按优先级为：正在运行的源码和测试、`package.json` 脚本、`.github/DEVELOPMENT.md`、本文件、`README.md`、`docs/plans/`。不要引用已删除的 `docs/DEVELOPMENT_STANDARD.md`、`docs/RELEASE.md` 或 `docs/acp-contract.md`。

English: Sources of truth, in order, are the running source and tests, `package.json` scripts, `.github/DEVELOPMENT.md`, this file, `README.md`, and `docs/plans/`. Do not reference the removed `docs/DEVELOPMENT_STANDARD.md`, `docs/RELEASE.md`, or `docs/acp-contract.md`.

开发规范见 `.github/DEVELOPMENT.md`，实现前先阅读。 / Read `.github/DEVELOPMENT.md` before implementation.

## 当前进度（2026-07-20）/ Current Progress

### 已提交基线 / Committed Baseline

- Monochrome V2 视觉系统、AppShell、会话侧栏、消息流、Composer、Changes 面板、设置页和快捷键已在 2026-07-18 的 V2 提交序列中落地。
- 当前 `master` 比 `origin/master` 超前；不要把“尚未推送”误判成“尚未实现”。

English:

- The Monochrome V2 visual system, AppShell, session sidebar, message stream, composer, Changes panel, settings shell, and shortcuts landed in the 2026-07-18 V2 commit series.
- Local `master` is ahead of `origin/master`; do not confuse “not pushed” with “not implemented.”

### 工作区内已实现、仍待收口 / Implemented In The Worktree, Pending Integration

- 后端已迁移到 ACP-only：`AcpProcessManager` 负责会话 wire 流，`AcpDesktopClient` 负责会话 RPC；Python sidecar、`sidecar.rs` 和 bundled `kimi-sidecar` 正在删除。
- V2 已重新接入单一活动 `useSessionStream`、历史回放、附件、状态消息、工具 display blocks、子代理步骤，以及通用未知 payload fallback。
- Workspace 已接入 Changes、Files、Agents、Tasks；Composer 已接入 slash 命令、文件上传、忙碌时队列和全局模型显示。
- `/usage` / `/status` 由桌面本地拉取平台额度（5h / 7d）；未知斜杠指令会拦截提示。详见 `docs/SLASH_COMMAND_PARITY.md`。
- 会话侧栏已接入 active/archived 分页、归档/恢复、标题生成、批量归档/恢复/删除；设置已接入 dark/light 主题、全局配置、原始 `config.toml` 和 MCP。
- 发送反馈已具备通用状态：立即显示“消息发送中”，首个可见响应后移除；空终态或 ACP 错误显示持久错误。

English:

- The backend worktree is ACP-only: `AcpProcessManager` owns session wire streams and `AcpDesktopClient` owns session RPC. The Python sidecar, `sidecar.rs`, and bundled `kimi-sidecar` are being removed.
- V2 reconnects a single active `useSessionStream`, replay, attachments, status messages, tool display blocks, subagent steps, and a generic fallback for unknown payloads.
- Workspace exposes Changes, Files, Agents, and Tasks. Composer exposes slash commands, uploads, busy-state queueing, and the global model label.
- `/usage` / `/status` are handled locally with platform quotas (5h / 7d); unknown slash commands are blocked with a desktop hint. See `docs/SLASH_COMMAND_PARITY.md`.
- Sessions expose active/archived pagination, archive/restore, title generation, and bulk archive/restore/delete. Settings expose dark/light theme, global config, raw `config.toml`, and MCP.
- Generic send feedback shows “消息发送中” immediately, clears on the first visible response, and preserves empty-terminal or ACP failures as visible errors.

这些内容横跨 staged、unstaged 和 untracked 文件。除非完整验证通过并完成提交，否则在交接中称为“工作区已实现”或“集成中”，不要称为稳定发布能力。

English: This work spans staged, unstaged, and untracked files. Until full verification and commit are complete, describe it as “implemented in the worktree” or “in integration,” not as a stable release capability.

### 尚未完成的验收 / Remaining Acceptance

- 按 `docs/plans/2026-07-18-webview2-acceptance.md` 在真实 Tauri + 已认证 `kimi acp` 路径上验证会话、prompt、工具、Workspace 和 Settings；浏览器 mock 不等于桌面验收。
- 完成 `docs/plans/2026-07-18-v2-ui-integration.md` 的剩余差距审计，特别检查桌面完成通知和所有真实运行时入口；不要仅按文件存在判断完成。`fork_session` 当前因 ACP 不支持而明确返回错误，不得伪造 fork-at-turn UI 或静默改走旧 API。
- 补齐 Settings、Sessions sidebar 和 Workspace 的集成测试；system theme 尚未接入，不能把 dark/light 切换描述为完整的三态主题支持。
- Share 没有真实后端契约时应保持移除或禁用，不要制作假入口。
- 发布脚本和 MSI 只有在 `release:preflight` / `release:msi` 通过后才能声明可发布。

English:

- Follow `docs/plans/2026-07-18-webview2-acceptance.md` against real Tauri plus an authenticated `kimi acp`. Browser mocks are not desktop acceptance.
- Audit remaining gaps in `docs/plans/2026-07-18-v2-ui-integration.md`, especially desktop completion notifications and all real runtime entry points. `fork_session` currently returns an explicit error because ACP does not support it; do not fake fork-at-turn UI or silently route it through a legacy API.
- Add integration coverage for Settings, Sessions sidebar, and Workspace. System theme is not wired yet, so do not describe the dark/light toggle as complete three-state theme support.
- Keep Share removed or disabled until a real backend contract exists.
- Do not claim release readiness until `release:preflight` and, when relevant, `release:msi` pass.

## 运行链路 / Runtime Chain

桌面应用为 **ACP-only**；不要恢复 Python sidecar 或新增 legacy runtime fallback。

```text
React app shell / useSessionStream

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [P-A-N-52/kimi-code-desktop](https://github.com/P-A-N-52/kimi-code-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
