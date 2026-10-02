---
trigger: always_on
description: Android Xposed module (LSPosed/libxposed API v102) targeting **Yadea SmartMoto** (`com.yadea.smartmoto`).
---

# YadeaHook

Android Xposed module (LSPosed/libxposed API v102) targeting **Yadea SmartMoto** (`com.yadea.smartmoto`).

## Build & Run

```bash
./gradlew assembleDebug          # Build debug APK
./gradlew assembleRelease        # Build release APK
./gradlew test                   # Run unit tests
./gradlew connectedAndroidTest   # Run instrumented tests (requires device/emulator)
```

Output APK: `app/build/outputs/apk/debug/app-debug.apk`

## Project Structure

Single module `:app`. Source files under `app/src/main/java/com/ghhccghk/yadeahook/`:

- `MainHook.kt` — Xposed module entry. Extends `XposedModule`. Handles 360加固壳 and delegates to `VehicleServiceLoad`.
- `BaseHook.kt` — 抽象基类，提供 `safeHook()`、`logHook()`、`getFieldValue()`、`dumpFields()`、`parseTtpInfo()` 等工具方法。
- `VehicleServiceLoad.kt` — 协调器，管理所有 hook 的注册和类查找。
- `VehicleStatusStore.kt` — 全局车辆状态存储，数据变化时通过广播发送 JSON。
- `VehicleController.kt` — 车辆控制命令发送器，反射调用 companion 方法。
- `hooks/BleConnectStateHook.kt` — BLE 连接状态 hook。
- `hooks/VehicleControlHook.kt` — Companion 方法监控 hook。
- `hooks/VehicleStatusHook.kt` — 电动车状态 hook（TtpInfo 字段读取）。
- `hooks/ScooterStatusHook.kt` — 滑板车状态 hook（TtpInfo 字段读取）。
- `hooks/ScooterStatusInfoHook.kt` — 滑板车状态 hook（精确匹配 `G(ScooterStatusInfo, TtpInfo)` 方法）。
- `provider/HookLogger.kt` — 日志工具，通过广播发送 hook 事件。
- `ui/LogScreen.kt` — Compose 日志界面，显示 hook 日志和 TtpInfo 数据面板。
- `ui/LogReceiver.kt` — 日志广播接收器。
- `MainActivity.kt` — Compose UI 入口，`NavigationSuiteScaffold` 导航。

## Key Libraries

| Library | Purpose |
|---------|---------|
| `ezhooktool` (v1.1.3) | Xposed hooking helper — use `EzXposed` for hook registration |
| `libxposed:api` (v102.0.0) | Core Xposed module API — `compileOnly` only, not bundled |

## Xposed Module Conventions

- `MainHook.onModuleLoaded()` calls `EzXposed.initOnModuleLoaded()` then schedules `initHooks()` via `onTargetReady`.
- `onPackageLoaded` and `onPackageReady` guard on `param.isFirstPackage` AND `param.packageName == TargetApp`.
- Hot-reload is supported via `onHotReloading` / `onHotReloaded` — delegates to `EzXposed.handleHotReloading*`.
- Target app constant: `private const val TargetApp = "com.yadea.smartmoto"` at top of `MainHook.kt`.

## 360加固壳处理

目标 APP 使用 360加固保护。需要在 `initHooks()` 中 hook `com.stub.StubApp.attachBaseContext` 获取解密后的 Context：

```kotlin
private fun initHooks() {
    loadClassOrNull("com.stub.StubApp")?.let { stubClass ->
        stubClass.findMethod {
            name("attachBaseContext")
            paramCount(1)
        }.createHook {
            after { param ->
                val context = param.args[0] as Context
                VehicleServiceLoad.initHooks(context)
            }
        }
    } ?: run {
        VehicleServiceLoad.initHooks(context = appContext)
    }
}
```

Hook 模块的 `initHooks(context: Context)` 必须使用传入的 Context 的 ClassLoader 来查找类。

## Hook 架构

```
MainHook → VehicleServiceLoad.initHooks(context)
  → 进程检查: 只在 com.yadea.smartmoto 主进程初始化
  → BleConnectStateHook.init(classLoader, context)
  → VehicleControlHook.init(classLoader, context)
  → VehicleStatusHook.init(classLoader, context)
  → ScooterStatusHook.init(classLoader, context)
  → ScooterStatusInfoHook.init(classLoader, context)
```

每个 hook 继承 `BaseHook`，实现 `init(classLoader, context)`。
类查找使用 `loadClassOrNull()` 按全限定名直接加载（360加固壳下 `findClassIf` 的 `simpleName` 搜索不可用）。

## VehicleService 类结构

目标类: `com.yadea.smartmoto.vehicle.service.VehicleService` (Kotlin)

| 内部类 | 用途 |
|--------|------|
| `a` (Companion) | 静态方法、命令发送、单例管理。实例通过 `VehicleService.o` 静态字段获取 |
| `b` | BLE 连接回调 (onConnectStateChange, onNotifyDataCallBack) |
| `c` | BluetoothGattCallback |
| `d` | NFC 部件回调 |

### Companion (VehicleService$a) 关键方法

| 方法 | 参数 | 用途 |
|------|------|------|
| `i(CommandBean)` | CommandBean | 自行车/网络控制（直接发送，不做命令映射） |
| `k(CommandBean)` | CommandBean | 滑板车/BLE 控制（调用 `t()` 做命令映射后发送） |
| `t(CommandBean, boolean)` | CommandBean, boolean | 命令类型映射：SCOOTER_LOCK→FORTIFY, SCOOTER_UNLOCK→RELEASE_FORTIFY 等 |
| `g(String)` | mac | 滑板车连接 |
| `e(String, boolean)` | mac, flag | 自行车连接 |
| `o()` | 无 | 断开连接 |
| `d()` | 无 | 取消扫描 |

### VehicleService 实例方法

| 方法 | 签名 | 用途 |
|------|------|------|
| `E` | `(PanelInfo, FaultInfo, TtpInfo)V` | 处理电动车状态 |
| `G` | `(ScooterStatusInfo, TtpInfo)V` | 处理滑板车状态 |

注意：EzXposed 的 `param.args` 包含 `this` 引用作为 `args[0]`，所以 `args[1]` 才是第一个参数。

### TtpInfo (com.yadea.blecontrol.entity.TtpInfo)

包含所有车辆数据的实体类。字段直接是数据值（如 `busbarVoltage`、`currentOdoValue`），不是字符串。
通过 `BleNotifyEntity.ttpInfo` 获取 TtpInfo 对象，再通过 `getFieldValue()` 读取字段。

**关键字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `parkingStatus` | int | 锁车状态：0=已锁, 1=未锁 |
| `onOffStatus` | int | 开机状态（11=开机） |
| `totalBatteryVoltage` | float | 总电池电压 |
| `totalBatteryElectricity` | float | 总电池电流 |
| `pBElectricitySoc` | float | 电池 SOC 百分比 |
| `currentOdoValue` | float | ODO 当前值 |
| `currentSpeedValue` | float | 车速当前值 |
| `remainingMileage` | float | 剩余里程 |
| `maximumSpeedCurrent` | int | 最高车速当前值 |
| `drivingMemory` | int | 档位记忆 |
| `cruiseControl` | int | 巡航状态 |

### CommandBean (com.yadea.smartmoto.vehicle.bean.CommandBean)

控制命令载体。构造函数：`CommandBean(String commandType)`。
发送前需调用 `t()` 做命令映射（如 `SCOOTER_LOCK` → `FORTIFY`）。

**滑板车命令映射 (Companion.t)**

| 原始命令 | 映射命令 | 参数 |
|----------|----------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghhccghk/Yadea_Hook](https://github.com/ghhccghk/Yadea_Hook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
