---
trigger: always_on
description: - 重点提供开发环境、构建方式、模块路径和常见改动入口。
---

# Manga Translator 开发文档

## 文档目的
- 面向 AI AGENT 。
- 重点提供开发环境、构建方式、模块路径和常见改动入口。

---

## 快速导航

| 场景 | 关键位置 |
|-----|---------|
| 翻译主流程 | `app/src/main/java/com/manga/translate/translation/TranslationPipeline.kt` |
| 多供应商调度 | `SettingsFragment.kt`、`SettingsStore.kt`、`ProviderProfileStore.kt`、`TranslationProviderScheduler.kt`、`FolderTranslationCoordinator.kt` |
| 页面区域检测 | `app/src/main/java/com/manga/translate/detection/PageRegionDetector.kt`、`BubbleDetector.kt`（Manga109 气泡分割）、`TextDetector.kt`（yolo11n-text 游离文字） |
| OCR 相关 | `app/src/main/java/com/manga/translate/ocr/OcrSharedTools.kt`、`OcrEngine.kt`、`MangaOcrMobile.kt`、`KoreanOcr.kt`、`model/OcrApiFormat.kt`、`network/BaiduAccessTokenManager.kt`、`network/LlmClient.kt` |
| 漫画库 / 导入导出 | `LibraryFragment.kt`、`LibraryRepository.kt`、`LibraryImportExportCoordinator.kt` |
| 阅读与气泡编辑 | `ReadingFragment.kt`、`ReadingSessionViewModel.kt`、`ReadingImageTransformController.kt`、`FloatingTranslationView.kt`、`BubbleRenderer.kt`、`BubbleTextScaling.kt` |
| 设置页与参数持久化 | `SettingsFragment.kt`、`SettingsStore.kt`、`ApiSettingsStore.kt`、`OcrSettingsStore.kt`、`RenderSettingsStore.kt`、`LlmParameterStore.kt`、`ProviderProfileStore.kt` |
| 后台翻译保活 / 恢复 | `TranslationKeepAliveService.kt`、`TranslationTaskPersistence.kt`、`LibraryUiBridge.kt`、`ServiceLibraryUiCallbacks.kt` |
| 悬浮窗翻译 | `FloatingBallOverlayService.kt`、`FloatingDetectionOverlayView.kt`、`FloatingBubbleTranslationCoordinator.kt`、`FloatingEmptyBubbleCoordinator.kt` |
| 更新检测 | `UpdateChecker.kt`、`update.json` |
| 应用入口 / 共享依赖 | `MangaTranslateApp.kt`、`di/AppContainer.kt`、`MainActivity.kt` |

---

## 开发环境与构建

### 推荐环境
- 操作系统：Windows + WSL2 (Ubuntu) 或原生 Linux。
- JDK：17。
- Kotlin：2.0.0+。
- Gradle：使用项目自带 Wrapper。
- Android SDK：
  - `platforms;android-36`
  - `build-tools;36.0.0`
  - `platform-tools`

### 环境变量
```bash
export ANDROID_HOME=/home/jed/Android
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

### 常用命令
```bash
./gradlew tasks
./gradlew :app:assembleDebug
./gradlew :app:assembleRelease
./gradlew :app:compileDebugKotlin
./gradlew :app:lint
```

### 命令超时建议
- Gradle 冷启动、Kotlin 编译和定向单元测试可能超过默认的 120 秒；执行 `:app:compileDebugKotlin` 或 `:app:testDebugUnitTest` 时，工具超时至少设置为 `300000ms`（5 分钟）。
- 执行 `:app:assembleDebug`、`:app:assembleRelease`、完整单元测试和 `:app:lint` 时，工具超时至少设置为 `600000ms`（10 分钟）。

### 构建产物
- Debug APK：`app/build/outputs/apk/debug/`
- Release APK：`app/build/outputs/apk/release/`
- Lint 报告：`app/build/reports/`

### 仓库与依赖说明
- 单模块 Android 项目。
- 仓库源统一在 `settings.gradle.kts`。

---

## 仓库结构

```text
.
├── app/
├── assets/
├── dev_doc/
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── gradlew
├── update.json
└── README.md
```

### 关键目录
- `app/`：Android 应用源码与资源。
- `assets/`：模型、Prompt 等静态资源，构建时并入 app assets。
- `dev_doc/`：开发计划与开发文档。

---

## 模块路径定位

源码根目录为 `app/src/main/java/com/manga/translate`，按领域划分为 `app`、`background`、`library`、`reader`、`floating`、`settings`（设置 UI 位于 `settings/ui`）、`translation`、`network`、`ocr`、`detection`、`rendering`、`storage`、`model`、`platform` 和 `di`。

详细职责与依赖方向见 `dev_doc/package_architecture.md`。

### 入口层
- `MangaTranslateApp.kt`：Application 入口，初始化主题、语言、日志和全局依赖容器。
- `di/AppContainer.kt`：共享依赖统一创建入口。
- `MainActivity.kt`：主 Activity，负责三页导航、全局状态展示、更新检查等。
- `MainPagerAdapter.kt`：三标签页索引维护。

### UI 层
- `LibraryFragment.kt`：漫画库主页面与文件夹详情入口。
- `ReadingFragment.kt`：阅读页面容器。
- `SettingsFragment.kt`：设置页主入口。
- `SettingsStore.kt`：设置门面，向外保持统一 API，向内委托给多个子 store。
- `ApiSettingsStore.kt`、`OcrSettingsStore.kt`、`RenderSettingsStore.kt`、`AppSettingsStore.kt`、`LlmParameterStore.kt`、`ProviderProfileStore.kt`：按领域拆分后的设置持久化实现。

### Library 相关
- `LibraryRepository.kt`：漫画库目录与文件操作的核心数据入口。
- `LibraryImportExportCoordinator.kt`：导入导出流程协调。
- `FolderTranslationCoordinator.kt`：文件夹和批量翻译协调。
- `LibrarySelectionController.kt`：列表选择态控制。
- `LibraryDialogs.kt`：通用对话框封装。

### Reading 相关
- `ReadingSessionViewModel.kt`：阅读会话共享。
- `ReadingImageTransformController.kt`：阅读页手势与矩阵控制。
- `ReadingBitmapDecoder.kt` / `ReadingRegionImageView.kt`：阅读解码。普通页优先 `ARGB_8888`、detail≈屏×3；长图/超高分图走区域分块，布局坐标用源分辨率，`decodeSample` 随缩放动态降到 1 以在放大时接近原画，低分 tile 可作 fallback。
- `ReadingEmptyBubbleCoordinator.kt`：阅读页空白气泡补翻，OCR 后批量提交空白气泡翻译。
- `WebtoonReadingAdapter.kt`：条漫模式列表适配。
- `BubbleRenderer.kt`：气泡渲染。
- `FloatingTranslationView.kt`：阅读页 / 条漫页翻译覆盖层。
- `BubbleShapePaths.kt`：普通气泡框轮廓 path 组装与回缩处理。
- `BubbleTextScaling.kt`：文本缩放共享算法，提供布局拟合判定、密度自适应文字区域解析（按字数与气泡面积计算密度，不足时自动扩大路径）、水平文本二分搜索与路径缩放，三个渲染端点 (`BubbleRenderer`、`FloatingTranslationView`、`FloatingDetectionOverlayView`) 均调用该模块。

### 翻译与 OCR
- `TranslationPipeline.kt`：翻译主流程编排。
- `TranslationProviderScheduler.kt`：多供应商调度相关类型，包含附加供应商配置结构、加权候选项、页面级供应商上下文和调度器。
- `PageRegionDetector.kt`：页面区域检测公共模块；先用 Manga109 YOLO11n-seg 模型检测普通气泡并屏蔽对应区域，再用 yolo11n-text 补检游离文字，最后做 overlap/filter 与 `source` 组装。
- `LlmClient.kt`：LLM 请求客户端；文本气泡翻译已支持结构化 `items[{id,text}] -> items[{id,translation}]` 协议解析，当前网络层基于 `OkHttp`。主 AI 请求默认会读取设置页里的"API 最大重试次数 (1–50，默认 3)"并在可重试错误时按固定延时自动重试。OCR API 请求支持根据 `OcrApiFormat` 分发到 OpenAI 兼容端点或百度 AI OCR 端点，百度 AI 模式由 `BaiduAccessTokenManager` 管理 OAuth 令牌。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jedzqer/manga-translator-android](https://github.com/jedzqer/manga-translator-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
