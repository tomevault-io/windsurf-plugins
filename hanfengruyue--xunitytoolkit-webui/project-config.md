---
trigger: always_on
description: 本文件用于给后续代理提供可直接复用的仓库上下文。当前仓库已经统一改为由根目录 `AGENTS.md` 维护全部项目级、后端级、前端级说明；历史 `CLAUDE.md` 已全部删除，不再作为维护入口。
---

# AGENTS.md

本文件用于给后续代理提供可直接复用的仓库上下文。当前仓库已经统一改为由根目录 `AGENTS.md` 维护全部项目级、后端级、前端级说明；历史 `CLAUDE.md` 已全部删除，不再作为维护入口。

## 1. 权威文档入口

开始动手前，优先参考这些文件：

- `AGENTS.md`
- `README.md`

规则：

- `AGENTS.md` 是当前唯一维护中的仓库工作手册，所有原 `CLAUDE.md` 内容已并入本文件。
- `README.md` 主要面向产品说明、构建入口和用户侧信息。
- 仓库内不再保留任何 `CLAUDE.md`；如发现重新出现，应视为需要回收进 `AGENTS.md` 的重复文档。
- 若文档与源码冲突，以源码为准，并在改动后同步更新 `AGENTS.md`。

补充说明：

- 仓库内 `.claude/` 目录只有 `scheduled_tasks.lock`，没有额外项目记忆文件。
- 后端与前端原先分散在 `CLAUDE.md`、`XUnityToolkit-WebUI/CLAUDE.md`、`XUnityToolkit-Vue/CLAUDE.md` 的内容，现已统一整理到本文件后半部分的专项章节中。

## 2. 项目概览

XUnityToolkit-WebUI 是一个面向 Unity 游戏汉化/翻译工作流的 Windows 桌面工具，能力包括：

- 一键安装 BepInEx 与 XUnity.AutoTranslator
- 通过 `LLMTranslate.dll` 将游戏文本转发到本地 Web API 做 AI 翻译
- 云端 LLM 与本地 llama.cpp 模式
- 资产提取与预翻译
- TextMesh Pro 字体替换与 SDF 字体生成
- 游戏库管理、封面/图标/背景图管理
- BepInEx 日志分析、插件健康检查
- 更新器与 MSI 安装包

## 3. 仓库结构

顶层关键目录：

- `XUnityToolkit-WebUI/`
  后端主程序。ASP.NET Core Minimal API + WinForms/WebView2 宿主。
- `XUnityToolkit-Vue/`
  前端。Vue 3 + TypeScript + Naive UI + Pinia + Vite。
- `XUnityToolkit-WebUI.Tests/`
  后端测试工程。xUnit，当前覆盖翻译响应解析与运行时占位符保护等关键回归。
- `TranslatorEndpoint/`
  `net35` 的 `LLMTranslate.dll`，供 XUnity.AutoTranslator 调用。
- `Updater/`
  AOT 更新器，负责文件替换、删除、回滚、重启。
- `Installer/`
  WiX 安装器工程。
- `bundled/`
  构建/发布时附带的字体、脚本预设、BepInEx/XUnity/llama 资源。
- `.github/workflows/`
  CI/CD 工作流。

## 4. 技术栈

后端：

- `.NET 10` `net10.0-windows`
- ASP.NET Core Minimal API
- SignalR
- WinForms + WebView2
- AssetsTools.NET
- FreeTypeSharp

前端：

- Vue 3
- TypeScript
- Naive UI
- Pinia
- Vite 8

其他子项目：

- `TranslatorEndpoint`: `net35`, C# 7.3
- `Updater`: `net10.0`, `PublishAot=true`
- `Installer`: WixToolset v6（当前工程为 `WixToolset.Sdk/6.0.2`）

## 5. 常用命令

后端构建：

```bash
dotnet build XUnityToolkit-WebUI/XUnityToolkit-WebUI.csproj
dotnet build XUnityToolkit-WebUI/XUnityToolkit-WebUI.csproj -p:SkipFrontendBuild=true
dotnet run --project XUnityToolkit-WebUI/XUnityToolkit-WebUI.csproj
```

前端：

```bash
cd XUnityToolkit-Vue
npm run dev
npm run build
npx vue-tsc --build
```

测试：

```bash
dotnet test
```

补充约束：

- `dotnet build XUnityToolkit-WebUI/...` 与 `dotnet test XUnityToolkit-WebUI.Tests/...` 不要并行执行；`StaticWebAssets` 会竞争 `obj/.../rpswa.dswa.cache.json`，应串行验证

翻译端点：

```bash
dotnet build TranslatorEndpoint/TranslatorEndpoint.csproj -c Release
```

本地完整构建：

```bash
.\build.ps1
.\build.ps1 -SkipDownload
```

重要说明：

- `XUnityToolkit-WebUI.csproj` 默认会在构建前自动执行前端 `npm install` + `npm run build`。
- 前端开发代理到 `http://127.0.0.1:51821`，不要改成 `localhost`。
- 完整 UI 预览优先看后端端口 `51821`，因为它同时承载静态前端和 API。
- 发布流程当前不会在构建脚本或 GitHub Actions 中自动启动 EXE 做首页 smoke check；若需要运行态验收，请单独执行。

## 6. 运行时架构

后端启动入口：

- `XUnityToolkit-WebUI/Program.cs`

主要职责：

- 读取 `settings.json` 中 `aiTranslation.port`，动态决定监听端口，默认 `51821`
- 强制绑定 `http://127.0.0.1:{port}`
- `ContentRootPath` 与 `WebRootPath` 必须固定到 `AppContext.BaseDirectory`，不要依赖当前工作目录；否则更新器、安装器或外部启动器从错误目录拉起时会出现首页 404 但 API 仍可访问
- 注册各类命名 `HttpClient`
- 注册所有核心服务为单例
- 配置 SignalR
- 提供静态文件与 SPA fallback
- 启动日志必须记录 `CurrentDirectory`、`BaseDirectory`、`ContentRoot`、`WebRoot` 以及 `wwwroot/index.html` 是否存在；若入口文件缺失应记录 `Critical`
- 注册全部 Minimal API 端点
- 在 `ApplicationStopping` 时立即隐藏 UI，并刷新脏的翻译记忆
- 在 `ApplicationStarted` 时异步初始化 AI 翻译状态，并自动检查更新

前端入口：

- `XUnityToolkit-Vue/src/main.ts`
- `XUnityToolkit-Vue/src/App.vue`
- `XUnityToolkit-Vue/src/components/layout/AppShell.vue`

前端骨架：

- `RouterView + KeepAlive + Pinia`
- 顶层页面：游戏库、AI 翻译、字体生成、运行日志、设置
- 游戏子页面：配置编辑、资产提取、翻译编辑、术语编辑、字体替换、BepInEx 日志、插件管理

实时通信：

- 单个 SignalR Hub：`InstallProgressHub`
- 关键分组：
  - `game-{id}`
  - `ai-translation`
  - `logs`
  - `pre-translation-{gameId}`
  - `local-llm`
  - `font-replacement-{gameId}`
  - `font-generation`
  - `update`

## 7. 运行时数据布局

运行时数据根目录：

- 默认：`%AppData%\XUnityToolkit`
- 可通过配置键 `AppData:Root` 覆盖
- 主要路径集中定义在 `XUnityToolkit-WebUI/Infrastructure/AppDataPaths.cs`
- 更新暂存目录 `update-staging/` 当前由 `XUnityToolkit-WebUI/Services/UpdateService.cs` 直接管理

关键文件/目录：

- `library.json`
- `settings.json`
- `local-llm-settings.json`
- `glossaries/`
- `script-tags/`
- `translation-memory/`
- `dynamic-patterns/`
- `term-candidates/`
- `cache/covers`
- `cache/icons`
- `cache/backgrounds`
- `cache/extracted-texts`
- `cache/pre-translation-regex`
  当前 `cache/pre-translation-regex/<gameId>.txt` 只镜像 legacy compatibility 所需的 custom 正则区块；完整托管文件位于游戏目录 `BepInEx/Translation/<lang>/Text/_PreTranslated_Regex.txt`
- `cache/pre-translation-sessions`
- `models/`
- `llama/`
- `llama/launch-cache`
- `font-generation/uploads`
- `font-generation/temp`
- `generated-fonts/`
- `font-backups/`
- `custom-fonts/`
  当前字体替换自定义源按 `custom-fonts/<gameId>/ttf/` 与 `custom-fonts/<gameId>/tmp/` 分目录管理
- `backups/`
- `logs/`
- `update-staging/`

安全相关：

- API Key 与 SteamGridDB Key 使用 DPAPI 加密
- JSON 原子写入统一走 `FileHelper.WriteJsonAtomicAsync`
- 路径拼接统一优先使用 `PathSecurity.SafeJoin`
- 外部 URL 校验使用 `PathSecurity.ValidateExternalUrl`

## 8. 后端模块地图

高频关键服务：

- `GameLibraryService`
  游戏库增删改查，落盘到 `library.json`
- `AppSettingsService`
  设置缓存、DPAPI 加解密、读改写
- `ConfigurationService`
  读写 `AutoTranslatorConfig.ini`
- `UnityDetectionService`
  检测 Unity 后端、架构、可执行文件、TextMeshPro 支持
- `InstallOrchestrator`
  安装/卸载编排
- `LlmTranslationService`
  AI 翻译总入口，负责并发、统计、术语、TM、端点调度
- `TranslationMemoryService`
  每游戏翻译记忆，精确/模式/模糊匹配
- `PreTranslationService`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HanFengRuYue/XUnityToolkit-WebUI](https://github.com/HanFengRuYue/XUnityToolkit-WebUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
