---
trigger: always_on
description: 本文件面向**开发者与 AI 编码代理**：项目是什么、边界在哪、怎么改、怎么验、怎么发。
---

# AGENTS.md — 开发与维护规范

本文件面向**开发者与 AI 编码代理**：项目是什么、边界在哪、怎么改、怎么验、怎么发。
**面向用户的使用说明不放这里，在 [README.md](README.md)。**

任何实现、重构、审查都必须遵守本文件。与最新 Android 强制安全要求冲突时以平台要求为准；
若新要求会突破第 2 节的安全边界，**停止实现并明确报告，不得自行扩大权限**。

---

## 1. 项目是什么

系统启动助手（NetBoot）是一个 Android 应用（API 26–37），让**已由 KernelSU 明确授权本应用使用 `su`** 的手机提供：

1. **PXE 网络启动服务** —— 通过手机现有局域网接口，用 DHCP / ProxyDHCP + TFTP + HTTP Boot + iPXE 给同一网络内的 PC 提供网络引导。
2. **USB 安装介质** —— 把应用私有目录中的镜像临时映射成只读 USB 存储，供 PC 引导。
3. **镜像获取与管理** —— 微软官方 ISO 临时链接获取、多连接断点下载、本地导入、SHA-256 与状态管理。

USB 数据流：`下载或导入镜像 → 制作或复用介质 → 启用 → 使用 → 停止并恢复 USB`。
PXE 数据流：`选择接口与模式 → 配置启动文件和脚本 → 启动 → 使用 → 停止`。
镜像库与 PXE 文件目录独立，下载或导入 ISO 不会自动生成 PXE 安装源。

必须做成可发布、可持续维护的正式产品：**不接受 Demo、原型、占位实现、假数据、只覆盖成功
路径或需要开发者手工收尾的流程。**

### 1.1 Root 前提（不可含糊）

| 状态 | 本项目 |
| --- | --- |
| **KernelSU 已授权本应用**（App Profile / 授权列表允许 `com.sky22333.netboot` 使用 `su`） | ✅ 唯一受支持配置 |

应用启动时自动检测现有 Root 和 USB 配置。权限不可用时提示检查 KernelSU 授权，
**不尝试安装或修补 KernelSU**。

---

## 2. 安全边界（最重要的一节）

这条边界是产品验收标准的一部分：**允许内核 panic 重启，但手机必须始终能正常开机与使用。**

### 允许

- 应用私有目录中的数据库、配置、日志、下载临时文件与镜像。
- 用户通过 SAF 明确选择的目标，仅限已确认的导入/导出。
- configfs 中**经白名单限定**的 USB gadget 节点。这是内核内存态、重启即复位，不是分区内容。

### 绝对禁止

- 写入/删除/覆盖/格式化/刷写任何真实分区块设备，尤其 `/dev/block/*`。
- 写或可写挂载 boot、init_boot、vendor_boot、recovery、abl、xbl、dtbo、vbmeta、
  system、vendor、product、super、odm、persist、modem、metadata 等分区或数据。
- 修改 `/system`、`/vendor`、`/product`、`/system_ext`、`/odm`、`/metadata`、`/persist`、
  `/data/system`、`/data/misc`、`/data/vendor` 等目录。
- 关闭 AVB / dm-verity / SELinux，设置全局 permissive，改启动镜像、装内核、修补 Boot、解 BL。
- 安装 KernelSU 模块、metamodule、OverlayFS 或任何开机脚本。
- 调用 `dd`、`mkfs`、`flash*`、`setenforce 0`、`mount -o rw`、`resetprop` 或等价操作。
- 写 `persist.*` 属性；不修改 Android USB 持久配置。

### 强制门禁

- **任何特权路径写入（包含从状态文件读回的路径）必须先过 `ConfigfsGuard` 的 canonical
  校验**；校验失败一律拒绝执行，**不得降级为"尽力而为"**。
- native 侧（`app/src/main/cpp/media.cpp`）通过调用方传入、验证为普通文件的描述符读写输入与输出，
  临时文件仅在调用方校验后的私有工作目录生成。`disk_initialize` 拒绝非 0 的 drive，
  构建入口断言 `getuid() != 0`；媒体制作全程在**非 root 的应用进程**内完成。
- 设备若无法在上述边界内完成能力，必须明确显示"此设备不支持"。

---

## 3. 技术基线

| 项 | 值 |
| --- | --- |
| minSdk / compileSdk / targetSdk | 26 / 37 / 37 |
| Kotlin & JVM target | 2.4.20 / JVM 17 |
| CI 构建 JDK | 25 |
| AGP / Gradle | 9.4.0 / 9.7.1 |
| NDK（AGP 与 gomobile 共用同一常量） | 28.2.13676358 |
| Go | 1.27.1 |
| 发布 ABI | `arm64-v8a`、`armeabi-v7a`（`x86_64` 仅供模拟器） |

Android 依赖版本**只在 `gradle/libs.versions.toml` 声明**；Go 依赖由 `core-go/go.mod` 固定，
native 源码下载地址与哈希集中在 `app/src/main/cpp/dependencies.cmake`。禁止 `+`、
`latest.release`、SNAPSHOT、Alpha、Beta、RC、EAP 与预发布组件。

升级依赖：核实官方最新**稳定**版 → 读目标版本源码/迁移说明 → 单独提交并更新元数据 →
跑完整门禁（第 6 节）。**不为了"最新"强行拼出二进制不兼容的组合**；不兼容时保留最近一个
已验证稳定组合并在提交信息里写明证据。

---

## 4. 代码结构

Gradle 仅包含 `:app`；`core-go` 是独立 Go 模块，`buildSrc` 提供构建任务。
**禁止**新增 `core`/`common`/`utils`/`domain`/`base` 这类无业务所有权的抽象模块。

```text
app/src/main/java/com/sky22333/netboot/
  MainActivity.kt     单 Activity + Compose，四个一级页面 + 脚本编辑器
  MainViewModel.kt
  PxeFormState.kt     接口、地址池与脚本的编辑状态
  data/              Room、DataStore、镜像与启动文件、微软目录、媒体布局
  download/          下载仓库、前台服务与远端文件探测
  root/              Root Broker、configfs 白名单、USB 控制
  runtime/           会话编排、前台服务与 DHCP 地址池
app/src/main/cpp/    media.cpp、CMakeLists.txt、dependencies.cmake
app/schemas/com.sky22333.netboot.data.AppDatabase/1.json
core-go/mobilecore/  Go 核心与 gomobile 导出（DHCP/TFTP/HTTP/事件/路径校验）
buildSrc/           Go AAR 构建与 ABI 校验任务
gradle/libs.versions.toml
.github/workflows/    发布流水线
```

### 数据流与职责

```text
Miuix Screen → MainViewModel → Repository / Service
                               ├─ Room / DataStore / OkHttp
                               ├─ DownloadService
                               └─ Root Broker → Go AAR / configfs
                            ← StateFlow（运行状态、配置、镜像、任务、事件）
```

- UI 只渲染不可变 `UiState` 并上报事件；Composable 无副作用。
- ViewModel 不持有 Activity / Service / View / NavController / 可变 Context。
- **Repository 是业务数据唯一写入口**；Room 是任务、镜像、事件记录的事实来源；
  DataStore 只放轻量偏好（主题、下载连接数）。
- Service 不是数据源，运行状态写回 `RuntimeRepository`。
- 一个屏幕一个 `UiState`；不要维护多个互相推导的 `MutableStateFlow`。
- 单次导航、Snackbar 等用明确的事件流，不把已消费事件永久留在 StateFlow。
- **默认不加 Domain/UseCase 层**；仅当同一段非平凡逻辑被两个以上 ViewModel 真实复用时才提取。

---

## 5. 关键实现约束

### 5.1 Go 核心

`core-go/mobilecore` 只暴露 gomobile 稳定支持的窄接口：

```text
ValidateConfig(configJson) -> error
Start(configJson, listener) -> error
Stop() -> error
StatusJSON() -> String
```

iPXE 脚本作为 `config.ipxeScript` 随配置传入；修改配置后需重新启动 PXE 才生效。

- 不跨 JNI 暴露 Go struct / map / channel / context / 文件对象。
- Listener 只发**事件码 + 结构化参数**，不发成品文本；由 Kotlin 本地化。
- 高频进度与日志必须合并，禁止每包/每块跨 JNI 回调。
- `Start` 在已运行时返回错误，不隐式重启。`Stop` 可重复调用，取消根 context、关闭 socket 与
  HTTP server 并等待 goroutine 退出。
- Go 核心不认识 Android UI、Room、下载任务或本地化。
- 协议要求：DHCP 两种互斥模式；启动时仅校验配置与绑定端口，**不探测现有 DHCP 服务器、不绑定客户端 UDP 68 端口**；完整 DHCP 由用户确认在隔离网络使用；TFTP 支持 RRQ/重传/
  `blksize`/`tsize` 与并发上限；TFTP 与 HTTP 路径必须 clean+canonical+根边界校验，拒绝
  `..`、绝对路径与符号链接越界；HTTP Boot 支持 HEAD/Range 且**不得把整文件读入内存**。
- 脚本以协议保留名下发（`autoexec.ipxe`、`boot.ipxe`），两者都返回同一份 `config.ipxeScript`。
- 两种 DHCP 模式均监听 UDP 67/4011；PXE 启动发现先于租约处理。仅服务默认启动类型 0、层 0，
  明确的启动项目请求须返回对应的 option 43/71；4011 允许省略该项目。代理模式不抢答普通租约请求。
  `config.bootFile` 留空＝按 option 93 架构选择内置镜像，显式填写＝强制使用；EFI32 与未上报
  架构没有内置镜像，此时不下发启动文件名并记录 `boot_file_unsupported`。

### 5.2 Root Broker

Broker 与主应用在**同一个签名 APK** 内，由 `RootBrokerClient` 启动一个

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sky22333/netboot-android](https://github.com/sky22333/netboot-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
