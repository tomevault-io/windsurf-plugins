---
trigger: always_on
description: BlackBoxPro 是一个 Minecraft 自动化黑盒测试框架，通过 Plugin Message Channel 实现服务端→客户端的指令下发与结果回报。服务端插件向客户端 Mod 发送 JSON 指令，Mod 在客户端模拟真实玩家行为（移动、交互、GUI 操作、战斗等），用于对服务端插件逻辑进行自动化功能测试。
---

# BlackBoxPro — AI 上下文文件

## 项目概述

BlackBoxPro 是一个 Minecraft 自动化黑盒测试框架，通过 Plugin Message Channel 实现服务端→客户端的指令下发与结果回报。服务端插件向客户端 Mod 发送 JSON 指令，Mod 在客户端模拟真实玩家行为（移动、交互、GUI 操作、战斗等），用于对服务端插件逻辑进行自动化功能测试。

- 语言：Kotlin，JVM 21（1.21.11 模块）/ JVM 8（1.12.2 模块），`-Xjvm-default=all`
- 构建工具：Gradle (Kotlin DSL)，多模块项目
- 包根路径：`com.blackboxpro`
- Minecraft 版本：1.21.11、1.12.2（多版本架构，模块名含 MC 版本号）

## 项目结构

```
BlackBoxPro/                    # 根聚合项目
├── common/                     # 共享协议层
├── mod/                        # 客户端相关独立 Gradle 工程
│   ├── 1.21.11/
│   │   ├── runtime/            # 1.21.11 公共运行时核心（共享桥接 + 当前 NeoForge MC 实现）
│   │   ├── fabric/             # Fabric wrapper + 平台实现
│   │   └── neoforge/           # NeoForge wrapper + 平台实现
│   └── 1.12.2/                 # Forge 1.12.2 独立构建根
│       ├── runtime/            # 1.12.2 运行时桥接
│       └── forge/              # 1.12.2 Forge 客户端产物
├── plugin/                     # Bukkit 服务端插件（独立项目）
├── gradle.properties           # 根版本号与 1.21.11 依赖版本
├── build.gradle.kts            # 根聚合构建入口
└── settings.gradle.kts         # 根项目仅声明聚合名
```

### 多版本模块命名规则

物理目录按 `mod/{mc_version}/{loader}` 组织（如 `mod/1.21.11/fabric`、`mod/1.21.11/neoforge`），Gradle 逻辑模块名为 `mod:{mc_version}:{loader}`。Kotlin 包名保持 `com.blackboxpro.{fabric|neoforge|forge}` 不含版本号。

### 子模块职责

| 模块 | 角色 | 框架 | 入口类 |
|------|------|------|--------|
| `mod:1.21.11:runtime` | 1.21.11 公共运行时核心 | NeoForm + Kotlin JVM | 无 loader 入口 |
| `mod:1.21.11:fabric` | Fabric wrapper + 平台实现 | Fabric 1.21.11 + fabric-language-kotlin | `BlackBoxProFabric : ClientModInitializer` |
| `mod:1.21.11:neoforge` | NeoForge wrapper + 平台实现 | NeoForge 21.11.x + KotlinForForge | `BlackBoxProNeoForge` (`@Mod`) |
| `mod/1.12.2` | 客户端 Mod | Forge 1.12.2 + Kotlin 1.9.25（独立 Gradle 项目，JDK 8） | `BlackBoxProForge` (`@Mod` object) |
| `plugin` | 服务端插件 | Paper/Spigot + TabooLib 6.2.4 | `BlackBoxPro : Plugin()` (object) |
| `common` | 共享协议层 | Kotlin + Gson | 无 MC 入口 |

### 版本号管理

- `gradle.properties` (根) → `version=x.x.x` → `common`、Fabric、NeoForge、plugin、mod/1.12.2 共用
- `plugin/gradle.properties` 仅保留 group 等补充属性
- `mod/1.12.2` 为独立 Gradle 构建根，单独运行时通过 `includeBuild('../../common')` 依赖顶层 `common`
- `plugin` 通过 composite build 依赖顶层 `common`
- CI 与本地构建统一以根聚合任务为入口

## 通讯架构

```
┌─────────────────────┐     blackbox:command      ┌─────────────────────┐
│   Bukkit Server     │ ────────────────────────▶  │  Fabric/NeoForge Mod │
│   (plugin 模块)     │                            │  (客户端执行端)      │
│                     │ ◀────────────────────────  │                     │
│                     │     blackbox:response      │                     │
└─────────────────────┘                            └─────────────────────┘
```

消息格式：JSON over Plugin Message Channel，VarInt(length) + UTF-8 bytes 编码。

指令消息 (Server → Client)：`{ id, action, params, delay }`
响应消息 (Client → Server)：`{ id, status, message, data }`

## 技术栈

### 客户端 Mod (mod:1.21.11:runtime / mod:1.21.11:fabric / mod:1.21.11:neoforge)

- NeoForge 侧现为 `common + runtime + neoforge-wrapper` 分层：
  - `runtime`：开始承载 1.21.11 公共 bridge/core，并保留当前 Mojang/NeoForm 命名下的 MC 实现
  - `neoforge`：入口、事件桥、配置加载、网络注册等 loader 包装层
- Fabric 侧已接入 `runtime` 中的公共 bridge/core 源码，平台实现仍保留在 Fabric 模块
- 当前已共享到 `runtime` 的代表能力：dispatcher / scheduler / config snapshot / response sender / `wait` / `batch` / `screenshot`
- Fabric API / NeoForge / NeoForm
- fabric-language-kotlin / KotlinForForge
- SLF4J 日志
- 无外部依赖，纯 Minecraft 协议操作

### 客户端 Mod (mod/1.12.2)

- Forge 1.12.2-14.23.5.2860 + ForgeGradle 2.3
- Kotlin 1.9.25（JDK 8）
- Log4j 日志（Forge 内置）
- 独立 Gradle 项目（Groovy DSL），不在根 settings.gradle.kts 中
- 网络层使用 FMLEventChannel + CPacketCustomPayload
- 功能为 1.21.11 版本的最大兼容子集（约 80+ 个 Action）

### 服务端插件 (plugin)

- TabooLib 6.2.4（`io.izzel.taboolib` Gradle 插件 2.0.30）
- TabooLib 模块：Basic, Bukkit, BukkitUtil, CommandHelper, MinecraftChat
- NMS：`ink.ptms.core:v12105:12105` (mapped + universal)
- Gson 2.11.0

## 客户端 Mod 架构 (fabric / neoforge / forge 三端对称)

三个 Mod 模块结构对称，逻辑一致（mod/1.12.2 为最大兼容子集）：

```
com.blackboxpro.{fabric|neoforge|forge}
├── action/              # 行为执行器（每个 Action 一个类）
│   ├── movement/        # 移动类：PlayerMoveAction, PlayerLookAction...
│   ├── block/           # 方块交互：DigStartAction, PlaceBlockAction...
│   ├── entity/          # 实体交互：AttackEntityAction, InteractEntityAction...
│   ├── container/       # 容器/GUI：ClickSlotAction, CloseContainerAction...
│   ├── player/          # 玩家状态：SneakStartAction, DropItemAction...
│   ├── chat/            # 聊天命令：ChatMessageAction, ChatCommandAction
│   ├── client/          # 客户端设置：ClientInformationAction, ScreenshotAction...
│   ├── advanced/        # 进阶交互：EditBookAction, UpdateSignAction...
│   ├── debug/           # 调试：KeepAliveAction, PongAction...
│   ├── query/           # 查询行为：QueryPlayerStateAction, QueryBlockStateAction...
│   └── composite/       # 复合行为：PathfindToAction, BreakBlockAction, BatchAction...
├── dispatcher/          # 调度层：ActionRegistry, CommandDispatcher, 消息模型
├── network/             # 网络层：Channel 注册、Payload 编解码
├── config/              # 配置：BlackBoxConfig
└── util/                # 工具：DirectionUtil, HandUtil, JsonUtil, MathUtil, InjectedInput
```

核心流程：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YsGqHY/BlackBoxPro](https://github.com/YsGqHY/BlackBoxPro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
