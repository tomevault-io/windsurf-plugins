---
trigger: always_on
description: > 本文档为指导 AI AGENT 的开发协作规范，同时也作为 AI AGENT 开发指南和持久化项目记忆存在。
---


# DuckCoding 开发协作规范

> 本文档为指导 AI AGENT 的开发协作规范，同时也作为 AI AGENT 开发指南和持久化项目记忆存在。
> 本文档作为项目记忆文档，需要及时更新。**请务必在开发完成后根据代码的实际情况更新本文档需要修改的地方以反映真实代码情况!!!**

## 核心命令一览

- `npm install`：安装前后端依赖（Node 18+ / npm 9+）。
- `npm run check`：开发工具链主入口，统一调度 AI Agent 配置检查 → ESLint → Clippy → Prettier → cargo fmt，并输出中文摘要。若缺少 `dist/`，会自动尝试 `npm run build` 供 Tauri Clippy 使用。
- `npm run check:fix`：修复版入口，顺序同上，遇可修复项会自动 `--fix`。
- `npm run tauri dev`：本地启动 Tauri 应用进行端到端手动验证。
- `npm run tauri build`: 本地构建 Tauri 应用安装包。
- `npm run test` / `npm run test:rs`：后端 Rust 单测（当前无前端测试，test 等同 test:rs）。
- `cargo test --locked`：Rust 单测执行器；缺乏覆盖时请补测试后再运行。
- `npm run coverage:rs`：后端覆盖率检查（基于 cargo-llvm-cov，默认行覆盖阈值 90%，需先安装 llvm-tools-preview 与 cargo-llvm-cov，可运行 `npm run coverage:rs:setup` 自动安装依赖）。

## 日常开发流程

0. **fork项目**：在 github 找到本项目的上游仓库: <https://github.com/DuckCoding-dev/DuckCoding> 并 fork 最新的 main 分支（后续开发前需确保sync fork以避免冲突），clone 到本地。
1. **创建分支**：`git switch -c feature/<scope>` 或 `refactor/<scope>`，避免多人在同一文件上叠加提交。
2. **编码前**：阅读/更新对应任务的设计文档，确保拆分策略一致，减少冲突。
3. **开发中**：
   - 大改动请模块化提交，保持 main.rs / App.tsx 等中心文件最小改动范围。
   - 随手运行 `npm run check:fix`，保持 0 ESLint/Clippy 告警。
4. **提交前**：
   - 运行 `npm run check`；失败立即`npm run check:fix`尝试自动修复，若无法自动修复则手动修复，禁止忽略告警。
   - 运行 `cargo test --locked` 与必要的端测脚本。
   - 若有必要，更新 `CLAUDE.md`
5. **提交/PR**：
   - commit/pr 需遵循 Conventional Commits 规范，description使用简体中文。
   - pr 描述需包含：动机、主要改动点、测试情况、风险评估。
   - 避免"修复 CI"类模糊描述，明确指出受影响模块。
   - 如有可关闭的 issue，应在 pr 内提及，以便在 merge 后自动关闭。

## 零警告与质量门禁

- ESLint、Clippy、Prettier、`cargo fmt` 必须全部通过，禁止忽略/跳过检查。
- CI 未通过禁止合并；若需临时跳过必须在 PR 中详细说明原因并获 Reviewer 认可。
- 引入第三方依赖需说明用途、体积和维护计划。

## AI 自动阅读文档前提

- `CLAUDE.md` 默认为 Claude-Code 使用
- Codex 使用需要设置 ~/.codex/config.toml 中的
  ```toml
  project_doc_fallback_filenames = ["CLAUDE.md"]
  ```
- Gemini-CLI 使用需要设置 ~/.gemini/settings.json 中的
  ```json
  {
    "context": {
      "fileName": "CLAUDE.md"
    }
  }
  ```
- `npm run check` 会检查这两项配置（仅当检测到对应工具已安装时），未通过显示警告。可用 `npm run check:fix` 自动修复。

## PR 清单

- [ ] 已运行 `npm run check` 且全部通过。
- [ ] Rust/前端测试已运行（或说明尚未覆盖的原因）。
- [ ] 重要变更附测试或验证截图，方便 Reviewer。

## CI / PR 检查

- `.github/workflows/pr-check.yml` 在 pull_request / workflow_dispatch 下运行，矩阵覆盖 ubuntu-22.04、windows-latest、macos-14 (arm64)、macos-13 (x64)，策略 `fail-fast: false`。
- 每个平台执行 `npm ci` → `npm run check`；若首次检查失败，会继续跑 `npm run check:fix` 与复验 `npm run check` 以判断是否可自动修复，但只要初次检查失败，该平台作业仍标红以阻止合并。
- PR 事件下只保留一条自动评论，双语表格固定展示四个平台；未跑完的平台显示"运行中..."，跑完后实时更新结果、check/fix/recheck 状态、run 链接与日志包名（artifact `pr-check-<platform>.zip`，含 `npm run check` / `check:fix` / `recheck` 输出）。文案提示：如首检失败请本地 `npm run check:fix` → `npm run check` 并提交修复；若 fix 仍失败则需本地排查；跨平台差异无法复现可复制日志给 AI 获取排查建议。
- Linux 装 `libwebkit2gtk-4.1-dev`、`libjavascriptcoregtk-4.1-dev`、`patchelf` 等 Tauri v2 依赖；Windows 确保 WebView2 Runtime（先查注册表，winget 安装失败则回退微软官方静默安装包）；Node 20.19.0，Rust stable（含 clippy / rustfmt），启用 npm 与 cargo 缓存。
- CI 未通过不得合并；缺少 dist 时会在 `npm run check` 内自动触发 `npm run build` 以满足 Clippy 输入。

## 架构记忆（2025-12-12）

- **main.rs 模块化重构（2025-12-15）**：
  - **问题**：原 `src-tauri/src/main.rs` 文件过大（652行），包含启动、托盘、窗口、迁移、命令注册等多种职责
  - **解决方案**：按启动流程分层，拆分为 `setup/` 模块
  - **新架构**（位于 `src-tauri/src/setup/`）：
    - `tray.rs` (195行)：托盘菜单创建、窗口管理（显示/隐藏/聚焦/恢复）、事件处理
    - `initialization.rs` (161行)：启动初始化流程（日志/迁移/Profile/代理自启动/工具注册表）
    - `mod.rs` (9行)：模块导出
  - **main.rs 重构**（402行，从 652 行减少 -38%）：
    - 保留：应用启动、状态管理、builder 配置、macOS 事件循环
    - 辅助函数：工作目录设置、配置监听、更新检查调度、单实例判断
    - 命令注册：保留内联（按功能分组注释），避免宏卫生性问题
  - **架构原则**：遵循单一职责原则（SOLID - SRP），按启动流程分层，main() 函数仅保留核心逻辑
  - **代码质量**：所有检查通过（ESLint + Clippy + Prettier + fmt），单元测试 199 通过
- `src-tauri/src/main.rs` 仅保留应用启动与托盘事件注册，所有 Tauri Commands 拆分到 `src-tauri/src/commands/*`，服务实现位于 `services/*`，核心设施放在 `core/*`（HTTP、日志、错误）。
- **配置管理系统（2025-12-12 重构）**：
  - `services/config/` 模块化拆分为 6 个子模块：
    - `types.rs`：共享类型定义（`CodexSettingsPayload`、`ClaudeSettingsPayload`、`GeminiEnvPayload` 等，60行）
    - `utils.rs`：工具函数（`merge_toml_tables` 保留 TOML 注释，85行）
    - `claude.rs`：Claude Code 配置管理（4个公共函数，实现 `ToolConfigManager` trait，177行）
    - `codex.rs`：Codex 配置管理（支持 config.toml + auth.json，保留 TOML 格式，204行）
    - `gemini.rs`：Gemini CLI 配置管理（支持 settings.json + .env 环境变量，199行）
    - `watcher.rs`：外部变更检测 + 文件监听 + 差异分析（合并原 `config_watcher.rs`，~650行）
      - **差异分析（2026-01-07）**：
        - `compute_diff()`：递归比较 JSON 对象，检测字段变更/新增/删除
        - `matches_pattern()`：支持通配符模式匹配（如 `model_providers.*.base_url`）
        - `is_sensitive_change()`：判断变更是否包含敏感字段（API Key、Base URL 等）
        - `filter_blacklisted()`：过滤黑名单字段（如 theme、ui.\*、model 等自动修改字段）
      - **双模式监听**：
        - **默认模式（Default）**：仅通知敏感字段变更（API Key/URL），忽略其他变更
        - **全量模式（Full）**：通知所有非黑名单字段的变更
      - **变更处理**：
        - `mark_external_change()`：增强版，包含完整差异分析和模式判断
        - `block_external_change`：恢复快照到原生配置文件
        - `allow_external_change`：更新快照为当前文件内容
      - 变更检测：`detect_external_changes`、`acknowledge_external_change`
      - Profile 导入：`import_external_change`
      - 文件监听：`ConfigWatcher`（轮询，跨平台）、`NotifyWatcherManager`（notify，高性能）
      - 核心函数：`config_paths`（返回主配置 + 附属文件）、`compute_native_checksum`（SHA256 校验和）
  - 统一接口：`ToolConfigManager` trait 定义标准的 `read_settings`、`save_settings`、`get_schema`
  - 废弃功能：删除 `ConfigService::save_backup` 系列函数（由 `ProfileManager` 替代）
  - 变更检测：与 `ProfileManager` 集成，自动同步激活状态的 dirty 标记和 checksum

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DuckCoding-dev/DuckCoding](https://github.com/DuckCoding-dev/DuckCoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
