---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

**PF.AutoFramework** 是面向工业自动化控制（半导体制造、多工位流水线 OCR 检测）的 .NET 8 WPF 企业框架。内置 Demo 工站 `PF.WorkStation.AutoOcr` 展示双工位五站流水线 + SECS/GEM 协议完整实现。

---

## 构建命令

```powershell
dotnet restore
dotnet build -c Release --no-restore
dotnet pack  -c Release --no-build -o nupkg   # 打包为 NuGet

publish-only.bat   # Clean → Restore → Build → Pack（三步自动化）
push-only.bat      # 推送 nupkg 到私有 BaGet 服务器 https://nuget.powerfocus.com.cn/api/v2/package（推送地址与 API Key 见脚本内 $PUSH_URL / $API_KEY，勿在此重复记录以免失步）
```

无单元测试项目，运行验证依赖 Shell 启动 + Debug/Logging 模块。

---

## 七层单向架构

```
/00. Global Config    — Directory.Build.props / Common.props / Directory.Packages.props
/01. Foundation       — PF.Core（42 接口，零外部依赖）/ PF.Infrastructure（硬件/模组/工站基类）
/02. UI               — PF.UI.Controls / PF.UI.Infrastructure / PF.UI.Resources / PF.UI.Shared
/03. Data             — PF.Data（EF Core + Repository）/ PF.SecsGem.DataBase
/04. Tools/Services   — PF.CommonTools / PF.Services（8 个服务实现）
/05. Modules          — Alarm / Debug / Identity / Logging / Parameter / ProductionRecord / SecsGem
/06. Application      — PF.Application.Shell（WPF 入口，全量 DI 注册）
/07. Workstation      — PF.WorkStation.AutoOcr + AutoOcr.UI（完整 Demo）
/08. External         — PF.SecsGem.Service（Windows 后台服务）
/09. MetaPackage      — PF.AutoFramework.Meta（NuGet 元包）
```

**关键约束**：`PF.Infrastructure` 零 Prism 依赖，Prism EventAggregator 桥接在 `App.xaml.cs` 中完成。上层只能依赖下层。

---

## 核心设计模式

### 三层硬件抽象（模板方法模式）

```
BaseMotionCard    ← 厂商 SDK 封装，21 个抽象成员（4 属性 + 17 方法）+ 3 个 BaseDevice 钩子
    ↓ AttachToCard()
BaseAxisDevice    ← 代理到 ParentCard，子类只需提供 AxisIndex / Param
BaseIODevice      ← 代理到 ParentCard，WaitInputAsync 以 20ms 轮询，泛型枚举重载
```

- **BaseDevice**：3 次重试连接（间隔 2s），健康监控循环（默认 1000ms，仿真模式 ×5），抽象钩子 `InternalConnectAsync` / `InternalDisconnectAsync` / `InternalResetAsync`
- **BaseMechanism**：聚合多硬件，`RegisterHardwareDevice()` 自动订阅报警聚合 + 批量复位，内置 `WaitAxisMoveDoneAsync`（50ms 轮询，30s 超时）/ `EnsurePointsExist<TEnum>(IAxis)`（点表自动补全 + 持久化）/ `CheckReady()`（防呆保护）
- **StationBase\<T\>**：8 状态机（Stateless）+ `SemaphoreSlim(1,1)` 状态锁 + `CancellationTokenSource _runCts` 取消式暂停，子类实现 `ProcessNormalLoopAsync` / `ProcessDryRunLoopAsync`
- **BaseMasterController**：编排所有 `IStation`，并行初始化（最大并发度 4），初始化超时 120s，复位超时 30s，防撕裂守卫（子站意外跌落 Uninitialized 时全局报警）

### 8 状态机（所有工站 + 主控共用）

```
Uninitialized ──(Initialize)──► Initializing ──(InitializeDone)──► Idle
                                      │(Error)                        │(Start)
                                   InitAlarm                        Running
                                      │(Reset)               (Pause)↕ ↕(Resume)
                                   Resetting ◄──(Reset)──── RunAlarm  Paused
                                      ├──(ResetDone)──────────────── Idle
                                      └──(ResetDoneUninitialized)── Uninitialized
```

触发器（10 个）：`Initialize` / `InitializeDone` / `Start` / `Pause` / `Resume` / `Stop` / `Error` / `Reset` / `ResetDone` / `ResetDoneUninitialized`

> **关键**：触发 `Start` / `Resume` 必须用 `await FireAsync()`，确保旧任务彻底终止后才启动新任务。

### 自动发现特性

| 特性 | 作用目标 | 消费者 |
|------|----------|--------|
| `[ModuleNavigation]` | 侧边栏菜单项（ViewName/Title/GroupName/Icon/Order） | `NavigationMenuService` |
| `[MechanismUI]` | 调试面板自动注册 | `MechanismDebugViewModel` |
| `[StationUI]` | 工站调试面板自动注册 | `StationDebugViewModel` |
| `[AlarmInfo]` | 报警码元数据（反射扫描） | `AlarmDictionaryService` |
| `[ParamView]` | 参数视图路由 | `ParameterModule` |

> **关键约束**：特性中的 `ViewName` 必须与 `RegisterForNavigation` 的 key 完全一致，否则导航失败。

---

## PF.Core 接口全览（42 个，零外部依赖）

**硬件设备**：`IHardwareDevice` / `IMotionCard` / `IAttachedDevice` / `IAxis` / `IIOController` / `ILightController` / `IBarcodeScan` / `IIntelligentCamera` / `IHardwareManagerService`

**机构 & 工站**：`IMechanism` / `IStation` / `IMasterController`

**服务**：`IParamService` / `ILogService` / `IAlarmService` / `IAlarmDictionaryService` / `IAlarmEventPublisher` / `IUserService` / `IProductionDataService` / `IRecipeService<T>` / `ITowerLightService` / `ITowerLightDoWriter` / `IAppTimerService` / `IStationSyncService`

**数据**：`IEntity` / `IGenericRepository<T>` / `IDataBase` / `IParamRepository`

**通信**：`IServer` / `IClient` / `IClientConnection`

**SECS/GEM**：`ISecsGemManager` / `ICommandManager` / `ISFCommand` / `ISecsGemDataBase` / `IParams` / `IinternalClient` / `ISecsGemMessageUpdater`

**关键枚举**：
- `MachineState`（8 个）/ `MachineTrigger`（10 个）/ `OperationMode`（Normal / DryRun）
- `HardwareCategory`（General / Axis / IOController / Camera / Robot / Scanner / Instrument / MotionCard / LightController）
- `UserLevel`（Null=-1 / Operator=0 / Engineer=1 / Administrator=2 / SuperUser=3）

---

## Shell 启动序列（App.xaml.cs）

1. `RunningInstance()` — 全局互斥锁，单实例保护
2. `CreateShell()` → `PerformInitializationAsync()` — 加载配置，Splash 进度驱动硬件初始化
3. `RegisterTypes()` — 全量 DI 注册（~450 行）：

   **硬件工厂**（6 种）：
   - `LTDMCMotionCard` — 雷赛运动控制卡（`CardIndex`）
   - `EtherCatAxis` — EtherCAT 伺服轴（`AxisIndex` / `AxisParam` JSON）
   - `EtherCatIO` — EtherCAT 数字 IO（`InPutCount` / `OutPutCount`）
   - `HKBarcodeScan` — 海康条码扫描仪（`IP` / `TiggerPort` / `UserPort` / `TimeOutMs`）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaoqiafan/PF.AutoFramework](https://github.com/yaoqiafan/PF.AutoFramework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
