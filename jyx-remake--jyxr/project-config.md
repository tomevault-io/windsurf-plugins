---
trigger: always_on
description: 除非用户明确要求，不要访问当前工作区以外的文件。
---

# 会话续接摘要

除非用户明确要求，不要访问当前工作区以外的文件。

遵循第一性原理，且不做补丁性、兼容性修改。

## 当前项目定位

- 这是一个基于 `.NET 10` 与 `Godot 4.6` 的 2D 半即时制战棋 RPG 内核原型。
- 仓库根目录就是 Godot 工程根。
- 根目录 `engine-free-rpg.csproj` 是当前唯一 Godot 宿主程序集项目，只编译 `src/Game.Godot/**/*.cs`，并引用 `Game.Content`、`Game.Application` 与 `Game.Presentation`。
- 当前重点是：
  - 角色与技能规则内核
  - 轻量 affix 投影
  - 轻量战斗原型
  - JSON 内容装配
  - 剧本解释器与剧情状态
  - 地图进入与点位交互
  - 持久化模型
  - 应用层会话与会话事件
  - 与 UI 引擎无关的展示流程
  - Godot 宿主层、HUD、地图、背包、商店、战斗 UI、剧情 UI、音频接线
- 当前已接入第一版轻量战斗内核与 Godot 战斗界面；正式战场系统后续仍按 `docs/battlefield-system-design.md` 继续重建。

## 当前目录与数据位置

- `mods`
  - 当前未提交的 MOD 化改造已把正式 JSON 内容从仓库根 `data` 迁到 `mods/jyxr-base/data`。
  - `mods/jyxr-base/mod.json` 是基础内容包清单；内容目录固定约定为 MOD 根目录下的 `data`。
- `launcher`
  - 当前保存 launcher 级配置文件；`userdata/<modId>` 保存各 MOD 自己的存档、档案与设置。
- `assets`
  - Godot 内置资源目录，当前主要是 `art`、`audio`、战斗动画库与图集纹理。
  - MOD 资源覆盖走 PCK；不支持 loose assets 目录覆盖。
- `autoload`
  - 目录名仍叫 `autoload`，但当前 `project.godot` 没有 `[autoload]` 注册。
  - `World`、`UIRoot`、`AudioManager` 目前由 `GameRuntimeBootstrap` 手动实例化到 `/root/__GameRuntime` 下，实质是 bootstrap 托管的运行时单例节点，不是 Godot 项目设置里的 autoload。
- `scenes`
  - 当前 Godot 主场景已切到 MOD launcher：`scenes/ui/mod_launcher/mod_launcher_panel.tscn`。
  - 还包含地图场景、HUD、剧情 UI、队伍 UI 等。
- `src/Game.Core`
  - 领域模型、定义模型、角色状态、技能实例、背包/装备实例、affix 投影、轻量战斗状态/引擎、剧情运行时、存档记录。
- `src/Game.Content`
  - JSON 内容输入、引用校验、runtime definition 装配、内存仓储。
  - `SampleData` 只保留测试样例内容。
- `src/Game.Application`
  - 应用态会话、全局档案、存读档、角色服务、背包服务、物品使用服务、商店服务、地图服务、剧情服务、剧情命令行、诊断日志抽象、会话事件。
  - 当前未提交的 MOD 改造新增 `src/Game.Application/Mods`，包含 `ProjectDataRoot`、`ModManifest`、`ModContext`、`ModRegistry`、`ModStoragePaths`、`LauncherSettingsRecord`。
- `src/Game.Presentation`
  - 普通 .NET 展示层，只表达与 UI 引擎无关的交互状态、UI intent、展示流程和宿主能力抽象。
  - 当前承载战斗 GoF State 流程与 `IBattleFlowContext`，只引用 `Game.Core`，不引用 Godot。
- `src/Game.Godot`
  - Godot 宿主适配层源码，由根目录 `engine-free-rpg.csproj` 编译。
  - 当前还包含 MOD runtime bootstrap、本地存档/档案持久化适配 `src/Game.Godot/Persistence`、主菜单、失败界面、储物箱 UI 和战斗 UI。
  - 当前未提交的 MOD launcher UI 位于 `src/Game.Godot/UI/ModLauncher` 与 `scenes/ui/mod_launcher`。
- `legacy_scenes`
  - 原 Godot/GDScript 参考场景，不是当前主运行路径。
- `jyx-legacy-data` / `jyx-legacy-dll`
  - 原版参考资源，通过 git submodule 挂载。

## 当前分层边界

- `Game.Core`
  - 不依赖 Godot。
  - 可以持有内容仓储抽象 `IContentRepository`，但不直接依赖具体 loader。
  - 存放定义、运行时状态、存档记录、剧情 runtime。
- `Game.Content`
  - 负责把 JSON 装配成 `InMemoryContentRepository`。
  - 负责 definition 引用解析、affix 引用集中解析和仓储级校验。
- `Game.Application`
  - 负责用例级服务和应用态会话。
  - 服务显式接收 `GameSession`，业务代码通过私有转发属性访问 `State`、`ContentRepository` 或其他服务。
- `Game.Presentation`
  - 负责“展示什么、当前允许哪些交互、展示流程如何转换”，不负责领域规则或应用用例。
  - 不依赖 Godot 节点、信号、资源、音频或 Tween；当前只依赖 `Game.Core`。
  - 现有各层中纯展示用途的 Presenter、ViewModel 和文案格式化可以逐步迁入；规则计算、应用用例、序列化协议与资源路径解析不要迁入。
  - `Game.Tests` 可以直接测试该层，不应为了展示流程测试引用 Godot 宿主项目或 `GodotSharp`。
- `Game.Godot`
  - 薄宿主层，负责 Godot 节点、场景、资源、音频、UI 与宿主表现。
  - Godot 侧业务日志统一经 `Game.Logger` 输出；运行期代码不要直接调用 `GD.Print` / `GD.PushError`。

## 当前运行时与存档约束

- 运行时对象可以直接持有已解析 definition 引用：
  - `CharacterInstance.Definition`
  - 技能运行时实例的 `Definition`
  - `EquipmentInstance.Definition`
  - `InventoryEntry` 中的 `ItemDefinition` / `EquipmentDefinition`
- 存档层只保存稳定 ID 和实例状态，不保存 runtime definition 引用。
- runtime definition 应尽量在内容装配阶段拿到已解析引用，不在高频路径反复按字符串查仓储。
- `GameState` 是当前可存档的大状态，持有：
  - `Adventure`
  - `Party`
  - `Inventory`
  - `Chest`
  - `EquipmentInstanceFactory`
  - `Currency`
  - `Clock`
  - `Location`
  - `MapEventProgress`
  - `Shop`
  - `Story`
  - `Journal`
- `GameState` 不使用业务构造函数，创建后通过显式 setter 装配状态。
- `AdventureState` 是当前周目的运行态上下文，属于 `GameState`，当前持有：
  - `Round`
  - `Difficulty`
  - `SectId`
  - `Morality`
  - `Favorability`
  - `Rank`
- `AdventureState.Difficulty` 当前使用稳定模式 id：
  - `normal`
  - `hard`
  - `crazy`
- `GameProfile` 是当前全局档案状态，不属于单个 `SaveGame` 槽位。
- `GameProfile` 当前持有：
  - 已解锁称号
  - 累计死亡数
  - 累计击杀数
  - 跨存档共享元宝
- `SaveGame` 是单个存档槽数据。
- `SaveGame` 不保存 `GameProfile`；全局档案单独持久化为 `GameProfileRecord`。
- 读档由 `SaveGameService.LoadSave(...)` 恢复各子状态，创建新的 `GameState`，然后调用 `GameSession.ReplaceState(...)`。
- 读全局档案由 `ProfileService.LoadProfile(...)` 恢复 `GameProfile`，然后调用 `GameSession.ReplaceProfile(...)`。
- 当前阶段服务私有字段视为非存档运行态；读档只替换 `GameSession.State`，读档案只替换 `GameSession.Profile`，不额外重建或刷新服务。

## 当前会话与宿主入口

- `GameSession` 是普通类，不是单例。
- 不要把 `GameSession` 改成静态单例类，除非用户明确要求。
- `GameSession` 当前持有：
  - `State`
  - `Profile`
  - `Config`
  - `ContentRepository`
  - `SaveGameService`
  - `ProfileService`
  - `SessionFlowService`
  - `PartyService`
  - `CharacterService`
  - `InventoryService`
  - `ChestService`
  - `ItemUseService`
  - `ShopService`
  - `MapService`
  - `StoryTimeKeyExpirationService`
  - `StoryService`
  - `SessionEvents`
- `Game` 是 Godot 宿主层全局入口，当前转发：
  - `Session`
  - `State`
  - `Profile`
  - `Config`
  - `ContentRepository`
  - `SaveGameService`
  - `ProfileService`
  - `SessionFlowService`
  - `PartyService`
  - `CharacterService`
  - `InventoryService`
  - `ChestService`
  - `ItemUseService`
  - `ShopService`
  - `MapService`
  - `StoryService`
  - `Audio`
  - `Logger`
- `Game.Initialize(...)` 当前接收已构造的 `GameSession`、当前 `ModContext` 和 logger；`GameConfig` 从 `GameSession.Config` 读取。它属于宿主启动装配，不属于业务流程调用点。
- `Game.ActiveMod` 当前保存正在运行的 MOD 上下文；本地存档、档案和设置会通过它落到该 MOD 独立的 `userdata/<modId>` 目录。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jyx-remake/jyxr](https://github.com/jyx-remake/jyxr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
