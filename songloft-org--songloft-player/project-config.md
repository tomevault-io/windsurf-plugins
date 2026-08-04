---
trigger: always_on
description: 本文件为 AI 编程助手提供 Songloft Flutter 前端的**入口信息**。代码本身是真实来源，本文件仅提供导航和约定。
---

# AGENTS.md

本文件为 AI 编程助手提供 Songloft Flutter 前端的**入口信息**。代码本身是真实来源，本文件仅提供导航和约定。

> **详细文档**：
> - 开发指南：[docs/development.md](docs/development.md)
> - 架构补充：[docs/architecture.md](docs/architecture.md)
> - 平台注意事项：[docs/platform-notes.md](docs/platform-notes.md)
> - 构建指南：[docs/build_guide.md](docs/build_guide.md)
> - 版本发布：[scripts/README.md](scripts/README.md)
> - 架构概览：父仓库 `docs/architecture_frontend.md`
> - 颜色系统：父仓库 `docs/color_system.md`

---

## 项目概述

Songloft 跨平台音乐播放器，基于 Flutter 3.29+ / Dart 3.7+ 构建，支持 iOS、Android、macOS、Windows、Linux、Web 六端。

独立仓库 [songloft-org/songloft-player](https://github.com/songloft-org/songloft-player)，作为父仓库 [songloft](https://github.com/songloft-org/songloft) 的子模块。后端 API 默认 `http://localhost:58091`（账号 admin/admin）。

---

## 目录结构速查

```
lib/
├── main.dart              # 应用入口，audioHandlerProvider 定义在此
├── config/                # app_config（部署模式、baseUrl）、constants（分页、播放模式、歌曲类型）
├── core/                  # 核心基础设施
│   ├── audio/             # SongloftAudioHandler、system_volume_provider
│   ├── env/               # tv_detector（TV 模式检测）
│   ├── network/           # api_client（Dio）、auth_interceptor（JWT 双 Token）、base_url_provider、servers_provider
│   ├── platform/          # live_activity_service（iOS Live Activity）
│   ├── router/            # app_router（GoRouter + 认证守卫）
│   ├── storage/           # secure_storage、app_preferences、playback_state、lyric_cache
│   ├── theme/             # app_theme（Material 3）、responsive（4 级断点）、tv_theme、app_dimensions
│   ├── tracely/           # tracely_client（可选监控）
│   └── utils/             # formatters、platform_utils、url_helper、window_tray_manager、color_extraction
├── features/              # 功能模块，每个含 data/domain/presentation 三层
│   ├── auth/              # 认证（登录/登出/Token 管理）
│   ├── home/              # 首页 + 插件 Tab/WebView
│   ├── jsplugin/          # JS 插件管理（安装/更新/注册表）
│   ├── library/           # 歌曲库（分页、搜索、编辑、收藏）
│   ├── player/            # 播放器（桌面/移动/TV/迷你 多布局 + 歌词）
│   ├── playlist/          # 歌单管理（CRUD、排序、批量操作）
│   ├── settings/          # 设置（扫描/缓存/升级/Tab 配置/重复检查/多服务器/HLS 代理/HTTP 代理）
│   └── startup/           # 启动门控（加载完成前的等待页）
└── shared/                # 共享层
    ├── layouts/           # adaptive_scaffold、shell_layout、active_destinations
    ├── models/            # Song、Playlist、Pagination、ApiResponse
    ├── utils/             # responsive_snackbar
    └── widgets/           # cover_image、confirm_dialog、song_picker、tv_focusable 等
```

---

## 常用命令

```bash
flutter pub get                                       # 安装依赖
flutter run -d chrome --no-web-resources-cdn           # Web standalone 开发
flutter run -d chrome --dart-define=DEPLOY_MODE=embedded  # Web embedded 开发
flutter run -d macos                                   # macOS 开发
flutter run -d linux                                   # Linux 开发
flutter analyze                                        # 静态分析
flutter test                                           # 运行测试

# 构建脚本
./scripts/build-frontend.sh <platform|all>             # 多平台构建
./scripts/release-frontend.sh <patch|minor|major>      # 版本发布
```

---

## 编码约定

- **状态管理**：flutter_riverpod 手写 Provider（**不使用** code generation / build_runner），三种类型：`Provider`、`NotifierProvider`、`FutureProvider`（含 `AsyncNotifierProvider`）
- **路由**：go_router 声明式路由，路径常量定义在 `AppRoutes`，认证守卫在 `redirect` 中
- **HTTP**：Dio 封装在 `core/network/api_client.dart`，`AuthInterceptor` 自动处理 JWT 双 Token 刷新
- **主题**：Material 3，seedColor `indigo-500`，**禁止**硬编码颜色值，一律 `Theme.of(context).colorScheme`
- **响应式**：4 级断点 — Mobile < 600px / Tablet 600-900px / Desktop 900-1920px / TV >= 1920px
- **条件导入**：Web 平台不支持的功能用 stub + native 文件对（如 `plugin_webview_page.dart` / `_native.dart` / `_stub.dart`）
- **import 路径**：相对路径（`../../`）
- **Lint 规则**（`analysis_options.yaml`）：`prefer_const_constructors`、`prefer_single_quotes`、`avoid_print`、`prefer_const_declarations`
- **Feature 模块结构**：`features/<name>/data/`（API 类 + Repository）、`domain/`（状态模型 + 业务逻辑）、`presentation/`（页面 + `providers/` + `widgets/`）

---

## 核心 Provider 速查

| Provider | 文件 | 职责 |
|----------|------|------|
| `authStateProvider` | `features/auth/.../auth_provider.dart` | 认证状态（登录/登出/Token） |
| `appPreferencesProvider` | 同上 | 本地偏好设置 |
| `playerStateProvider` | `features/player/.../player_provider.dart` | 播放器完整状态（当前歌曲、队列、播放模式、进度） |
| `audioHandlerProvider` | `main.dart` | SongloftAudioHandler 单例 |
| `lyricStateProvider` | `features/player/.../lyric_provider.dart` | 歌词解析与当前行定位 |
| `songsListProvider` | `features/library/.../songs_provider.dart` | 歌曲列表分页加载 |
| `songDetailProvider` | 同上 | 单曲详情 |
| `favoriteProvider` | `features/library/.../favorite_provider.dart` | 收藏状态管理 |
| `playlistListProvider` | `features/playlist/.../playlist_provider.dart` | 歌单列表 |
| `playlistNotifierProvider` | 同上 | 歌单 CRUD 操作 |
| `dioProvider` | `core/network/api_client.dart` | Dio HTTP 客户端 |
| `baseUrlProvider` | `core/network/base_url_provider.dart` | 动态 baseUrl 切换 |
| `serversProvider` | `core/network/servers_provider.dart` | 多服务器管理 |
| `routerProvider` | `core/router/app_router.dart` | GoRouter 实例 |
| `themeModeProvider` | `features/settings/.../settings_provider.dart` | 亮色/暗色/跟随系统 |
| `tabConfigProvider` | 同上 | 底栏/侧栏 Tab 配置 |
| `activeDestinationsProvider` | `shared/layouts/active_destinations.dart` | 当前激活的导航目标 |

---

## API 类


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [songloft-org/songloft-player](https://github.com/songloft-org/songloft-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
