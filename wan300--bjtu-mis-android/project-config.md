---
trigger: always_on
description: 本文件给后续 Codex 会话使用。内容只基于当前仓库中可见的 README、Gradle/npm 配置、源码目录和测试结构；不确定的信息必须标为 `TODO: confirm`。
---

# BJTU MIS Android 代理工作指南

本文件给后续 Codex 会话使用。内容只基于当前仓库中可见的 README、Gradle/npm 配置、源码目录和测试结构；不确定的信息必须标为 `TODO: confirm`。

## 0. 项目原则权威

- 项目原则以 `.specify/memory/constitution.md` 为权威来源；本文件提供代理执行细则和仓库事实。
- 如本文件与 constitution 冲突，以 constitution 为准，并同步修正文档。
- 第三方插件安全策略以 constitution 原则 VII、`docs/third-party-services.md` 和 Manifest v3 schema 为准。任何旧 v1/v2 spec、plan、API 或迁移说明仅用于历史追踪和无桥数据救援，不得作为降低当前安全边界的依据。

## 1. 项目概览

BJTU MIS Android 是面向北京交通大学学生的校园学习服务 Android App。应用直接访问 BJTU CAS、MIS、AA、VE、Coremail、知行论坛、就业服务等系统，并在设备本地保存加密凭据、Cookie 与 Room 数据快照。应用内的智能助手入口是嵌入式 Open WebUI Agent。

主要技术栈：

- Android 原生工程：Kotlin、少量 Java、Android Gradle Plugin 8.7.3、Kotlin 2.0.21、KSP。
- UI：Jetpack Compose、Material3、Navigation Compose。
- 数据与后台：Room、DataStore、WorkManager、OkHttp、Jsoup、kotlinx.serialization。
- 原生能力：Capacitor/Cordova、PyTorch Android、Media3、PDFBox Android、Commons Compress。
- 嵌入式前端：`android/open-webui/`，使用 SvelteKit、Svelte 5、TypeScript、Vite、Tailwind、Vitest。
- 网站：`web/` 下的静态前端，以及 `web/platform/` 下的 Node.js 22、TypeScript、Fastify、PostgreSQL 插件平台后端。

重要目录：

- `android/`：Android Studio 应打开的主工程目录。
- `android/app/src/main/java/cn/edu/bjtu/mis/data/`：网络访问、解析、Room、仓库、安全存储、同步、Agent 工具等数据层代码。
- `android/app/src/main/java/cn/edu/bjtu/mis/di/AppContainer.kt`：应用依赖装配入口。
- `android/app/src/main/java/cn/edu/bjtu/mis/model/`：共享模型、序列化数据结构和模块 key。
- `android/app/src/main/java/cn/edu/bjtu/mis/ui/`：Compose 应用外壳、页面、组件和主题。
- `android/app/src/main/java/cn/edu/bjtu/mis/openwebui/`：Android 与 Open WebUI 的 Capacitor 插件桥接。
- `android/app/src/main/java/cn/edu/bjtu/mis/widget/`：桌面小组件。
- `android/open-webui/`：嵌入到 Android WebView 的 Open WebUI 前端源码。
- `android/open-webui/src/lib/local-first/`：Open WebUI local-first provider、Agent loop、Android 原生工具注册和作业 handoff 逻辑。
- `android/app/src/test/kotlin/`：Android/JVM 单元测试。
- `android/app/src/test/resources/fixtures/`：解析器与 provider 测试 fixture。
- `web/`：bjtu.cc 静态前端、插件大厅页面和共享 schema。
- `web/platform/`：插件平台 API、worker、SQL migration、Dockerfile 与自动化测试。
- `plugin-tooling/`：Capability Contract Registry、确定性生成器、TypeScript SDK、CLI、Mock Host 与默认 Vite 模板。
- `deploy/`：插件平台 Docker Compose 与 bjtu.cc Nginx 配置。
- `docs/`：README 图片、插件 Manifest v3 权威开发文档、共享 schema 与各专项 ExecPlan。

## 2. Setup and local development

环境要求：

- 使用 Android Studio 打开 `android/`，等待 Gradle Sync 完成后运行 `app` target。
- JDK 17。
- Android SDK：当前 `compileSdk = 35`、`targetSdk = 35`、`minSdk = 26`。
- Node.js 与 npm：用于构建嵌入式 Open WebUI；`android/open-webui/package.json` 要求 Node `>=18.13.0 <=22.x.x`、npm `>=6.0.0`。
- 访问真实校园服务需要可访问 BJTU 相关站点的网络和有效校园账号。

安装与本地命令：

```powershell
Set-Location android
.\gradlew.bat test
.\gradlew.bat assembleDebug
```

```powershell
Set-Location android\open-webui
npm ci
npm run check
npm run test:frontend -- --run
```

```powershell
Set-Location web\platform
npm ci
npm run typecheck
npm test
npm run test:integration
npm run test:e2e
```

构建说明：

- `assembleDebug` 和 `assembleRelease` 会在 Android `preBuild` 阶段自动构建 `android/open-webui`，并同步到 `android/app/src/main/assets/public`。
- Open WebUI 构建会使用 `ENABLE_MOBILE_CLIENT=true` 和 `ENABLE_MOBILE_NATIVE_FEATURES=true`。
- `android/local.properties` 是本机 Android SDK 配置，不应提交。
- 插件平台生产编排位于 `deploy/docker-compose.plugins.yml`，Dockerfile 位于 `web/platform/Dockerfile`。
- Manifest v3 阻塞 CI 位于 `.github/workflows/plugin-platform-v3.yml`；devcontainer：TODO: confirm，当前未发现 `.devcontainer`。

## 3. Build, test, lint, and typecheck

Android：

- `Set-Location android`
- `.\gradlew.bat test`：运行 Android/JVM 单元测试；已验证当前可通过。
- `.\gradlew.bat assembleDebug`：构建 debug APK，并触发 Open WebUI 前端构建与资产同步；适合验证打包、资源和 WebView 资产路径。

Open WebUI：

- `Set-Location android\open-webui`
- `npm ci`：按 `package-lock.json` 安装依赖。
- `npm run test:frontend -- --run`：运行 Vitest 前端测试；已验证当前 9 个测试文件、81 个测试通过。
- `npm run check`：运行 `svelte-kit sync && svelte-check --tsconfig ./tsconfig.json`。当前命令可执行但存在已知基线失败，主要是 Open WebUI 中既有 TypeScript/Svelte 类型诊断。
- `npm run lint:frontend` 会执行 `eslint . --fix`，会修改文件；不要把它当作只读检查随意运行。
- `npm run format` 会执行 Prettier 写入；仅在明确需要格式化前端文件时使用。
- `npm run lint` 包含 `lint:backend` 的 `pylint backend/`，但当前仓库用途和后端目录状态需核对：TODO: confirm。

插件平台：

- `Set-Location web\platform`
- `npm run typecheck`：检查 TypeScript 类型。
- `npm test`：运行平台单元测试。
- `npm run test:integration`：设置 `TEST_DATABASE_URL` 时执行真实 PostgreSQL migration 与约束测试；未设置时相关用例会跳过。
- `npm run test:e2e`：运行 API health e2e 测试。

插件工具链：

- `Set-Location plugin-tooling`
- `npm ci`
- `npm run generate:check`：确认 schema、TypeScript、Kotlin 与文档生成物没有漂移。
- `npm run typecheck`
- `npm test`
- `npm run pack:check`：验证 npm 包和插件 ZIP 可确定性构建。

## 4. Coding conventions

Android/Kotlin：

- 遵循 `android/gradle.properties` 中的 `kotlin.code.style=official`。
- 新增 Android 依赖优先加入 `android/gradle/libs.versions.toml`，再在 Gradle 模块中引用。
- 共享 JSON 配置使用 `cn.edu.bjtu.mis.data.AppJson`，其配置包括 `ignoreUnknownKeys = true`、`explicitNulls = false`、`JsonNamingStrategy.SnakeCase`、`encodeDefaults = true`。
- 插件平台 `/api/v3` 使用 camelCase；目录客户端必须使用专用 JSON 编解码器，不得复用带 SnakeCase naming strategy 的 `AppJson`。`/api/v2` 仅保留 P0-A 只读目录兼容。
- 与外部系统交互的 suspend 函数多位于 provider/repository 层，网络错误通常以 `IOException` 或业务异常抛出，并在 UI 层用 `runCatching` 转为加载/错误状态。
- UI 延续 Compose 模式，优先复用 `MaterialTheme`、现有共享组件、`LoadState`、`ProgressiveModuleState`、`remember`、`LaunchedEffect`、`rememberCoroutineScope`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wan300/bjtu_mis_Android](https://github.com/wan300/bjtu_mis_Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
