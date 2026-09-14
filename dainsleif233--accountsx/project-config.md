---
trigger: always_on
description: AI agent 和所有开发者的项目权威手册。保持与代码同步；任何约束变更必须同步更新此文件。
---

# AGENTS.md

AI agent 和所有开发者的项目权威手册。保持与代码同步；任何约束变更必须同步更新此文件。

## 项目简介

Accounts X 是一个**客户端 Fabric 模组**，用于 Minecraft 多账号切换。支持离线账号、Microsoft（device-code OAuth）、Authlib-Injector（第三方 Yggdrasil）和 United-Injector 账号，以及启动器提供的环境账号。

- 基础包：`top.syshub.accountsx`
- Java **25**（`sourceCompatibility` / `targetCompatibility`）
- Gradle **9.7.0**（wrapper），Fabric Loom **1.17-SNAPSHOT**
- Fabric Loader：**0.19.3**（各适配器固定版本；common 使用 `compileOnly`）
- 版本号：`gradle.properties`
- 许可：GPL-3.0
- 仓库：https://github.com/Dainsleif233/AccountsX

### MC 版本支持矩阵

| Minecraft | authlib | Loom 插件         | Fabric API |
|-----------|---------|-------------------|------------|
| 1.20      | 4.0.43  | fabric-loom-remap | 0.83.0     |
| 1.20.2    | 5.0.47  | fabric-loom-remap | 0.91.6     |
| 1.20.3    | 6.0.52  | fabric-loom-remap | 0.91.1     |
| 1.20.5    | 6.0.54  | fabric-loom-remap | 0.97.8     |
| 1.21      | 6.0.54  | fabric-loom-remap | 0.102.0    |
| 1.21.2    | 6.0.54  | fabric-loom-remap | 0.106.1    |
| 1.21.4    | 6.0.54  | fabric-loom-remap | 0.118.5    |
| 1.21.6    | 6.0.54  | fabric-loom-remap | 0.128.0    |
| 1.21.9    | 7.0.61  | fabric-loom-remap | 0.134.0    |
| 1.21.11   | 7.0.61  | fabric-loom-remap | 0.139.4    |
| 26.1      | 7.0.61  | fabric-loom       | 0.145.1    |
| 26.2      | 7.0.61  | fabric-loom       | 0.158.0    |

每个 MC 适配器的 `depends.minecraft` 使用 `>=<版本>` 无上界，Fabric Loader 在所有满足的候选适配器中选版本号最大的那个。  
这是有意设计——新增更高版本 MC 时**不需要修改已有适配器的上界**。  
MC 26.1+ 是非混淆版本（无 ProGuard），Loom 没有 `remapJar` 任务，`officialMojangMappings()` 会抛异常。

上表是 `gradle/adapters.toml` 的人类可读副本；改矩阵改 toml，然后同步此表。

## 硬性不变量

| 不变量                                                                                                                                                     | 为什么                                                                                                                                                                                                 | 违反后的症状                                                            | 强制机制                                                                                                    |
|------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| common 不得 import `net/minecraft/*` 或 `com/mojang/authlib/*`；`java/awt/*`、`javax/imageio/*` 已加入白名单（头像渲染位于 `common/utils/image` 包，允许） | common 被 12 个不同 MC 版本共用编译；AWT 头像渲染已迁回 common 的 `utils/image` 包（曾迁出到独立 `:core-image` 模块，P1.4 / 决策 D4），AWT 在 common 内白名单放行                                      | 编译失败（`checkArchitecture` 拒绝 net/minecraft / com/mojang/authlib） | `:checkArchitecture`（P0.5：net/minecraft、com/mojang/authlib 硬性禁止；java.awt/javax.imageio 白名单放行） |
| `MinecraftAdapterImpl` 类名是数据契约                                                                                                                      | 写在各适配器 `fabric.mod.json` 的 `accountsx:adapter.mc.class` 里                                                                                                                                      | 12 个适配器中找不到实现，模组崩溃                                       | 人工校验 + fabric.mod.json 校验                                                                             |
| `~/.accountsx/` 下的文件含明文 token                                                                                                                       | 所有日志、报错、提交都不得包含其内容                                                                                                                                                                   | token 泄露                                                              | 人工 review                                                                                                 |
| `depends.minecraft` 使用 `>=` 无上界                                                                                                                       | Loader 在候选中取版本号最大者；适配器版本为 `${minecraft}-${version}`，MC 版本是最左的版本核心段（数字逐位比较，短者补 0），而 CI 注入的 `+build.N` 落在被忽略的 build 段，不会破坏排序                | 版本排序被破坏时选错适配器 → mixin 目标不存在 → 崩溃                    | `:validateAdapterMatrix`（P0.5）                                                                            |
| 载荷读失败时**不得**写回配置                                                                                                                               | `initialize()` 末尾的无条件 `save()` 会把文件覆写成 `[]`，静默丢失用户全部账号                                                                                                                         | 一次瞬时 I/O 错误永久删除所有账号                                       | P1 修复（只读降级模式）                                                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dainsleif233/AccountsX](https://github.com/Dainsleif233/AccountsX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
