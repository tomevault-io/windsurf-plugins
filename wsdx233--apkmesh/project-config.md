---
trigger: always_on
description: 本文件适用于仓库根目录及其所有子目录。父目录中的 `/home/wsdx233/AGENTS.md` 也适用；规则冲突时，以路径更具体的规则为准。
---

# APK Mesh 代理开发指南

本文件适用于仓库根目录及其所有子目录。父目录中的 `/home/wsdx233/AGENTS.md` 也适用；规则冲突时，以路径更具体的规则为准。

## 项目概览

APK Mesh 是一个 Flutter 多平台 APK 源聚合客户端。应用 UI 使用 Dart/Flutter 实现，源适配器是由 QuickJS 执行的独立 JavaScript 文件。JavaScript 不能直接访问网络、文件系统或系统安装器，而是通过宿主提供的受权限策略约束的 `apkmesh.*` API 工作。

当前包要求 Dart SDK `^3.12.2`，Flutter 工具链使用 stable channel。Android 是具备完整 QuickJS、隐藏 WebView、下载和 APK 安装能力的主要运行目标；其他平台保留不同程度的演示或受限实现。

## 目录结构

```text
.
├── lib/
│   ├── main.dart                    # 仅负责 Flutter 入口
│   ├── app.dart                     # 应用生命周期、主题和一级导航 Shell
│   ├── pages/                       # 按用户流程拆分的页面和复杂弹层
│   │   ├── home_page.dart           # 首页、搜索、分页和分类内容
│   │   ├── downloads_page.dart      # 下载列表、控制和安装动作
│   │   ├── sources_page.dart        # 源管理和源导入
│   │   ├── settings_page.dart       # 设置和翻译设置
│   │   ├── debug_sheet.dart         # 调试面板及其详情视图
│   │   └── details_sheet.dart       # 应用详情、源匹配和下载候选
│   ├── core/                        # 状态、模型、源运行时和平台能力
│   │   ├── app_state.dart           # 全局状态、源生命周期、搜索和下载编排
│   │   ├── models.dart              # 源、应用、分类、调试、下载数据模型
│   │   ├── source_runtime.dart      # SourceHostApi、源接口、权限策略、SourceRegistry
│   │   ├── quickjs_source*.dart     # QuickJS 条件导入门面和平台实现
│   │   ├── host_factory*.dart       # HTTP、WebView、下载、安装宿主
│   │   ├── debug_log.dart           # 请求、WebView 和运行日志存储
│   │   └── download_* / translation_service.dart # 下载持久化、通知和翻译
│   └── widgets/                     # 跨页面复用的展示组件
├── assets/sources/                   # Source API v1 的独立 JavaScript 源脚本
├── docs/source-api.md                # JavaScript 源契约和宿主能力文档
├── test/                             # Flutter 单元测试、Widget 测试和运行时测试
├── tools/source_debugger/            # 独立 Python QuickJS/HTTP/Playwright 调试器
│   ├── src/apkmesh_debug/            # CLI、运行时、策略、HTTP、浏览器、回放和 trace
│   ├── tests/                        # 调试器通用测试
│   ├── examples/                     # 示例源、回放 fixture 和目录检查脚本
│   ├── fixtures/                     # 可提交的示例回放定义
│   ├── pyproject.toml                # Python 项目和 pytest 配置
│   └── uv.lock                       # Python 依赖锁文件
├── android/                          # Android Gradle 工程和 Kotlin 原生桥接
├── ios/ macos/ linux/ web/ windows/  # Flutter 各平台壳工程
├── pubspec.yaml / pubspec.lock       # Dart/Flutter 依赖和锁文件
├── analysis_options.yaml             # Flutter lint 配置
├── README.md                         # 用户说明、开发入口和法律边界
└── LICENSE
```

平台目录中由 Flutter 或插件生成的文件（例如 `build/`、`.dart_tool/`、`linux/flutter/`、`windows/flutter/` 中的生成注册文件）不是业务逻辑。除非任务明确涉及平台工程，否则不要手工修改或提交生成产物。

### Dart/Flutter 模块边界

- `main.dart` 只保留 `main()` 和入口导入；主题、生命周期和一级导航放在 `app.dart`。
- `pages/` 按用户流程组织页面。页面可以组合 `core` 状态和 `widgets` 展示组件，但网络解析、权限校验和下载编排必须留在 `core` 或源脚本边界。
- `widgets/` 只放跨页面复用的展示组件或局部交互组件；例如下载状态控件和截图查看器不得重新复制到各页面。
- 新增跨文件使用的 Widget 或 helper 才公开命名；仅在所属模块内使用的类型保持私有，避免无意扩大 API。公开 Widget 构造函数必须保留 `key`。
- 单文件超过约 1000 行时，优先按完整职责提取到独立模块，并同步迁移测试和 import；不要用机械切片或为了降低行数引入无语义的包装层。`core` 中的状态编排可保持集中，但新增功能应优先放入独立服务或运行时模块。
- `core` 不得反向依赖 `pages/` 或 `widgets/`；入口层可以依赖页面层，页面层可以依赖核心层。

`lib/features/` 目前不是实际的业务代码目录；新增页面应放在 `lib/pages/`，只有在拆分能降低真实复杂度时才增加新的功能子目录。

## 运行时架构

主调用链如下：

```text
Flutter UI (`app.dart` + `pages/`)
        |
     AppState
        |
   SourceRegistry  <-- ApkSourceScript / SourceCatalogScript / DebugProjectSource
        |
QuickJsApkSourceScript (Android) or built-in Demo source
        |
SourceHostApi
        |
NativeHostApi (HTTP + Headless WebView + download/install)
       or DemoHostApi (受限/不可用的演示实现)
```

- `main.dart` 的 `Shell` 已迁移到 `app.dart`，提供主页、下载、源管理、设置四个一级入口；详情、调试和截图查看器分别位于 `pages/` 与 `widgets/`，仍以 sheet/dialog 形式打开。
- `AppState.initialize()` 当前在 Android 上通过 Flutter `AssetManifest` 自动扫描并加载 `assets/sources/` 下的所有 `.js` 源脚本，源管理元数据来自各脚本的 manifest；不支持 QuickJS 的平台保留内置演示源。新增源脚本只要纳入 `assets/sources/` 资产即可被发现。
- `SourceRegistry.search()` 并发调用启用源，保留单个源错误并聚合其他成功结果；修改此行为时必须同步更新测试和 UI 的错误展示逻辑。
- `quickjs_source_io.dart` 目前只在 Android 创建 QuickJS 源运行时；`quickjs_source_stub.dart` 在不支持平台返回空实现。`host_factory_io.dart` 的隐藏 WebView 支持 Android/iOS/macOS，APK 安装仅支持 Android；Linux/Windows 没有隐藏 WebView 实现，Web 使用 stub 宿主。
- Android 原生 `MainActivity.kt` 通过 `com.apkmesh/install` 和 `com.apkmesh/download_notifications` 两个 MethodChannel 提供安装权限检查、未知来源权限跳转和下载通知。
- `tools/source_debugger` 是与 Flutter 宿主平行的 Python 实现，用于无 Android 设备执行源脚本，并支持 live、record、replay 三种模式。它不是 Flutter 应用的运行时依赖。

## Source API 约束

Source API 的规范来源是 [`docs/source-api.md`](docs/source-api.md)。每个源是单个可独立加载的 JavaScript 文件，必须设置 `globalThis.source`，并提供：

- `manifest`：至少包含源 ID、名称、版本、主页和权限声明；
- `async search(query, page)`：返回应用摘要数组；
- `async details(idOrUrl)`：返回完整应用详情和下载项；
- 可选的 `home()`、`category(categoryId)` 和 `debug(projectId, input)`。

源返回的数据必须符合文档中的字段形状。新增或修改以下任一契约时，必须成组检查并按需更新：

1. `docs/source-api.md`；
2. Dart 的 `SourceHostApi`、模型、QuickJS bootstrap/消息桥和宿主实现；
3. Python 调试器的 bootstrap、dispatch、模型和宿主实现；
4. 相关 JavaScript 源、回放 fixture 及 Dart/Python 测试。

不要只修改一个端的桥接代码。尤其要确认 `headers`、选择器查询结果、异步 Promise、下载和错误的序列化在 Flutter QuickJS 与 Python QuickJS 中保持一致。

## 安全与法律边界

这些规则是功能约束，不是可选的产品建议：

- 所有源网络、浏览器、下载和安装操作都必须通过 `SourcePolicy` 声明的权限；不得在 UI 或源脚本中绕过宿主检查。
- 网络规则使用 manifest 中的精确主机名、`*.example.com` 子域名规则或显式的 `'*'` 任意主机权限。`'*'` 只允许 `http`/`https`，适用于用户明确信任且会返回临时下载主机的源；每一次 HTTP 重定向仍要重新校验协议和 manifest 权限。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wsdx233/ApkMesh](https://github.com/wsdx233/ApkMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
