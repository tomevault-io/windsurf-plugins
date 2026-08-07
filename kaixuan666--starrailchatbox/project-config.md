---
trigger: always_on
description: - 若子目录中存在更具体的 `AGENTS.md`，则以距离目标文件最近的规则为准。
---

# StarRailChatBox Agent Guide

## 适用范围与指令优先级

- 本文件适用于整个仓库。
- 若子目录中存在更具体的 `AGENTS.md`，则以距离目标文件最近的规则为准。
- 用户当前任务中的明确要求优先于本文件；发生冲突时应指出冲突并遵循更高优先级要求。
- 开始修改前先检查相关源码、构建脚本、测试和 `git status`，不要覆盖或回退用户已有的未提交改动。
- 读取已知具体文本文件时，使用：

  ```powershell
  Get-Content -LiteralPath "<path>" -Encoding UTF8
  ```

## 项目概览

StarRailChatBox 是使用 Kotlin Multiplatform 和 Compose Multiplatform 构建的多端项目。

主要模块：

- `shared/`：共享业务逻辑、状态管理、Compose UI 和多平台资源。
- `androidApp/`：Android 应用入口与 Android 专属配置。
- `desktopApp/`：Desktop/JVM 应用入口与桌面打包配置。
- `webApp/`：JS 与 WasmJS Web 应用入口。
- `iosApp/`：iOS/Xcode 应用入口，加载 `Shared` framework。

`shared` 当前目标平台：

- Android
- iOS Arm64
- iOS Simulator Arm64
- Desktop/JVM
- JavaScript
- WasmJS

除非平台能力确实不同，功能、业务逻辑和 UI 应优先实现在
`shared/src/commonMain`。

## 技术栈

新增数据层、网络层、依赖注入、存储和加密功能时，优先使用以下统一技术栈，
不要为相同职责重复引入其他框架：

- 网络请求：Ktor Client。
- JSON：`kotlinx.serialization`，并通过 Ktor `ContentNegotiation` 集成。
- Android HTTP 引擎：Ktor OkHttp Engine，依赖仅放在 Android 源码集。
- iOS HTTP 引擎：Ktor Darwin Engine，依赖仅放在 iOS 源码集。
- Desktop、JavaScript 和 WasmJS HTTP 引擎：选择 Ktor 官方支持对应目标的引擎，
  并保持上层 `HttpClient` 配置和接口共享。
- 接口封装：Ktorfit。接口定义和可共享转换逻辑放在公共源码集。
- 错误封装：项目自定义不可变 `ApiResult`，统一表达成功、业务错误、网络错误和
  未预期错误；不得把 Ktor、OkHttp 或平台异常直接暴露给 UI。
- 日志：Ktor Logging + Napier。Ktor 日志通过自定义 `Logger` 转发给 Napier，
  并脱敏 `Authorization`、Cookie、令牌及其他敏感请求头和请求体。
- 依赖注入：Koin。公共模块声明放在 `commonMain`，需要平台对象的绑定放在对应
  平台源码集，并由各应用入口初始化。
- 权限管理：MOKO Permissions。用于跨平台（Android/iOS）请求麦克风等敏感权限。
- 文件选择与处理：FileKit。提供跨平台的文件/图片选择器，并支持与 Coil 集成。
- 轻量键值存储：DataStore Preferences KMP。
- I/O 操作：Okio。用于跨平台文件流读写和文件系统抽象。
- 加密：`cryptography-kotlin`，优先使用适合目标平台的 provider，不自行实现
  密码算法。
- 图片加载：Coil 3 Compose Multiplatform。公共 UI 使用共享头像/图片组件加载
  `avatarUri`、资源 URI、文件路径或浏览器 `data:` URI，不在业务 Composable 中
  手动解码头像字节。
- 结构化数据库：Room KMP。Room 实体、DAO、数据库和共享实现放在仅供 Android、
  iOS、Desktop/JVM 使用的 `shared/src/roomMain`；数据库路径、Builder、SQLite
  driver、私有文件目录和其他平台配置放在对应平台源码集。

### KMP 支持边界

以上技术选型已按本项目 Android、iOS、Desktop/JVM、JavaScript 和 WasmJS 目标
核对，但“KMP”不代表自动覆盖全部目标。引入具体版本时仍必须重新检查其发布构件
与当前 Kotlin、KSP、Ktor 和 Compose 版本的兼容性。

| 技术 | Android | iOS | Desktop/JVM | JavaScript | WasmJS | 使用约束 |
| --- | --- | --- | --- | --- | --- | --- |
| Ktor Client | 支持 | 支持 | 支持 | 支持 | 支持 | 引擎按平台配置；OkHttp 仅用于 Android/JVM，Darwin 仅用于 Apple Native |
| `kotlinx.serialization` JSON | 支持 | 支持 | 支持 | 支持 | 支持 | JSON 模型与配置可放入 `commonMain` |
| Ktorfit | 支持 | 支持 | 支持 | 支持 | 支持 | 使用前核对 KSP、Kotlin 与 Ktorfit 版本矩阵 |
| 自定义 `ApiResult` | 支持 | 支持 | 支持 | 支持 | 支持 | 仅使用公共 Kotlin 类型，不包含平台异常类型 |
| Ktor Logging | 支持 | 支持 | 支持 | 支持 | 支持 | 生产环境限制级别并强制脱敏 |
| Napier | 支持 | 支持 | 支持 | 支持 | 未明确支持 | 不得直接加入会破坏 WasmJS 编译的公共源码集；WasmJS 使用公共日志抽象和兼容实现 |
| Koin | 支持 | 支持 | 支持 | 支持 | 支持 | 平台对象通过平台模块绑定 |
| MOKO Permissions | 支持 | 支持 | 不支持 | 不支持 | 不支持 | 仅用于 Android 和 iOS；其他平台通过平台能力探测跳过权限请求 |
| FileKit | 支持 | 支持 | 支持 | 支持 | 支持 | 优先使用 `FileKit.pickFile` 或 `pickImage` 进行多媒体输入 |
| DataStore Preferences KMP | 支持 | 支持 | 支持 | 不支持 | 不支持 | 仅用于 Android、iOS 和 Desktop；Web 通过公共存储接口接入浏览器实现 |
| Okio | 支持 | 支持 | 支持 | 支持 | 支持 | 用于文件系统操作和测试 mock |
| `cryptography-kotlin` | 支持 | 支持 | 支持 | 支持 | 支持 | provider 和算法支持可能因平台不同，使用前逐项确认 |
| Coil 3 Compose | 支持 | 支持 | 支持 | 支持 | 支持 | 公共 UI 使用 `avatarUri` 等可加载模型；引入或升级时需检查 Compose/Skiko 兼容警告 |
| Room KMP | 支持 | 支持 | 支持 | 不支持 | 不支持 | 仅用于 Android、iOS 和 Desktop；Web 通过公共仓库接口使用独立持久化实现 |

不得为了接入 DataStore、Room 或 Napier 而删除或禁用 JavaScript/WasmJS 目标，也不得
把不支持 Web 的构件加入 Web 可见的源码集。公共业务层应依赖项目自有接口，由
Android、iOS、Desktop 和 Web 分别提供实现；平台差异只停留在数据源创建、引擎、
driver、provider 和日志出口等边界。

### 跨平台代码编写规范
- 编写跨平台代码前，先检测已经依赖的第三方库是否支持所需功能，如果支持，优先使用第三方库，不要直接编写跨平台代码。
- 编写跨平台代码前，如果有比较成熟的第三方库，优先使用第三方库，不要自行实现功能。

### 文件处理规范
- 用户选择头像或文件时，优先使用 FileKit，文件读写采用Okio 封装类 `KmpFileManager`，对于文件操作，不要自己写各平台实现。
- 对于 Android、iOS 和 Desktop，将返回的 `PlatformFile` 通过 `KmpFileManager` 处理并复制到应用私有目录。
- 对于 JS/WasmJS，直接使用 `PlatformFile` 提供的 Data URL (Base64) 或内存 URI，避免复杂的平台持久化。
- 注意FileKit选择出来的文件，可能路径会很奇怪，比如“content://media/picker/0/com.android.providers.media.photopicker/media/1000125767”
  所以拿到FileKit返回的path后，除了path，我们还需要传递image.name 和 image.extension给调用方，避免调用方拿不到扩展名，导致存储下来无扩展名的文件。
- 修改头像或删除角色时，应清理由应用私有目录下的旧头像文件。
- 头像文件名必须由角色 ID 经过跨平台安全编码生成，不直接使用角色名、中文、
  冒号或路径分隔符作为文件名。
- 内置角色资源来自 `shared/src/commonMain/composeResources/files/characters`。
  `DefaultCharacterRepository` 首次加载时将缺失角色幂等导入 `agent_role`；已存在
  或已软删除的相同 ID 不得被重新导入或覆盖头像。
- JS/WasmJS 不使用 Room，当前角色使用浏览器存储并以可加载的 `data:` URI 表示头像，
  模型配置使用内存实现；公共 UI 和业务代码不得直接依赖 Room 实体、DAO 或平台文件路径。

#### 两阶段落盘法 (Cache ➔ Files)
当出现用户选择文件 → 后续可以保存入库(uri入库)的情况时，（比如 `AgentRoleEntity` 中的 `avatarUri`和 `voiceSampleUri`，`MessageAttachmentEntity`里面的uri，还有 `AttachmentPanel`，用户选择添加附件）
应采用“临时缓存 -> 确认入库”的策略：
1. **阶段一（选择即缓存）：** 当用户选完文件后，立即将文件拷贝到 App 沙盒的【临时缓存目录 (Cache Dir)】中，并生成一个临时路径。
2. **阶段二（确认才入库）：** 只有当用户在 UI 上真正点击了“保存/提交”按钮时，将文件从【临时缓存目录】移动到【正式私有目录 (Files Dir)】，并完成数据库入库。


### Room 数据库
- 数据库统一使用 `StarRailDatabase`。Android、iOS 和 Desktop 启动时只创建一个
  Room 实例，并从该实例提供模型配置、角色、会话和消息相关 DAO 或 Repository；
  不要为不同 Repository 重复打开同一路径的数据库。
- 当前数据库表为 `agent_role`、`chat_session`、`chat_message`、`chat_summary`
  和 `model_config`。实体和 DAO 位于 `shared/src/roomMain`，不得移动到
  JS/WasmJS 可见的源码集。
- `chat_summary` 保存滚动摘要及其覆盖的消息序号范围；原始消息继续保留。默认在
  未压缩有效消息达到 30 条时后台总结，并保留最近 10 条原文。
- `agent_role` 的头像内容不保存为数据库 BLOB。Android、iOS 和 Desktop 应先将
  头像复制到应用私有目录 `character_avatars`，数据库仅在 `avatar_uri` 中保存

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaiXuan666/StarRailChatBox](https://github.com/KaiXuan666/StarRailChatBox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
