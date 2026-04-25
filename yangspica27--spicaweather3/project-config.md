---
trigger: always_on
description: 基于 Jetpack Compose 开发的 Android 天气应用（柠檬天气3），使用和风天气 API，采用 Clean Architecture 分层架构。项目强调声明式 UI 和响应式数据流，集成了丰富的自定义天气动画效果。支持多城市管理、天气卡片拖拽排序、实时定位等功能。
---

# SPICaWeather3 - Copilot Instructions

## 项目概述
基于 Jetpack Compose 开发的 Android 天气应用（柠檬天气3），使用和风天气 API，采用 Clean Architecture 分层架构。项目强调声明式 UI 和响应式数据流，集成了丰富的自定义天气动画效果。支持多城市管理、天气卡片拖拽排序、实时定位等功能。

## 构建命令

```bash
# 构建 debug 版本
./gradlew assembleDebug      # Linux/Mac
gradlew.bat assembleDebug    # Windows

# 安装到设备
./gradlew installDebug

# 运行单元测试
./gradlew test

# 运行 instrumentation 测试
./gradlew connectedAndroidTest

# 清理构建
./gradlew clean
```

**前置条件**: 在 `local.properties` 中配置 `HEFENG_API_KEY` 和 `HEFENG_API_ID`（和风天气 API）。

## 架构模式

### Clean Architecture 分层

```
UI层 (Compose) → ViewModel → UseCase → Repository Interface
                                              ↓
                                    Repository Implementation
                                       ↙           ↘
                               ApiService         Room DB
                           (data/remote/)      (data/local/)
```

**技术栈核心组件:**
- **UI**: Jetpack Compose (完全声明式，无 XML layouts)
- **DI**: Koin — 4个独立模块文件在 `core/di/`
- **Navigation**: **Navigation 3** (`androidx.navigation3`) — 路由在 `route/Routes.kt`，使用 `NavKey` 标记
- **Database**: Room + Flow (响应式数据流，主实体 `CityEntity`)
- **Network**: Retrofit + OkHttp + Sandwich (API 响应封装在 `ApiResponse<T>`)
- **状态管理**: StateFlow + DataStore (偏好设置)

### 依赖注入结构 (Koin)
DI 配置拆分在 `core/di/` 下的4个独立文件:
- **`networkModule`** (`NetworkModule.kt`): OkHttpClient、Retrofit、ApiService
- **`viewModelModule`** (`ViewModelModule.kt`): Repository 实现、UseCase、ViewModel
- **`persistenceModule`** (`DatabaseModule.kt`): Room Database、CityDao、DataStoreUtil
- **`utilsModule`** (`UtilsModule.kt`): LocationHelper (百度定位)

`App.kt` 中的注册顺序：`networkModule, persistenceModule, viewModelModule, utilsModule`

**新增组件流程:**
1. 在 `core/di/` 下对应模块文件中添加 Koin 注册代码
2. ViewModel 使用 `koinActivityViewModel<T>()` 或 `koinViewModel<T>()` 获取
3. UseCase 和 Repository 使用 `factory { }` 或 `single { }` 注册

## 关键开发约定

### Compose UI 开发规范

**1. 共享元素转场动画**  
页面间过渡使用 `SharedTransitionLayout` 和组合局部变量实现:
```kotlin
// 在 AppMain.kt 中全局提供
SharedTransitionLayout {
  CompositionLocalProvider(LocalSharedTransitionScope provides this@SharedTransitionLayout) {
    // NavDisplay...
  }
}

// 在各 Screen 中使用
with(LocalSharedTransitionScope.current) {
  Modifier.sharedElement(
    rememberSharedContentState(key = "unique-key"),
    animatedVisibilityScope = LocalNavAnimatedContentScope.current
  )
}
```
**关键点**: `key` 必须在源页面和目标页面保持一致。

**CompositionLocal 访问模式**:
- `LocalSharedTransitionScope` - 共享转场作用域 (来自 AppMain.kt)
- `LocalNavAnimatedContentScope` - 动画内容作用域 (来自 `androidx.navigation3.ui`)
- `LocalNavController` - 导航控制器，类型为 `NavBackStack<NavKey>` (定义在 route/Routes.kt)
- `LocalMenuState` - 底部菜单状态管理

**使用方式**:
```kotlin
// 在 Screen composable 中获取导航控制器
val backStack = LocalNavController.current
backStack.add(Routes.WeatherList)      // 导航到目标页
backStack.removeLastOrNull()           // 返回上一页

// 访问共享转场作用域
with(LocalSharedTransitionScope.current) {
  Modifier.sharedElement(...)
}
```

**2. 自定义天气动画系统**  
基于 `common/type/WeatherAnimType.kt` 密封类配置，每个类型定义:
- 顶部/底部渐变色 (`topColor`, `bottomColor`)
- 动画元素开关 (`showGalaxy`, `showCloud`, `showRain`, `showSnow`, `showSun`)

实现位置: `ui/widget/WeatherBackground.kt`，使用 `Canvas` + `Shader` 绘制渐变  
动画组件包: `ui/widget/rain/`, `ui/widget/cloud/`, `ui/widget/sun/`, `ui/widget/galaxy/`

**Canvas 集成模式** (参考 `RainView.kt`):
```kotlin
Canvas(Modifier.fillMaxSize()) {
  drawIntoCanvas { canvas ->
    customDrawLogic(canvas.nativeCanvas) // 使用 Android 原生 Canvas API
  }
}
```

**性能优化 - 延迟渲染重型动画**:
使用 `ShowOnIdleContent` 包装复杂动画组件，在主线程空闲时才显示:
```kotlin
ShowOnIdleContent(
  visible = show,
  modifier = Modifier.fillMaxSize()
) {
  // 复杂动画组件 (如 HazeView, 使用 JBox2D 物理引擎)
}
```
实现位置: `ui/widget/ShowOnIdleContent.kt`，内部使用 `doOnMainThreadIdle()` 延迟 750ms。

**JBox2D 物理引擎集成** (用于烟雾/粒子动画):
- 在组件 `remember {}` 中创建物理世界和粒子系统
- 使用专用 `Executors.newFixedThreadPool(1)` + `asCoroutineDispatcher()` 处理物理计算
- **必须**在 `DisposableEffect` 中关闭线程池: `computeContext.close()`
- 示例: `ui/widget/haze/HazeView.kt`

```kotlin
// 正确的 JBox2D 集成模式
val computeContext = remember { 
  Executors.newFixedThreadPool(1).asCoroutineDispatcher() 
}
DisposableEffect(Unit) {
  onDispose { 
    computeContext.close()  // ← 防止内存泄漏
  }
}
```

**3. 屏幕适配 (AndroidAutoSize)**  
- 基准宽度: **375dp**
- 实现位置: `MainActivity` 实现 `CustomAdapt` 接口
- 所有尺寸自动按设备宽度缩放，无需手动适配

**4. 自定义模糊效果 (RuntimeShader + AGSL)**  
项目实现了两种模糊效果，使用 Android 13+ 的 RuntimeShader API:
- **uniformBlur()**: 均匀模糊，使用 Compose 标准 `Modifier.blur()`
- **progressiveBlur(fromBottom)**: 渐进式模糊，使用自定义 AGSL Shader

实现位置: `utils/blur/BlurImpl.kt`，关键模式:
```kotlin
// 使用 RuntimeShader 实现渐进式模糊 (Android 13+)
val shader = remember { RuntimeShader(BLUR_SHADER) }
this.graphicsLayer {
  shader.setFloatUniform("uMaxSigma", maxSigma)
  shader.setFloatUniform("uImageHeight", size.height)
  // 两步高斯模糊: X 方向 + Y 方向
  val blurEffectX = RenderEffect.createRuntimeShaderEffect(shader, "uContent")
  val blurEffectY = RenderEffect.createRuntimeShaderEffect(shader, "uContent")
  this.renderEffect = RenderEffect.createChainEffect(blurEffectY, blurEffectX).asComposeRenderEffect()
}
```
**使用方式**:
```kotlin
// 在 Compose 组件中使用
Box(Modifier.progressiveBlur(fromBottom = true)) { /* content */ }
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangSpica27/SPICaWeather3](https://github.com/yangSpica27/SPICaWeather3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
