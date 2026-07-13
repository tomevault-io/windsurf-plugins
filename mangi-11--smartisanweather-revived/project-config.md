---
trigger: always_on
description: 基于锤子天气 8.1.3 逆向对照，使用现代 Android 技术栈重写。目标是尽可能 1:1 还原原版界面、动画和触摸反馈，同时更换天气数据源并使用当前稳定的 Android 工具链。
---

# Smartisan Weather Revived

基于锤子天气 8.1.3 逆向对照，使用现代 Android 技术栈重写。目标是尽可能 1:1 还原原版界面、动画和触摸反馈，同时更换天气数据源并使用当前稳定的 Android 工具链。

## 开发约束

- 源码全部使用 Kotlin，主包为 `com.smartisan.weather`，目录固定为 `app/src/main/kotlin/com/smartisan/weather/`。
- UI 使用 XML Layout + Android View + 自定义 View；项目已经移除 Jetpack Compose 和 Material3，不要重新引入 Compose 作为页面宿主或兼容层。
- 使用多 Activity：主天气、城市搜索、城市管理和天气预警分别由独立 Activity 承载。
- 这是无历史包袱的新项目，不需要兼容旧包路径、旧数据库、旧 SharedPreferences 或旧版应用升级流程。不要添加 `com.smartisanos.*`、Java 源码或仅用于迁移旧数据的分支。
- 原版 APK/XML/资源/反编译代码用于确认尺寸、层级、状态机和动画细节；现代化应集中在生命周期、状态管理、数据层和系统 Insets，不应无依据地改写视觉行为。
- 新增或升级依赖前先查阅 Android/Kotlin 官方文档，使用最新稳定版，不使用 alpha、beta、RC 或过时兼容库。依赖与插件版本以 `gradle/libs.versions.toml` 为准，Gradle 运行版本以 `gradle/wrapper/gradle-wrapper.properties` 为准。

## 项目结构

```text
app/src/main/kotlin/com/smartisan/weather/
├── SmartisanWeatherApplication.kt       # 轻量 Application 入口
├── MainActivity.kt                      # 主天气 Activity、生命周期与页面跳转
├── WeatherGroupContainer.kt             # 多城市分页、背景和主界面切换
├── WeatherContentViewUtil.kt            # 原版天气内容 View 的创建/绑定
├── custom/                              # 原版天气自定义 View 与动画 View
│   ├── WeatherMainTemView.kt
│   ├── WeatherTempAnimView.kt
│   ├── WeatherHourForecastView.kt
│   ├── SmartisanScrollView.kt
│   ├── RefreshViewGroupLayout.kt
│   ├── DragSortListView.kt
│   ├── ElasticOverScrollLayout.kt
│   └── IndicateView.kt
├── widget/                              # 本地实现的 Smartisan 风格通用控件
│   ├── TitleBar.kt
│   ├── SearchBar.kt
│   ├── MenuDialog.kt
│   └── ShadowButton.kt
├── bean/                                # 原版 View 层使用的数据 Bean
├── data/
│   ├── model/WeatherModels.kt           # 现代天气领域模型
│   ├── city/                            # Room 3 城市数据库与仓库
│   ├── location/                        # 系统反向地理编码与天气城市匹配
│   ├── settings/WeatherSettings.kt      # DataStore Preferences 设置源
│   └── weather/                         # 天气 API、JSON 解析和缓存
├── ui/
│   ├── main/                            # 主页面 ViewModel 与领域模型映射
│   ├── search/                          # XML/View 城市搜索 Activity
│   ├── citylist/                        # XML/View 城市管理 Activity
│   ├── alert/                           # XML/View 天气预警 Activity
│   ├── license/                         # 原版离线许可/隐私 HTML 页面
│   ├── privacy/                         # 首次启动隐私提示
│   └── navigation/                      # Activity 转场
└── util/                                # 资源映射、温度单位、主题和日志工具

app/src/main/res/
├── layout/                              # 原版层级恢复后的 XML Layout
├── drawable*/                           # selector、shape、PNG 与 NinePatch
└── anim/                                # Activity/View 转场资源
```

## 当前架构

- **UI**：纯 Kotlin + XML/View，多 Activity，原版自定义 View 直接承载复杂绘制和动画。
- **工具链**：Gradle 9.6.1、AGP 9.2.1、Kotlin 2.4.0（AGP 9 内置 Kotlin，由根构建脚本覆盖编译器版本）、KSP 2.3.10、JDK 25（Gradle Daemon）与 Java 17 字节码。
- **核心库**：Room 3.0.0 + bundled SQLite 2.7.0、Activity 1.13.0、Lifecycle 2.11.0、DataStore 1.2.1、RecyclerView 1.4.0、Coroutines 1.11.0。
- **状态**：AndroidX ViewModel + Kotlin Coroutines + `StateFlow`，Activity 使用 lifecycle-aware collection；首次隐私同意前不实例化天气 ViewModel、不发天气请求。
- **数据**：Room 3 + bundled SQLite 保存城市；DataStore Preferences 是温度单位等设置的唯一数据源。
- **网络**：`HttpURLConnection` + `org.json`，接入小米天气中国区混合数据接口，不使用 Retrofit/Moshi。
- **依赖注入**：Application/Repository 手动单例，不引入 DI 框架。
- **系统 UI**：target/compile API 37，页面按 View edge-to-edge 规则分别消费状态栏、导航栏和 IME Insets。
- **定位**：系统 `LocationManager` + AndroidX `LocationManagerCompat` 获取坐标，再由小米天气 `location/city/geo` 返回 canonical `weathercn` 城市；不引入第三方定位 SDK。
- **资源**：原版可用的 XML、PNG、selector、动画和 56 个源 NinePatch 已从 APK 资源表恢复并由 aapt2 正常编译；不要再用 Compose 渐变替换这些资源。

当前 namespace 为 `com.smartisan.weather`，applicationId 为 `app.smartisanweather.revived`。原版包名 `com.smartisanos.weather` 仅用于逆向对照，不得作为新源码包。

## 构建与验证

```bash
./gradlew testDebugUnitTest  # JVM 单元测试
./gradlew assembleDebug      # Debug APK
./gradlew lintDebug          # Android Lint
./gradlew assembleRelease    # Release APK，包含 R8 与资源压缩
```

提交页面或动画改动前，至少运行：

```bash
./gradlew testDebugUnitTest assembleDebug lintDebug
```

涉及 View 尺寸、Insets、触摸或动画的改动还必须安装到模拟器/真机，通过截图、录屏和交互实际验证，不能只以编译通过作为完成标准。

## 天气 API

截至 2026-07-11，应用只使用小米天气中国区混合数据源；没有 Open-Meteo 或其他备用源。网络失败仍是正常运行时状态，天气仓库会回退到同一 provider 的本地缓存。

- 小米天气协议说明：`https://zhuti.designer.xiaomi.com/docs/blog/weatherApi.html#小米天气新接口`
- 天气：`https://weatherapi.market.xiaomi.com/wtr-v3/weather/all`
- 城市搜索：`https://weatherapi.market.xiaomi.com/wtr-v3/location/city/search`
- 坐标反查：`https://weatherapi.market.xiaomi.com/wtr-v3/location/city/geo`
- `locationKey` 固定为 `weathercn:{cityId}`，只接受 9 位中国天气城市码；搜索和坐标反查必须过滤 `status == 0` 且 key 以 `weathercn:` 开头的结果，不能把 `accu:` 海外结果混入中国源。
- 天气请求固定 `days=15`、`appKey=weather20151024`、`sign=zUFJoAR2ZVrDy1vF3D07`、`isGlobal=false`、`locale=zh_cn`。这些是公开客户端协议中的固定标识，不是用户凭证，不得输出完整请求 URL 到日志。
- `weather/all` 当前会按 `locationKey` 自行规范化彩云网格数据所需坐标；传 `latitude=0&longitude=0` 与城市标准坐标返回一致。此结论不适用于未接入的 minutely API；在增加分钟降水功能前不要为坐标修改 Room schema。
- 城市搜索没有分页，一次最多返回 20 条；UI 不得按结果数量重复请求“下一页”。
- 小米当前天气码采用中国气象局常见语义，`0..9` 通常只需补成 `00..09`，但必须按文字语义映射到原 View：小米 `20=沙尘暴` 应转换为原 View 的 `31`；不能套用主题 ContentProvider 页面中的另一套紧凑码表。未能语义等价的扩展码必须映射为 `99`。
- 每日核心行数取 `forecastDaily.weather.value` 与 `temperature.value` 的可用交集；逐小时同理。AQI、风、日出日落等可选数组长度不保证一致，缺失时不能补造 `0℃`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mangi-11/SmartisanWeather-Revived](https://github.com/Mangi-11/SmartisanWeather-Revived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
