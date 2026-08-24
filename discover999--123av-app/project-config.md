---
trigger: always_on
description: 本文件为 AI 编码代理提供即时可用的项目上下文、约定和常用命令，便于高效修改与维护代码。
---

# Copilot 使用说明（项目：123AV_app）

本文件为 AI 编码代理提供即时可用的项目上下文、约定和常用命令，便于高效修改与维护代码。

## 1) 项目架构
- **开发语言**：Kotlin 2.0.21
- **UI 框架**：Jetpack Compose（组合式 UI）
- **主包路径**：[app/src/main/java/com/android123av/app](app/src/main/java/com/android123av/app)
- **构建工具**：Gradle 8.13.2 + Kotlin DSL + 版本目录（`libs.versions.toml`）
- **最低 SDK**：Android 11 (API 30)
- **目标 SDK**：Android 14 (API 36)
- **视频播放**：AndroidX Media3 (ExoPlayer) 1.3.1
- **本地存储**：Room Database 2.8.4

## 2) 关键组件与数据流

### UI 层
- **screens/**：主要页面组件
  - [HomeScreen.kt](app/src/main/java/com/android123av/app/screens/HomeScreen.kt) - 首页，包含视频列表、搜索、分类切换
  - [VideoPlayerScreen.kt](app/src/main/java/com/android123av/app/screens/VideoPlayerScreen.kt) - 视频播放页面
  - [FavoritesScreen.kt](app/src/main/java/com/android123av/app/screens/FavoritesScreen.kt) - 收藏页面
  - [DownloadsScreen.kt](app/src/main/java/com/android123av/app/screens/DownloadsScreen.kt) - 下载页面
  - [ProfileScreen.kt](app/src/main/java/com/android123av/app/screens/ProfileScreen.kt) - 个人中心
  - [SettingsScreen.kt](app/src/main/java/com/android123av/app/screens/SettingsScreen.kt) - 设置页面
  - [HelpScreen.kt](app/src/main/java/com/android123av/app/screens/HelpScreen.kt) - 帮助页面

### 网络层
- **network/**：网络请求与解析
  - [NetworkService.kt](app/src/main/java/com/android123av/app/network/NetworkService.kt) - 网络请求服务，包含 LRU 缓存（50 项，30 分钟过期）
  - [HtmlParser.kt](app/src/main/java/com/android123av/app/network/HtmlParser.kt) - HTML 解析器
  - [SiteManager.kt](app/src/main/java/com/android123av/app/network/SiteManager.kt) - 站点管理
  - [PersistentCookieJar.kt](app/src/main/java/com/android123av/app/network/PersistentCookieJar.kt) - Cookie 持久化

### 数据模型
- **models/**：数据模型
  - [Models.kt](app/src/main/java/com/android123av/app/models/Models.kt) - 视频数据模型
  - [VideoDetails.kt](app/src/main/java/com/android123av/app/models/VideoDetails.kt) - 视频详情模型
  - [PlayerState.kt](app/src/main/java/com/android123av/app/models/PlayerState.kt) - 播放器状态

### 视频播放
- **player/**：播放器管理
  - [ExoPlayerManager.kt](app/src/main/java/com/android123av/app/player/ExoPlayerManager.kt) - ExoPlayer 管理器
- **viewmodel/**：播放器 ViewModel
  - [VideoPlayerViewModel.kt](app/src/main/java/com/android123av/app/viewmodel/VideoPlayerViewModel.kt) - 播放器 ViewModel
  - [VideoPlayerViewModelFactory.kt](app/src/main/java/com/android123av/app/viewmodel/VideoPlayerViewModelFactory.kt) - ViewModel 工厂

### 下载管理
- **download/**：下载管理
  - [M3U8DownloadManager.kt](app/src/main/java/com/android123av/app/download/M3U8DownloadManager.kt) - M3U8 下载管理器，支持多线程下载、AES-128 解密
  - [DownloadDatabase.kt](app/src/main/java/com/android123av/app/download/DownloadDatabase.kt) - 下载数据库
  - [DownloadModels.kt](app/src/main/java/com/android123av/app/download/DownloadModels.kt) - 下载任务模型
  - [CachedVideoDetails.kt](app/src/main/java/com/android123av/app/download/CachedVideoDetails.kt) - 缓存视频详情
  - [VideoDetailsCacheManager.kt](app/src/main/java/com/android123av/app/download/VideoDetailsCacheManager.kt) - 视频详情缓存管理

### 状态管理
- **state/**：状态管理
  - [UserStateManager.kt](app/src/main/java/com/android123av/app/state/UserStateManager.kt) - 用户状态管理
  - [AppState.kt](app/src/main/java/com/android123av/app/state/AppState.kt) - 应用状态
  - [DownloadPathManager.kt](app/src/main/java/com/android123av/app/state/DownloadPathManager.kt) - 下载路径管理
  - [SearchHistoryManager.kt](app/src/main/java/com/android123av/app/state/SearchHistoryManager.kt) - 搜索历史管理
  - [ThemeStateManager.kt](app/src/main/java/com/android123av/app/state/ThemeStateManager.kt) - 主题状态管理

### UI 组件
- **components/**：可复用 UI 组件
  - [CategoryTabs.kt](app/src/main/java/com/android123av/app/components/CategoryTabs.kt) - 分类标签组件
  - [PaginationComponent.kt](app/src/main/java/com/android123av/app/components/PaginationComponent.kt) - 分页组件
  - [VideoItem.kt](app/src/main/java/com/android123av/app/components/VideoItem.kt) - 视频列表项组件
  - [NavigationComponent.kt](app/src/main/java/com/android123av/app/components/NavigationComponent.kt) - 导航组件
- **ui/components/**：UI 子组件
  - [PlayerControls.kt](app/src/main/java/com/android123av/app/ui/components/PlayerControls.kt) - 播放器控件
  - [VideoInfoPanel.kt](app/src/main/java/com/android123av/app/ui/components/VideoInfoPanel.kt) - 视频信息面板
  - [LoadingState.kt](app/src/main/java/com/android123av/app/ui/components/LoadingState.kt) - 加载状态
  - [VideoErrorState.kt](app/src/main/java/com/android123av/app/ui/components/VideoErrorState.kt) - 视频错误状态
- **ui/theme/**：主题配置
  - [Theme.kt](app/src/main/java/com/android123av/app/ui/theme/Theme.kt) - 应用主题
  - [Color.kt](app/src/main/java/com/android123av/app/ui/theme/Color.kt) - 颜色定义
  - [Type.kt](app/src/main/java/com/android123av/app/ui/theme/Type.kt) - 字体排版

## 3) 项目约定（重要，勿随意更改）

### Compose 状态管理
- 屏幕内大量使用 `mutableStateOf` / `StateFlow`，避免在非 UI 线程直接修改 Compose state
- 使用 `rememberCoroutineScope()` 与 `LaunchedEffect` 协调异步操作
- 使用 `DisposableEffect` 管理资源释放（如 ExoPlayer）

### 依赖管理
- 依赖通过版本目录引用（`libs.versions.toml`），修改依赖请同步更新该文件
- 使用 KSP（Kotlin Symbol Processing）处理 Room 数据库注解

### 视频播放
- 对 HLS (`.m3u8`) 有特殊处理（`createMediaSource`）
- 支持本地文件路径播放（`localVideoPath` 参数）
- 修改播放器请保持 `DisposableEffect` 中的 `exoPlayer.release()`
- 使用 Media3 (ExoPlayer) 1.3.1 版本

### 日志与错误处理
- 代码内广泛使用 `Log.d`/`Log.e` 与 `Toast`
- UI 交互在播放错误时恢复控件可用性

### 网络请求
- 使用 OkHttp 4.12.0 进行网络请求
- 使用 Jsoup 1.17.2 进行 HTML 解析
- 使用 Gson 2.10.1 进行 JSON 解析

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Discover999/123AV_app](https://github.com/Discover999/123AV_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
