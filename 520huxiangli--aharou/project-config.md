---
trigger: always_on
description: 本仓库的 AI 协作规则（Claude Code、AiCode 等 AI 编程助手均读取本文件），优先级高于默认通用规则。
---

# CLAUDE.md

本仓库的 AI 协作规则（Claude Code、AiCode 等 AI 编程助手均读取本文件），优先级高于默认通用规则。

## 基本约定

- **永远使用中文回复。**
- 文件读写用已有的文件工具，不用 `cat` / `sed` / `echo >` 代替。
- Android 应用：Kotlin + Compose + Hilt + Coroutines/Flow，模块 `:app` / `:terminal-emulator` / `:terminal-view`。后两个是 Termux 终端组件（包名 `com.termux.terminal` / `com.termux.view`），改终端功能优先动 `feature/terminal/`。
- **优先使用项目自定义组件（硬规则）**：
  - **输入框**：优先使用 `core/ui/AppTextField.kt`（及其配套配色 `appTextFieldColors()`、弹窗专用 `dialogTextFieldColors()`），禁止在业务页面无故裸写带有长下划线的原生 `TextField`，禁止在各模块私自重复包装 OutlinedTextField。
  - **开关**：一律使用 `core/ui/AppSwitch.kt`，禁止使用原生 M3 `Switch`。
  - **模态底栏**：一律使用 `core/ui/AdaptiveModalBottomSheet.kt`（已集成平板自适应与 fling fix），禁止直接调用原生 `ModalBottomSheet`；长表单容器外层注意追加 `.imePadding()` 避让软键盘。
  - **分组卡片与列表行**：设置与表单页面优先复用 `feature/settings/presentation/component/SettingsGroupComponents.kt` 中的 `SettingsGroup`、`SettingsRow`、`SettingsDivider`、`SettingsGroupHeader`，保持统一卡片与层级质感。
  - **左滑删除**：一律使用 `core/ui/SwipeToDeleteRow.kt`。
  - **主题与设计规范**：严格遵循 `core/theme/AIEditorTheme.kt` 的 `Spacing`、`Radius`（含 `Radius.mdLarge = 12.dp`）与 `MaterialTheme.semanticColors`，禁止随意硬编码非标魔数与色彩。

## 构建与验证

**改完编译型代码（`.kt` / `.gradle.kts` / `AndroidManifest.xml`）→ 提交前跑冒烟编译；并跑 `check_migrations.py` 迁移对账。** 改了 `skills/` 下官方技能或 `evals/` 时，跑 `check_skills.py` 技能自检。只改文档 / 资源文案 / 纯 `.md` 时这些都跳过。

单元测试（`:app:testUniversalDebugUnitTest`）在容器里可能根本跑不动：项目用 Robolectric，首次要下对应 SDK 的 `android-all-instrumented`（约 190 MB，**不走 Gradle 镜像**，流量网络下会永久 hang：worker `wchan=futex_wait`、CPU 近 0、daemon 日志停写）。**跑不动就跳过交给 CI** —— `.github/workflows/android-release.yml` 在构建前会跑同一套单测，失败会拦住发版。

| 用途 | 命令 |
| --- | --- |
| 冒烟编译（日常默认） | `./gradlew :app:assembleUniversalDebug` |
| 推送前单测 | `./gradlew :app:testUniversalDebugUnitTest` |
| 推送前迁移对账 | `python3 scripts/check_migrations.py`（或 `./gradlew checkMigrations`） |
| 推送前技能自检 | `python3 scripts/check_skills.py`（frontmatter/命名、description、验收用例齐全、市场配置与文档同步） |
| 发版构建 APK / AAB | `./gradlew assembleRelease` / `./gradlew bundleRelease` |

- **别用聚合任务做日常验证**：`assembleDebug` / `assembleRelease` / `test` / `build` 都会跨三个 flavor 全跑，耗时极长。
- **别用 `--rerun-tasks` 强制全量重编**：它与 dex 增量缓存冲突，会在 `dexBuilderUniversalDebug` 上报 `NoSuchFileException` 并让后续编译停在 `UP-TO-DATE`（日志照样写 SUCCESSFUL，但 APK 不更新）。真要强制重编就删中间产物。
- 产物：`app/build/outputs/apk/<flavor>/release/app-<flavor>-release.apk`、`.../bundle/<flavor>/release/app-<flavor>-release.aab`。
- flavor 按 ABI 拆分：`universal`（arm64-v8a + x86_64）、`armsolo`（仅 arm64-v8a）、`x86solo`（仅 x86_64）。
- buildType 三个：`debug`（applicationId 后缀 `.debug`）、`beta`（后缀 `.beta`，与 release 同配置同签名，CI 用它出测试包）、`release`。三者可与正式版同机共存，日常验证用 `.debug` 即可。
- release 签名凭据读 `app/keystore.properties`（`storeFile` / `storePassword` / `keyAlias` / `keyPassword`）；本地通常不存放签名文件，CI 从 GitHub secret 还原到 `app/aicode.jks`。
- **`targetSdk = 35`**（`minSdk = 26`）：早年锁 28 是为了绕开 PRoot 的 W^X / SELinux 限制（App 可写目录不许 execve），后来 proot 全套改由 jniLibs 装进 `nativeLibraryDir`（`apk_data_file`，允许 execve），这条理由已消失。34 起前台服务必须声明类型并申请对应权限，35 起 dataSync 前台服务有「24 小时内累计 6 小时」上限——故两个前台服务统一用 `specialUse`。共享存储直读依赖「所有文件访问」（`MANAGE_EXTERNAL_STORAGE`），代码改动前先看 `app/build.gradle.kts` 的 `defaultConfig` 注释。

### CI 全景（`.github/workflows/`）

- `ci.yml`：push 与 PR 门禁，构建前跑迁移对账、官方技能自检与同一套单测。
- `android-release.yml`：由 push `v*` tag 触发，构建 APK / AAB 并发布 GitHub Release；构建前同样跑迁移对账、技能自检与单测，失败会拦住发版。
- `beta.yml`：push `master` 时自动构建 `.beta` 测试包（universal、正式签名、与 release 同配置），**只传 Actions Artifacts（保留 90 天），不进 Release**；纯文档/资源改动按 `paths-ignore` 跳过。
- `docs-deploy.yml`：**只在 push `v*` tag 时部署**文档站到 GitHub Pages（https://520huxiangli.github.io/Aharou/）——改完 `docs-site/` 推 `master` 不会上线，要等下一次发版才生效。
- `sync-gitcode.yml` / `sync-models.yml`：每日 cron 定时同步 GitCode 镜像与 models.dev 模型数据，不用手动跑。

## 架构地图

feature-based 分层 + DDD。入口 `AIEditorApp` 初始化 `FileLogger`、`TerminalKeepaliveService`、`McpManager`。

- **`core/`**：跨 feature 基础设施 —— `db/`（含 `MigrationLoader.kt`）、`net/`、`theme/`、`ui/`、`util/`（含 `FileLogger`）。
- **`feature/`**（`app/src/main/java/com/aharou/feature/`）：
  - `agent`：AI agent 核心 —— 提示词、工具注册与权限、MCP、provider 适配（`data/remote/` 下 `anthropic` / `openai` / `gemini`）。
  - `terminal`：终端与会话。本地模式 Termux 组件 + PRoot（`LinuxContainerEngine`）；远程模式 sshj。
  - `workspace`：工作区与 DocumentsProvider，远程走 `RemoteSftpFileAccess`。
  - `editor`：sora-editor 编辑器。`git`：Git 操作。`settings`：provider、日志、保活等设置。
  - `backup`：备份恢复与加密。`credentials`：凭据管理与注入容器。
- **远程 SSH 链路**：`RemoteSshConnection`（共享 sshj client）+ `RemoteSshEngine`（执行命令）+ `RemoteSftpFileAccess`（文件）+ `RemoteTerminalSessionManager`（终端）。
- **工具系统**：`feature/agent/domain/tool/` 下各工具经 `ToolRegistry` 注册，执行权限由 `ToolPermissionManager` 与 `ToolPermissionPolicyEngine` 管控。
- **MCP**：`feature/agent/domain/mcp/`，连接远端 server 并动态注册其工具。**DI**：Hilt，各 feature 自带 DI 模块。

### 数据库与迁移

Room（`feature/agent/data/local/database/AgentDatabase.kt` + 各 DAO），迁移分两套，**一个版本只能二选一**：

- **文件式（默认）**：`core/db/MigrationLoader.kt` 读 `app/src/main/assets/migrations/{VERSION}_description.sql`，由 `SqlScriptSplitter` 按语句切分（识别注释与字符串字面量，**字符串里可放心写 `;`**，不再需要 `char(59)` 绕行）；整段迁移包事务、任一条失败整体回滚，成功记入 `migration_history` 表。
- **AutoMigration**：纯 schema 变更（加列/建表/索引）可用 `@AutoMigration(from = N-1, to = N)` 编译期自动生成，改 entity 忘写迁移会直接编译失败；含数据清理/重命名/改约束的版本必须走文件式。

改 schema 三步：

1. 递增 `AgentDatabase.kt` 的 `SCHEMA_VERSION`（当前 58）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [520huxiangli/Aharou](https://github.com/520huxiangli/Aharou) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
