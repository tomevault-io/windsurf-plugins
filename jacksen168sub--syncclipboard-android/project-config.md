---
trigger: always_on
description: SyncClipboard Android 是一个基于 SyncClipboard API 开发的 Android 客户端应用，用于实现跨设备的剪贴板同步功能。该项目采用现代 Android 开发技术栈，支持文本、图片和文件的实时同步，具备后台自启常驻、日志记录、实时推送、智能去重等功能。
---

# SyncClipboard Android - 项目上下文文档

## 项目概述

SyncClipboard Android 是一个基于 SyncClipboard API 开发的 Android 客户端应用，用于实现跨设备的剪贴板同步功能。该项目采用现代 Android 开发技术栈，支持文本、图片和文件的实时同步，具备后台自启常驻、日志记录、实时推送、智能去重等功能。

### 核心功能
- **剪贴板实时同步**：支持文本、图片和文件的跨设备同步
- **实时推送**：通过 SignalR WebSocket 实现服务端到客户端的实时推送
- **智能去重**：基于内容哈希的智能去重机制，避免重复条目
- **后台运行**：可后台自启常驻，支持多任务页隐藏
- **日志功能**：提供完整的同步日志记录和查看
- **快捷控制**：支持 Quick Settings Tile 快速同步控制
- **图片处理**：支持图片上传、下载和缓存管理
- **内容来源跟踪**：区分本地创建、服务器同步和合并的内容
- **历史记录**：支持获取和管理历史记录

### 技术栈
- **UI 框架**：Jetpack Compose + Material 3
- **架构模式**：MVVM + Clean Architecture
- **异步处理**：Kotlin Coroutines + Flow
- **数据存储**：Room (数据库) + DataStore (配置)
- **网络通信**：Retrofit 2 + OkHttp + WebSocket (SignalR)
- **后台任务**：WorkManager
- **图片加载**：Coil
- **注解处理**：KSP (Kotlin Symbol Processing)
- **实时通信**：自定义 SignalR 协议实现
- **同步控制**：Kotlin Mutex（防止并发冲突）

### 系统要求
- **最低版本**：Android 9.0 (API 28)
- **目标版本**：Android 14 (API 35)
- **编译 SDK**：API 35
- **JDK 版本**：Java 11+
- **当前版本**：1.5.0 (versionCode: 13)

## 项目结构

```
app/src/main/java/com/jacksen168/syncclipboard/
├── data/                          # 数据层
│   ├── api/                       # API 接口定义
│   │   ├── ApiClient.kt          # API 客户端配置（含 GitHub API）
│   │   ├── GitHubApiService.kt   # GitHub API 服务（更新检查）
│   │   └── SyncClipboardApi.kt   # 剪贴板同步 API
│   ├── database/                   # 数据库
│   │   └── ClipboardDatabase.kt  # Room 数据库配置
│   ├── model/                      # 数据模型
│   │   ├── ClipboardItem.kt      # 剪贴板项模型（含智能去重）
│   │   ├── Settings.kt           # 设置模型
│   │   └── UpdateInfo.kt         # 更新信息模型
│   ├── network/                   # 网络通信
│   │   ├── SignalRClient.kt      # SignalR WebSocket 客户端
│   │   └── SignalRMessage.kt     # SignalR 消息模型
│   └── repository/                 # 仓库层
│       ├── ClipboardRepository.kt # 剪贴板数据仓库（含智能去重）
│       ├── SettingsRepository.kt  # 设置数据仓库
│       └── UpdateRepository.kt    # 更新数据仓库
├── presentation/                  # UI 层
│   ├── ClipboardTileActivity.kt  # 透明 Activity（Tile 触发）
│   ├── MainActivity.kt           # 主 Activity
│   ├── component/                 # UI 组件
│   │   ├── ErrorDialog.kt        # 错误对话框
│   │   ├── NoUpdateDialog.kt     # 无更新对话框
│   │   ├── PermissionRequestDialog.kt # 权限请求对话框
│   │   ├── UpdateCheckCard.kt    # 更新检查卡片
│   │   └── UpdateDialog.kt       # 更新对话框
│   ├── navigation/                # 导航
│   │   └── Navigation.kt         # 导航配置
│   ├── screen/                    # 页面
│   │   ├── HomeScreen.kt         # 主页
│   │   ├── LogScreen.kt          # 日志页
│   │   └── SettingsScreen.kt     # 设置页
│   ├── theme/                     # 主题
│   │   ├── Color.kt              # 颜色定义
│   │   ├── Shape.kt              # 形状定义
│   │   └── Type.kt               # 字体定义
│   └── viewmodel/                 # ViewModel
│       ├── MainViewModel.kt      # 主页 ViewModel
│       ├── SettingsViewModel.kt   # 设置页 ViewModel
│       └── UpdateViewModel.kt    # 更新 ViewModel
├── receiver/                      # 广播接收器
│   └── BootReceiver.kt           # 开机自启接收器
├── service/                       # 服务层
│   ├── ClipboardManager.kt       # 剪贴板管理器
│   ├── ClipboardSyncService.kt   # 剪贴板同步服务
│   └── ClipboardSyncTileService.kt # Quick Settings Tile 服务
├── util/                          # 工具类
│   ├── ContentLimiter.kt         # 内容限制器（UI 性能优化）
│   ├── Logger.kt                 # 日志工具
│   └── PermissionManager.kt      # 权限管理器
└── work/                          # 后台任务
    └── SyncWorker.kt             # 同步 Worker
```

## 构建和运行

### 环境要求
- **Android Studio**：最新版（推荐 2024.1+）
- **JDK**：Java 11+（推荐 Java 17）
- **Android SDK**：API 28+
- **Gradle**：项目自带 Gradle Wrapper

### 签名配置

首次构建需要配置签名：

**已有密钥文件**：
1. 将 `.jks` 文件放到 `app/` 目录下
2. 创建或编辑 `keystore.properties` 文件，填入以下内容：
```properties
storeFile=app/syncclipboard-release.jks
storePassword=your_store_password
keyAlias=your_key_alias
keyPassword=your_key_password
```

**生成新密钥**：
- Windows：`generate-keystore.bat`
- Linux：`./generate-keystore.sh`

### 构建命令

**Windows**：
```batch
# 创建 local.properties 文件（首次需要）
# sdk.dir=C:\\Users\\<用户名>\\AppData\\Local\\Android\\Sdk

# Debug 版本
gradlew.bat assembleDebug

# Release 版本（签名）
gradlew.bat assembleRelease

# Release 版本（不签名）
gradlew.bat assembleUnsignedRelease

# 清理构建
gradlew.bat clean
```

**Linux/Mac**：
```bash
# 设置执行权限（首次需要）
chmod +x gradlew.sh
chmod +x generate-keystore.sh

# Debug 版本
./gradlew.sh assembleDebug

# Release 版本（签名）
./gradlew.sh assembleRelease

# Release 版本（不签名）
./gradlew.sh assembleUnsignedRelease

# 清理构建
./gradlew.sh clean
```

### 输出位置

- **Debug**：`app/build/outputs/apk/debug/app-debug.apk`
- **Release**：`app/build/outputs/apk/release/app-release-signed.apk`
- **Unsigned Release**：`app/build/outputs/apk/unsignedRelease/app-unsignedRelease.apk`

### 安装到设备

```bash
adb install app/build/outputs/apk/release/app-release-signed.apk
```

## 开发约定

### 代码风格
- 使用 Kotlin 语言
- 遵循官方 Kotlin 编码规范
- 使用 Jetpack Compose 构建 UI
- 采用 MVVM 架构模式
- 使用 Flow 进行响应式数据流

### 包结构约定
- `data/`：数据层，包含 API、数据库、Repository、网络通信
- `presentation/`：UI 层，包含 Activity、ViewModel、Composable 组件
- `service/`：服务层，包含后台服务
- `receiver/`：广播接收器
- `util/`：工具类
- `work/`：WorkManager 后台任务

### 依赖管理
- 使用 Gradle Version Catalog (`gradle/libs.versions.toml`) 管理依赖版本
- 使用 KSP 处理 Room 注解

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacksen168sub/SyncClipboard-Android](https://github.com/jacksen168sub/SyncClipboard-Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
