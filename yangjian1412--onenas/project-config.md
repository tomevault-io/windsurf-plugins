---
trigger: always_on
description: Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing any code.
---

# Expo HAS CHANGED

Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing any code.

## Project context

- Android-only React Native app (Expo SDK 57 + RN 0.86)
- Java toolchain: JDK 17 (Temurin)
- Android SDK 35/36 with NDK 27.x
- Gradle 8.14.x
- Builds via `gradlew.bat assembleDebug` in `android/`, output is `arm64-v8a` only (~52 MB)
- For China installs, run `fix-android-env.ps1` after every `npm install` (BOM + Aliyun mirror patches)
- Use Windows PowerShell 5.1 — `&&` chained commands don't work, use `cmd1; if ($?) { cmd2 }`
- See [README](../README.md) and [NEXT-PLAN](NEXT-PLAN.md) for full context

## App identity

- Display name: **One NAS** (`app.json name` + `android/.../res/values/strings.xml app_name`)
- Package: `com.unraiddash.app`
- Slug: `one-nas`

## Native modules (Kotlin) under `android/app/src/main/java/com/unraiddash/app/`

- `MainApplication.kt` — registers `DownloadManagerPackage()`
- `MainActivity.kt` — `setTheme(R.style.AppTheme)` before `super.onCreate(null)` to dismiss native splash
- `DownloadManagerPackage.kt` — exposes `DownloadManagerModule`
- `DownloadManagerModule.kt` — methods:
  - `isExternalStorageManager()` → `Environment.isExternalStorageManager()` (precise check, Android 11+)
  - `openAllFilesAccessSettings()` → `Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION` (direct deep-link to All-files-access page)
  - `enqueueDownload(url, fileName, authToken)` → `DownloadManager.enqueue(Request.addRequestHeader("X-Auth", token).setDestinationInExternalPublicDir(Downloads, "One NAS/$fileName"))`
  - `enqueueDownloadWithHeader(url, fileName, headerName, headerValue)` → same as above but with arbitrary header (used for WebDAV `Authorization: Basic xxx`)
  - `queryProgress(downloadId)` → `{bytesDownloaded, totalBytes, status, uri, reason}`
  - `cancelDownload(downloadId)` / `removeDownload(downloadId)` → `DownloadManager.remove`

## AndroidManifest permissions

- `INTERNET`
- `READ_EXTERNAL_STORAGE` (maxSdkVersion 32)
- `WRITE_EXTERNAL_STORAGE` (maxSdkVersion 32)
- `MANAGE_EXTERNAL_STORAGE` (Android 11+)

## Splash screen (native two-phase)

- `android/app/src/main/res/values/styles.xml` `AppTheme` defines `android:windowBackground = @drawable/splash_window`
- `AppTheme` extends `Theme.AppCompat.DayNight.NoActionBar`, transparent status/nav bar, edge-to-edge
- `android/app/src/main/res/values/colors.xml` → `window_background = #FFFFFF` (light); `values-night` → `#1a1a2e`
- `drawable/splash_window.xml` = layer-list: `@color/window_background` + 居中" One NAS" 位图（`drawable/splash_text.png`，深蓝 `#1b3a8c` / `drawable-night/splash_text.png` 浅蓝 `#64b5f6`），位图显式 340×113dp
- 位图垂直位置：item `gravity="center_horizontal|bottom"` + `bottom="150dp"`（屏幕下方约 1/4 区域，避免居中显得偏高）
- Android 12+ 系统 splash（`windowSplashScreen*`）：
  - `windowSplashScreenBackground = @color/window_background`
  - `windowSplashScreenAnimatedIcon = @drawable/splash_empty`（**透明 shape**，让系统 splash 只显示纯色，与窗口背景衔接无缝）
  - `windowSplashScreenIconBackgroundColor = @android:color/transparent`
- 文字 PNG：`drawable/splash_text.png` 1440×480 透明背景，用 `System.Drawing` 渲染 "One NAS" Segoe UI Bold 170px
- **无 JS SplashView**（`src/components/SplashView.tsx` 已删）—— `App.tsx` 直接渲染 `<TabNavigator />`，省 400ms 强制等待
- **不调用 `expo-splash-screen`**（项目未装该包），原生 splash 由系统自动管理

## Adaptive icon

- `mipmap-anydpi-v26/ic_launcher.xml` (and `ic_launcher_round.xml`) reference:
  - `<background android:drawable="@color/splashscreen_background"/>` — theme-aware background
  - `<foreground android:drawable="@mipmap/ic_launcher_foreground"/>` — deep-blue rack icon (`cbi--nas-v2.svg`, fill `#1b3a8c`) at 65% size
  - `<monochrome android:drawable="@mipmap/ic_launcher_monochrome"/>` — white version for Android 13+ themed icons
- `mipmap-{m,h,xh,xxh,xxxh}dpi/` — `.webp` files generated via `sharp` from `assets/cbi--nas-v2.svg`

## Unraid GraphQL docker mutations (compatibility)

Unraid API 不同版本的 `DockerMutations` 字段集不同（**严禁**带 `Container` 后缀或 `wait` 参数调用，那是早期错误写法）：

| Unraid API 版本 | `start` | `stop` | `restart` | `pause` | `unpause` |
|---|---|---|---|---|---|
| 4.32.x / 4.33.x / 4.34.x | ✅ | ✅ | ❌ | ✅ | ✅ |
| 4.35.0+ | ✅ | ✅ | ✅ | ✅ | ✅ |

实现位置：
- `src/lib/api/unraid.ts`：暴露 `startContainer`/`stopContainer`/`restartContainer(server, id)` 三个函数（API 与签名保持稳定）
- `src/lib/api/unraidCapabilities.ts`：introspection 探测 `{ __type(name: "DockerMutations") { fields { name } } }`，结果按 `server.id` 缓存在内存（**不持久化**到 AsyncStorage，避免服务端升级后旧缓存误导）
- `restartContainer` 路由：有 `restart` → 直接用；没有 → fallback `stop` + `await sleep(RESTART_FALLBACK_DELAY_MS=1500)` + `start`
- 服务端返回 `Cannot query field`/`Unknown field` 错误时自动 `invalidateDockerCapabilities(server.id)`，下次重新探测
- DockerScreen mount 时 `useEffect(() => { unraidServers.forEach((s) => void getDockerCapabilities(s)) }, [unraidServers])` 预探测，避免首次操作多 ~100ms

错误信息提示：探测到缺失字段时（如极端旧 API 无 `start`），返回 `{ ok: false, error: '当前 Unraid API 版本不支持 start 容器，请升级 Unraid API 插件至 ≥ 4.35' }`，走 DockerScreen 现有的 `Alert.alert('操作失败', result.error)`。

VM mutations (`vm { start/stop/reboot/pause/resume/forceStop/reset }`) schema 稳定，无需特殊处理。

## NAS 管理双后端（Unraid / Portainer）

- 设置 → 服务设置 → NAS 管理 可切换 `unraid` 或 `portainer`（`appStore.nasManagementBackend`，默认 `unraid`，持久化）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangjian1412/OneNAS](https://github.com/yangjian1412/OneNAS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
