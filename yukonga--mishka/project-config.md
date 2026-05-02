---
trigger: always_on
description: Compose Multiplatform + miuix + mihomo 跨平台代理客户端，首先完整支持 Android。
---

# Mishka

Compose Multiplatform + miuix + mihomo 跨平台代理客户端，首先完整支持 Android。

## 技术栈

| 组件                  | 版本          | 用途                                              |
| --------------------- | ------------- | ------------------------------------------------- |
| Kotlin                | 2.3.20        | 语言                                              |
| AGP                   | 9.1.1         | Android 构建                                      |
| KSP                   | 2.3.6         | 注解处理（Room）                                  |
| Compose Multiplatform | 1.10.3        | 跨平台 UI 框架                                    |
| miuix                 | 0.9.0         | UI 组件库 + 导航                                  |
| miuix-blur            | 0.9.0         | 模糊/着色器效果                                   |
| androidx.navigation3  | 1.1.0         | 类型安全路由                                      |
| Room                  | 3.0.0-alpha03 | 跨平台数据库（KMP）                               |
| Ktor                  | 3.4.2         | HTTP/WebSocket 客户端                             |
| kotlinx-coroutines    | 1.10.2        | 异步/并发                                         |
| kotlinx-datetime      | 0.7.1         | 日期时间处理                                      |
| kotlinx-serialization | 1.11.0        | JSON 序列化                                       |
| androidx.lifecycle    | 2.10.0        | ViewModel                                         |
| quickie               | 1.11.0        | QR Code 扫描                                      |
| hiddenapibypass       | 6.1           | 隐藏 API 访问（预测性返回）                       |
| mihomo                | v1.19.23 fork | 代理核心（含 --override-json + --prefetch patch） |

版本统一管理：`gradle/libs.versions.toml`（依赖）、`gradle.properties`（mihomo）、`buildSrc/ProjectConfig.kt`（应用）

## 项目结构

```
Mishka/
├── buildSrc/                         ProjectConfig + GenerateVersionInfoTask
├── shared/src/
│   ├── commonMain/kotlin/.../mishka/
│   │   ├── App.kt                    根组件 + 主题配置
│   │   ├── data/
│   │   │   ├── api/                  MihomoApiClient（REST）+ MihomoWebSocket（流）
│   │   │   ├── database/             Room 3.0 KMP（AppDatabase + 3 Entity + 3 DAO + ProfileTypeConverter）
│   │   │   ├── model/                @Serializable 数据模型 + ProfileType enum + ConfigurationOverride
│   │   │   └── repository/           MihomoRepository + SubscriptionRepository + SubscriptionFetcher + ProfileProcessor + OverrideJsonStore + SubscriptionProxyResolver
│   │   ├── platform/                 expect 声明（含 Toast）+ ProfileFileManager 接口 + ProxyServiceBridge
│   │   ├── ui/
│   │   │   ├── navigation/           AppNavigation（主导航树 + HorizontalPager）
│   │   │   ├── navigation3/          Route + Navigator（自定义栈）
│   │   │   ├── component/            SearchBar + SearchStatus + MenuPositionProvider + TriStatePreference + NullablePortPreference + ListEditDialog + RestartRequiredHint
│   │   │   │   └── effect/           BgEffectBackground（OS3 动态渐变着色器背景）
│   │   │   └── screen/               页面（home/ proxy/ subscription/ settings/ log/ provider/ dns/ connection/）
│   │   ├── viewmodel/                ViewModel
│   │   └── util/                     FormatUtils + ThrowableExt
│   ├── commonMain/composeResources/
│   │   ├── values/strings.xml        英文默认字符串
│   │   └── values-zh-rCN/strings.xml 中文字符串
│   ├── androidMain/                  actual 实现 + AppDatabaseBuilder
│   └── desktopMain/                  actual 桩实现 + AppDatabaseBuilder
├── android/src/main/
│   ├── kotlin/.../mishka/
│   │   ├── MainActivity.kt           应用入口
│   │   ├── MishkaApplication.kt      全局初始化（通知渠道 + GeoIP 提取 + 预测性返回手势 + 旧 root 文件 chown 迁移）
│   │   └── service/                  服务组件（含 ROOT 模式 + RuntimeOverrideBuilder + MihomoPrefetcher）
│   ├── res/
│   │   ├── values/strings.xml        Android 层英文字符串（通知/Tile）
│   │   └── values-zh-rCN/strings.xml Android 层中文字符串
│   ├── cpp/                          process_helper.c（JNI fork+exec）
│   └── jniLibs/arm64-v8a/            libmihomo.so
└── desktop/                          Desktop 预留入口
```

## 架构

### 依赖层级

```
MainActivity → App → AppNavigation
  → HorizontalPager（4 Tab）+ NavDisplay（二级页面）
    → Screen Composable
      → ViewModel
        → Repository（MihomoRepository / SubscriptionRepository）
          ├→ MihomoApiClient（Ktor HTTP）+ MihomoWebSocket（Ktor WS）
          │   → mihomo 进程 http://127.0.0.1:9090
          └→ Room Database（ImportedDao / PendingDao / SelectionDao）
```

### 核心模式

- **通信方案**：mihomo RESTful API + WebSocket（非 JNI），代码在 commonMain 跨平台共享
- **导航**：miuix NavDisplay + 自定义 Navigator（push/pop/popUntil + navigateForResult）+ LocalNavigator
- **主页 Tab**：HorizontalPager + MainPagerState + NavigationBar（4 Tab）
- **隧道三模式**：VPN / ROOT TUN / ROOT TPROXY（`TunMode { Vpn, RootTun, RootTproxy }`；旧 storage 值 `"root"` 自动迁移为 `"root_tun"`）
  - **VPN**：VpnService 创建 TUN fd，mihomo 写 `tun.file-descriptor` + `auto-route=false`，工作目录 `imported/{uuid}/`（app UID）
  - **ROOT TUN**：mihomo 以 root 自建 TUN，`auto-route=true` + `auto-detect-interface=true`，工作目录独立 `runtime/{uuid}/` 沙箱（启动前从 imported/ 拷贝，停止时 `su rm -rf`）；imported/ 永远 app UID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuKongA/Mishka](https://github.com/YuKongA/Mishka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
