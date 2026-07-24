---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Kototoro 是一个开源的 Android 应用，将漫画、小说和视频整合到一个阅读器中。核心特性包括：
- 本地 OCR + 机器翻译
- 视频超分辨率（Anime4K）
- 多平台进度追踪（MAL、Kitsu、AniList、Bangumi 等）
- 广泛的图源支持：Mihon、Aniyomi、IReader、Legado、TVBox 扩展
- 动态 UI 插件系统（通过外部 classloader）
- 纯 Kotlin 实现的 OTA 增量更新（bspatch）
- WebDAV 多设备同步

## 构建和测试命令

### 基础构建
```bash
# 构建 debug APK（applicationId 后缀 .debug）
./gradlew :app:assembleDebug

# 构建 release APK（需签名，R8 混淆 + 资源压缩）
./gradlew :app:assembleRelease

# 构建 nightly APK（后缀 .nightly，版本号基于日期自动生成）
./gradlew :app:assembleNightly

# 仅编译 Kotlin 代码（最快的验证方式）
./gradlew :app:compileDebugKotlin --no-daemon

# 本机完整编译命令（需 Java 17 + 代理）
JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 \
./gradlew :app:compileDebugKotlin --no-daemon \
  -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=7890 \
  -Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=7890

# 清理构建
./gradlew clean
```

### 测试
```bash
# 运行所有 JVM 单元测试
./gradlew :app:testDebugUnitTest --no-daemon

# 运行单个测试类
./gradlew :app:testDebugUnitTest --tests "org.skepsun.kototoro.ClassName" --no-daemon

# 运行单个测试方法
./gradlew :app:testDebugUnitTest --tests "org.skepsun.kototoro.ClassName.methodName" --no-daemon

# 运行设备/模拟器上的 instrumented 测试
./gradlew :app:connectedDebugAndroidTest

# 运行所有检查（lint + 测试）
./gradlew :app:check
```

### 文档
```bash
# 启动本地文档站点（VitePress）
npm ci && npm run docs:dev

# 构建静态文档
npm run docs:build
```

### 版本信息
```bash
# 获取版本号
./gradlew printVersionName
./gradlew printVersionCode
```

## 技术栈

- **Kotlin** 2.2.10 / **AGP** 8.12.0 / **Gradle** 9.0.0（使用 `gradle/libs.versions.toml` 版本目录）
- **compileSdk** 36 / **minSdk** 26 / **targetSdk** 36
- **JVM 目标**: Java 11（开启 desugaring 以使用现代 API）
- **UI**: Jetpack Compose + Material3 + ViewBinding（混合过渡期）
- **数据库**: Room 2.7.2，KSP 代码生成
- **DI**: Hilt/Dagger（dagger 2.57.2，hilt-gradle-plugin 1.3.0）
- **网络**: OkHttp 5.2.1
- **原生代码**: `app/src/main/cpp/CMakeLists.txt`（CMake 3.22.1，4 种 ABI）
- **序列化**: kotlinx.serialization
- **测试**: JUnit5 + Kotest + MockK + MockWebServer

注意：`app/build.gradle` 使用 Groovy DSL（非 Kotlin DSL），顶层脚本用 Gradle 9.0。

## 项目架构

### 模块结构
- `app/` - 主应用模块，包含所有功能实现（compose + view）
- `parser-api/` - 共享的解析器接口定义
- `docs/` - VitePress 文档站点

### 代码组织（app/src/main/kotlin/org/skepsun/kototoro/）
项目按功能模块组织，每个模块通常包含 `data`、`domain`、`ui` 三层：

**核心模块**：
- `core/` - 核心基础设施（数据库、网络、缓存、异常处理、模型）
  - `core/db/` - Room 数据库、DAO、实体、迁移
  - `core/network/` - OkHttp 拦截器、代理、Cookie 管理、WebView 集成
  - `core/parser/` - 解析规则引擎
  - `core/model/` - 核心数据模型
- `main/` - 主入口 Activity

**内容源集成**：
- `mihon/` - Mihon/Tachiyomi 扩展集成（动态 ClassLoader、依赖注入桥接）
- `aniyomi/` - Aniyomi 扩展集成
- `ireader/` - IReader 源集成
- `extensions/` - 扩展管理框架
- `local/` - 本地文件导入（CBZ、EPUB 等）
- `alternatives/` - 替代源/镜像站管理
- `explore/` - 源预设探索
- `remotelist/` - 远程内容列表
- `picker/` - 文件/内容选择器

**内容管理**：
- `home/` - 主页和内容列表
- `discover/` - 发现和浏览
- `search/` - 搜索功能
- `details/` - 内容详情页
- `favourites/` - 收藏管理
- `history/` - 历史记录
- `bookmarks/` - 书签
- `suggestions/` - 推荐/建议
- `filter/` - 内容筛选
- `list/` - 列表视图
- `entitygraph/` - 实体关系图谱（统一管理漫画/小说/视频之间的关联）
- `stats/` - 阅读统计

**阅读体验**：
- `reader/` - 漫画/小说阅读器
- `video/` - 视频播放器（超分辨率、DLNA）
- `image/` - 图片处理和 OCR + 翻译

**同步和备份**：
- `sync/` - WebDAV 同步
- `backups/` - 备份和恢复
- `tracker/` - 外部平台进度追踪（MAL、AniList、Bangumi 等）
- `tracking/` - 追踪网站发现与候选匹配
- `scrobbling/` - 进度同步

**其他功能**：
- `settings/` - 设置界面
- `download/` - 下载管理
- `browser/` - 内置浏览器（Cloudflare 绕过）
- `widget/` - 桌面小部件

### 关键技术实现

**外部扩展集成**（参考 `docs/architecture/external-extension-integration-guide.md`）：
- 使用 ChildFirstPathClassLoader 隔离扩展依赖
- 通过依赖注入桥接提供 Application Context 和网络实例
- 动态监听 APK 安装/卸载事件（BroadcastReceiver）
- 处理 Cloudflare 挑战和 Cookie 同步

**增量 OTA 更新**（参考 `docs/architecture/incremental-updates.md`）：
- CI/CD 使用 `bsdiff` 生成增量补丁
- 纯 Kotlin 实现的 `bspatch` 算法（无 NDK 依赖）
- 严格的版本匹配验证
- 自动回退到完整 APK 下载

**数据库**：
- Room 数据库（`MangaDatabase`），DATABASE_VERSION = 40，schema 位于 `app/schemas/org.skepsun.kototoro.core.db.MangaDatabase/`
- 迁移文件 `core/db/migrations/Migration1To2.kt` 到 `Migration39To40.kt`
- 使用 KSP 生成 Kotlin 代码

**依赖注入**：
- Hilt/Dagger 用于依赖注入
- 需要在修改后运行 KSP 处理器

**测试框架**：
- JUnit5 + Kotest + MockK（单元测试）
- AndroidX Test + Hilt Testing（instrumented 测试）
- MockWebServer（网络测试）

## 开发注意事项

### 签名配置
Release 构建需要在 `local.properties` 或环境变量中配置签名：
```properties
RELEASE_STORE_FILE=/path/to/keystore
RELEASE_STORE_PASSWORD=***
RELEASE_KEY_ALIAS=***
RELEASE_KEY_PASSWORD=***
```

### 本地属性
`local.properties` 中可配置：
- `tg_backup_bot_token` - Telegram 备份机器人 token
- `dandanplay.appId` / `dandanplay.appSecret` - 弹弹 Play API 凭证

### 版本管理
在 `app/build.gradle` 中更新：
- `versionCode` - 每次发布递增
- `versionName` - 语义化版本号（如 "1.0.0.prev"）

### 代码风格
- 遵循 `.editorconfig`：UTF-8、LF、4 空格缩进、120 字符行宽
- Kotlin 官方代码风格，启用尾随逗号
- 类名使用 PascalCase，方法和属性使用 camelCase
- Android 资源使用小写下划线命名（如 `pref_appearance.xml`）

### 命名空间说明
项目主代码使用 `org.skepsun.kototoro` 命名空间（从 Kotatsu 分叉而来）。部分历史代码（如 instrumented test runner `org.koitharu.kotatsu.HiltTestRunner` 和部分 androidTest 测试类）仍保留原始包路径，这是正常现象，不要批量重命名。

### 提交规范
推荐使用 Conventional Commits 格式：
- `feat: ...` - 新功能
- `fix(scope): ...` - 修复
- `docs: ...` - 文档
- `chore: ...` - 杂项

### 翻译
翻译内容通过 Weblate 管理，避免手动批量修改字符串资源。

### 发布流程
参考 `.github/RELEASE_GUIDE.md`：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kototoro-app/Kototoro](https://github.com/Kototoro-app/Kototoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
