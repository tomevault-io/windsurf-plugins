---
trigger: always_on
description: VoxFlow（码上写）是一款原生 macOS 菜单栏语音输入工具。按住快捷键说话，松开后文字回到当前光标所在位置。中文显示名"码上写"，英文品牌名"VoxFlow"。
---

# VoxFlow — AGENTS.md

## 项目概览

VoxFlow（码上写）是一款原生 macOS 菜单栏语音输入工具。按住快捷键说话，松开后文字回到当前光标所在位置。中文显示名"码上写"，英文品牌名"VoxFlow"。

它的定位是"语音键盘"，不是语音助手：不接管窗口，不自动发送内容，不把用户带到另一个输入框。核心体验围绕全局听写、稳定文本插入、本地优先数据、可选 LLM 保守纠错，以及多 ASR Provider（Apple Speech、Qwen3-ASR、Whisper、FunASR、SenseVoice 等）展开。

技术栈：Swift 6 + SwiftUI/AppKit + SwiftPM，最低支持 macOS 15。主要依赖包括 FluidAudio、WhisperKit/argmax-oss-swift、Sherpa-ONNX vendor runtime、Qwen3 MLX worker/托管 Python runtime 相关脚本，以及 `agent-cli/` 下的 Rust AI Coding 助手 helper/router。

## 构建与运行

| 命令 | 用途 |
|---|---|
| `make build` | Release 构建 + 打包 `.app`（Universal Binary，arm64 + x86_64） |
| `make run-dev` | prelaunch-cleanup → Debug 本机架构构建 → 打包并启动 app（日常开发优先用这个） |
| `make build-dev` | Debug 本机架构构建 + 打包 `.app`，不启动 |
| `make run-native` | prelaunch-cleanup → Release 本机架构构建 → 打包并启动 app |
| `make build-native` | Release 本机架构构建 + 打包 `.app`，不启动 |
| `make run` | prelaunch-cleanup → Universal Release build → 启动 app（发布前或兼容性验证用） |
| `make debug` | Debug 构建，开启 `-warnings-as-errors` |
| `make test` / `swift test` | 运行全部测试 |
| `make install` | 安装到 `/Applications/VoxFlow.app` |
| `make dmg` | 生成 DMG 安装包 |
| `make clean` | 清理构建产物 |

**不要用 `swift run` 代替 `make run-dev` / `make run`**——权限、签名、资源加载、worker 打包、LaunchServices 注册和状态栏缓存清理行为不同。

### 菜单栏图标 / StatusKit 缓存排查

当菜单栏图标消失、错位，且改 Bundle ID 后暂时恢复时，优先按 macOS 状态栏缓存问题处理，不要只改图标资源或反复 bump Bundle ID。

已知相关缓存层：

- App defaults：`~/Library/Preferences/com.voxflow.app*.plist` 中的 `NSStatusItem Preferred Position ...`、`NSStatusItem Visible ...`、`NSStatusItem VisibleCC ...`
- LaunchServices 注册库：由 `lsregister` 管理，可能残留 `.build/`、`/Applications/`、`~/.Trash/`、已挂载 DMG 中的旧 `VoxFlow.app` / `VoiceInput.app`
- IconServices 图标缓存：`$(getconf DARWIN_USER_CACHE_DIR)/com.apple.iconservices*`，以及系统级 `/Library/Caches/com.apple.iconservices.store`
- StatusKit / Control Center 私有状态：`~/Library/StatusKit`、`~/Library/Group Containers/group.com.apple.controlcenter`

常规清理优先使用 `make run-dev` 或 `make run`，它会执行 `prelaunch-cleanup`，覆盖本项目已知的 LaunchServices 反注册和 status item defaults 清理。如果仍不恢复，再做深度清理：

```bash
pkill -x VoxFlow 2>/dev/null || true
killall ControlCenter 2>/dev/null || true
killall SystemUIServer 2>/dev/null || true
killall iconservicesagent 2>/dev/null || true

rm -rf "$(getconf DARWIN_USER_CACHE_DIR)/com.apple.iconservices"
rm -rf "$(getconf DARWIN_USER_CACHE_DIR)/com.apple.iconservicesagent"

killall cfprefsd 2>/dev/null || true
killall ControlCenter 2>/dev/null || true
killall SystemUIServer 2>/dev/null || true
killall iconservicesagent 2>/dev/null || true
```

如果 `~/Library/StatusKit` 或 `~/Library/Group Containers/group.com.apple.controlcenter` 读写时报 `Operation not permitted`，这是 TCC 隐私保护，不是普通 Unix 权限。先在 System Settings → Privacy & Security → Full Disk Access 给当前终端 / Codex / iTerm / Ghostty 授权，并完全重启该终端。授权后才可清理：

```bash
pkill -x VoxFlow 2>/dev/null || true
killall ControlCenter 2>/dev/null || true
killall SystemUIServer 2>/dev/null || true

rm -rf "$HOME/Library/StatusKit"
rm -rf "$HOME/Library/Group Containers/group.com.apple.controlcenter"

killall cfprefsd 2>/dev/null || true
killall ControlCenter 2>/dev/null || true
killall SystemUIServer 2>/dev/null || true
```

注意：清理 `StatusKit` / `group.com.apple.controlcenter` 会重置部分 macOS 菜单栏和控制中心布局，只在普通 `prelaunch-cleanup`、LaunchServices 重建、IconServices 用户缓存清理都无效时使用。不要删除 `~/Library/Application Support/VoxFlow/`，那里是用户数据。

## 验证清单

完成高风险或发布前改动时，按顺序执行：

1. `swift test` — 全部测试通过（0 unexpected failures）
2. `make debug` — Debug 构建无 warning（`-warnings-as-errors`）
3. `make build` — Release 构建通过
4. 重要行为改动遵循 TDD：先写失败测试 → 最小实现 → 重构

如果全量门禁被当前工作树中无关迁移问题阻塞，必须明确报告具体命令、错误文件/行号、是否与本次改动相关，并至少完成本次改动的针对性测试或静态检查。

小 bug 修复不强制新增测试，也不要求默认跑完整门禁；根据风险选择最小验证即可，例如静态检查、局部构建、手工检查或复现路径验证。典型小 bug 包括提示文案、toast 可见性、轻量布局微调、明显的一行逻辑修正等。不要为了满足形式化 TDD 而新增低价值测试。

## 品牌约定

- 构建产物：`VoxFlow.app`，安装包：`VoxFlow-<version>-macOS.dmg`
- Bundle ID：`com.voxflow.app`
- SwiftPM executable product / target / module：`VoxFlowApp`
- App 源码目录：`Sources/VoxFlowApp/`
- App 测试目录：`Tests/VoxFlowAppTests/`
- 用户数据目录：`~/Library/Application Support/VoxFlow/`
- 主数据库：`voxflow.sqlite`
- SQLite schema 快照：`Sources/VoxFlowApp/Persistence/AppDatabaseSchema.sql`
- Keychain service：`com.voxflow.app.credentials`

## 项目结构

```
Sources/VoxFlowApp/             # App 壳层、UI、装配、macOS lifecycle glue
Sources/VoxFlowDomain/          # 领域模型、任务状态、输出结果、品牌常量
Sources/VoxFlowAudio/           # 音频帧、采集、转换、endpoint / flush
Sources/VoxFlowASRCore/         # Provider / Session / Event 协议
Sources/VoxFlowModelStore/      # 模型 manifest、下载、校验、安装状态
Sources/VoxFlowTextInsertion/   # 剪贴板事务、输入源切换、文本插入
Sources/VoxFlowProviders/VoxFlowProvider*/ # 各 ASR Provider runtime / descriptor / session/client，保持独立 SwiftPM target
Sources/VoxFlowScreenshotKit/    # 截图采集、标注、滚动截图和截图窗口展示
Packages/VoxFlowVoiceCorrectionKit/ # 易错词纠错引擎、benchmark fixtures 和独立测试
agent-cli/                      # AI Coding 助手 Rust helper/router 源码，构建产物为 bundled voxflow 与 vox shim
Tests/VoxFlowAppTests/          # App target 测试
Tests/VoxFlowProviders/VoxFlowProvider*Tests/ # Provider target 测试
Tests/VoxFlow*Tests/            # 其他独立模块测试
Tests/VoxFlowScreenshotKitTests/ # 截图采集与标注模块测试
Resources/                      # AppIcon.icns + iconset
Vendor/                         # 打包所需的本地 runtime/vendor 资源
docs/                           # GitHub Pages 落地页、隐私政策、设计/资源文档

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xingbofeng/VoxFlow](https://github.com/xingbofeng/VoxFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
