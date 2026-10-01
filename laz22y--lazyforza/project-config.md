---
trigger: always_on
description: 本文面向第一次接触仓库的 Coding Agent。它提供定位与验证路径，不替代当前代码、测试和专项文档。
---

# LazyForza 开发入口

本文面向第一次接触仓库的 Coding Agent。它提供定位与验证路径，不替代当前代码、测试和专项文档。

## 事实来源与阅读顺序

发生冲突时，按以下顺序判断当前行为：代码和项目配置 → 自动测试 → 构建、CI 与发布脚本 → 本文和 README → ADR → 历史专项资料。不要用早期 Prompt 或设计稿覆盖已经落地的实现。

收到任务后按需阅读，不要无差别加载整个 `docs`：

1. 阅读本文和任务直接涉及的源码；
2. 查找同名功能的测试，先确认边界与兼容行为；
3. 涉及整体数据流时阅读 [`ARCHITECTURE.md`](ARCHITECTURE.md)；
4. 涉及 FH6 UDP 字段时阅读 [`FH6_TELEMETRY_DEVELOPMENT_GUIDE.md`](FH6_TELEMETRY_DEVELOPMENT_GUIDE.md) 并核对当前解析器；
5. 涉及真实游戏、Overlay 或多人联机结论时阅读 [`VALIDATION_WITH_FH6.md`](VALIDATION_WITH_FH6.md)；
6. 涉及地产赛事协议或服务端行为时，同时检查 `../LazyForza.RaceServer/AGENTS.md` 和对应实现。

## 系统概览

LazyForza 是 Windows 10/11 x64 的 .NET 9 WPF 应用。它接收 FH6 官方 324 字节 UDP Data Out，也可使用确定性 Simulator 或 `.lfztelemetry` Replay。数据经解析与标准化进入单一 `TelemetryHub`，再由仪表盘、圈速分析、地产赛事、录制和 Overlay 消费。SQLite 只保存设置、元数据和派生数据；高频原始包写入版本化文件。

地产赛事是跨仓库系统：本客户端负责本机遥测、赛道几何和赛事 HUD；LazyForza.RaceServer 负责房间权威状态，并有原生 ASP.NET 和 Cloudflare Durable Objects 两套等价实现。

当前工具链以 [`global.json`](global.json)、各 `.csproj` 和 [`Directory.Build.props`](Directory.Build.props) 为准：.NET SDK 9.0.316、C# latest、nullable、warnings-as-errors、MSTest。主 UI 与 Overlay 均为 WPF；不要依据历史资料改成 WinUI，也不要自行升级目标框架。

## 项目边界

| 项目/目录 | 当前职责 | 不应放入 |
| --- | --- | --- |
| `LazyForza.Domain` | 不可变领域记录和跨核心层的数据形状 | WPF、socket、SQLite |
| `LazyForza.Telemetry` | 官方包解析、UDP、Simulator、流状态、录制/回放、`TelemetryHub` | 赛事或 UI 业务规则 |
| `LazyForza.Analysis` | 换挡、路线投影、赛道识别、分段与纯算法 | 窗口、数据库访问 |
| `LazyForza.Storage` | `winsqlite3` 薄封装、迁移、仓储、备份和交换文件 | 高频逐帧写入、UI |
| `LazyForza.Modules.Abstractions` | 模块生命周期、遥测订阅和 HUD contribution 契约 | 具体模块行为 |
| `LazyForza.Modules.Dashboard` | 仪表盘状态与车辆学习编排 | WPF 窗口实现 |
| `LazyForza.Modules.LapAnalysis` | 普通赛道与圈状态、分析数据编排 | 第二套 UDP 接收器 |
| `LazyForza.Modules.EstateRace` | 地产几何、计时、维修区、赛事网络客户端和生成的协议模型 | 服务端权威排名/处罚 |
| `LazyForza.EstatePeer` | 无发现／中继服务的直连邀请和回执、证书固定、UDP／QUIC 通路、组件安装与本机管道 | ASP.NET 或权威赛事规则 |
| `LazyForza.EstatePeer.Host` | 可选房主进程、固定版本 RaceServer Core 与本机 Web 总控的适配、加密恢复状态 | 主客户端必需运行时依赖、第二份手写赛事规则 |
| `LazyForza.Overlay` | WPF HUD、布局持久化映射、Win32 无焦点/穿透行为 | UDP 解析、赛事计算 |
| `LazyForza.Update` | 更新检查、下载与包完整性验证 | 主窗口业务 |
| `LazyForza.App` | WPF 主壳、页面、设置、模块装配和进程生命周期 | 可复用纯算法 |
| `tests/*` | 按层测试与跨模块回归 | 真实 FH6 结论 |
| `tools/*` | 赛道目录、性能与地产诊断工具 | 运行时产品依赖 |
| `scripts/*` | 版本、预览、正式发行与发布自动化 | 用户运行时数据 |

依赖应由外向内：`App`/`Overlay`/具体模块可以依赖契约和核心层；`Domain`、`Telemetry`、`Analysis` 不得反向依赖 WPF。模块通过 `BuiltInModuleCatalog` 编译期注册，不执行未知插件 DLL。

## 不变量

- 只使用 FH6 官方 UDP 和用户主动导入的数据；禁止读取游戏内存、注入 DLL 或修改游戏进程。
- FH6 未提供的赛事 ID、对手遥测、轮胎位置、车辆尺寸、调校 ID 等不得伪造成官方字段。算法推导必须保持可辨识的 LazyForza 语义。
- 包长固定为 324 字节；命名字段只到偏移 322，字节 323 保留且没有业务语义。修改字段必须保留原始值、标准化值和来源边界。
- `TelemetryHub` 是运行时唯一遥测源。订阅通道有界并采用 latest-wins/`DropOldest`；慢 UI 或网络消费者不得反压 UDP 接收，也不要另建平行接收器。
- 模块启停必须幂等，停止时取消自身任务、释放订阅并移除 HUD contribution。主壳和诊断不依赖业务模块保持开启。
- 当前数据库 `SchemaVersion` 为 15。结构变更必须追加迁移并覆盖旧库升级、读写、事务、备份/恢复；禁止重建或静默丢弃用户数据库。
- `.lfztelemetry` 原始容器、单圈导出容器和圈速分析交换文件均有独立版本。读取旧文件的兼容路径不能因新增字段失效。
- HUD 是状态快照的渲染层。业务算法不读取平滑后的显示值，模块不直接控制 Overlay 窗口。
- 自动测试、Simulator、Replay、离屏截图和本地服务均不能写成“真实 FH6 已验证”。实机证据只在 `VALIDATION_WITH_FH6.md` 中记录。

## 修改影响地图

| 修改类型 | 至少检查 |
| --- | --- |
| FH6 UDP 字段或标准化 | `Telemetry/ForzaPacketParser*`、原始/标准化模型、录制/回放兼容、`PacketParserTests`、`StreamAndReplayTests`、遥测指南 |
| 遥测分发、断流或性能 | `TelemetryHub`、订阅者生命周期、录制和三类模块、流统计测试；确认慢消费者不反压接收 |
| 换挡或车辆识别 | `Analysis`、Dashboard 学习编排、车辆指纹与 Storage、相应 Analysis/Storage/Integration 测试 |
| 普通赛道、计圈或 Delta | `Analysis`、`Modules.LapAnalysis`、Storage 的路线/圈/分段、Lap HUD、赛道与圈速测试、Replay 回归 |
| 地产录入、几何、维修区或切弯 | `Modules.EstateRace` 的模型/状态机/几何、`App/MainWindow.EstateRace*`、Storage 迁移/导入导出、Estate 集成测试；改变赛事含义时再同步服务端 |
| 地产赛事网络协议 | RaceServer 的 `protocol/race-protocol.schema.json`、三端生成文件、客户端 `EstateRaceWireProtocol.cs` 辅助逻辑、双权威核心/路由和双端测试 |
| 数据库结构或持久化 | `LazyForzaStore.CurrentSchemaVersion`、顺序迁移、所有读写路径、备份/恢复、交换文件、`StorageTests` 和相关集成测试 |
| WPF 主界面或设置 | `App` 的 XAML/代码后置/本地化与设置持久化、Windows build、实际窗口尺寸和中英文切换人工检查 |
| Overlay 布局或动画 | `Overlay` 状态/布局/Win32 互操作、App 布局编辑器、持久化兼容、`ModuleAndOverlayTests`、多 DPI/焦点/穿透人工验证 |
| 自动更新或打包 | `LazyForza.Update`、App 更新接线、`UpdatePipelineTests`、`scripts`、包内相对路径和旧版升级路径 |
| 正式版本号或公开功能 | 项目版本、README、完整文档、官网、RaceServer 兼容说明；仅在用户明确要求时发行 |

先从测试定位行为：遥测看 `tests/LazyForza.Telemetry.Tests`；纯算法看 `LazyForza.Analysis.Tests`；迁移/备份看 `LazyForza.Storage.Tests`；模块接线、Overlay、更新和地产赛事看 `LazyForza.IntegrationTests`。修复回归时优先添加能在旧实现失败的最小测试。

## 跨仓库契约

地产赛事协议由 RaceServer 仓库中的单一 Schema 生成：

- 唯一源：`../LazyForza.RaceServer/protocol/race-protocol.schema.json`；
- 客户端产物：`src/LazyForza.Modules.EstateRace/EstateRaceProtocol.g.cs`；
- 原生服务端产物：`../LazyForza.RaceServer/src/LazyForza.RaceServer.Protocol/RaceProtocolModels.g.cs`；
- Cloudflare 产物：`../LazyForza.RaceServer/cloudflare/src/protocol.generated.ts`。

不要手工修改生成文件。协议辅助逻辑仍分别位于客户端 `EstateRaceWireProtocol.cs`、原生 `RaceProtocolJson.cs`/`RaceProtocolValidation.cs` 和 Cloudflare `protocol.ts`。在两个仓库并列存在的开发环境中，从 RaceServer 根目录运行 `node scripts/generate-protocol.mjs`，会同时更新三端产物；客户端独立构建使用已提交的生成文件，不依赖 RaceServer 或 Node.js。

它们使用协议 v2、camelCase JSON 和字符串枚举，单条消息上限为 64 KiB。新增或改变 message type、DTO 字段、枚举、默认值、可空性或错误语义时，必须：

1. 明确旧客户端/旧服务端读取新消息的行为；优先使用可选字段保持兼容；
2. 修改单一 Schema 并重新生成三端模型；
3. 同步 `.NET RaceCoordinator` 与 `cloudflare/src/race-core.ts` 的权威行为；
4. 同步 ASP.NET 路由/WebSocket 与 `cloudflare/src/index.ts`；
5. 补客户端网络流测试、RaceServer MSTest 和 Cloudflare Vitest；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Laz22y/LazyForza](https://github.com/Laz22y/LazyForza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
