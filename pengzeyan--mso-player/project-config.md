---
trigger: always_on
description: 本文件适用于仓库根目录及其全部子目录。若某个子目录以后出现更具体的 `AGENTS.md`，则该文件只覆盖其所在目录及后代；系统指令、开发者指令和用户当前请求始终高于本文件。
---

# AGENTS.md

## 适用范围与优先级

本文件适用于仓库根目录及其全部子目录。若某个子目录以后出现更具体的 `AGENTS.md`，则该文件只覆盖其所在目录及后代；系统指令、开发者指令和用户当前请求始终高于本文件。

所有工作都应遵循以下基本原则：

- 只修改完成当前任务所必需的文件，不顺手重构无关代码。
- 开始前先读取 `git status --short` 和相关文件的现有差异，保留用户的未提交修改。
- 诊断、审查和“读取代码”类请求默认只读；只有明确要求实现或修复时才写文件。
- 不以“能够编译”代替 Unity Play Mode、播放器构建或真实媒体播放验证。
- 不隐瞒未执行的验证；最终说明必须区分已验证、静态推断和待验证内容。

## 项目定位

MSO-Player 是一个 MIT 许可的 Unity/libVLC 播放器项目，提供：

- `RawImage` 上的普通 2D 视频播放。
- `MeshRenderer`/球体材质上的 360 度全景视频播放。
- 本地媒体以及 HTTP、HLS、RTSP、RTMP 等 LibVLC 可处理的媒体源。
- 播放、暂停、停止、进度、音量、切流、状态事件和播放器对象池。
- Windows x86_64 与 Android ARMv7/ARM64 原生依赖。

Unity 编辑器版本以已提交的 `ProjectSettings/ProjectVersion.txt` 为准。截至本文件创建时，Git 基线为 `2022.3.33f1c1`。若工作区文件显示其他版本，应先视为本地未提交的编辑器迁移，不能擅自纳入提交。

仓库内置 Windows `libvlc.dll` 的文件版本为 3.0.19。修改 P/Invoke、原生插件或构建复制逻辑时，必须以实际打包版本的 ABI 为准。

## 事实来源

按以下顺序判断项目当前事实：

1. 当前源码、Unity YAML、插件导入设置和 Git 差异。
2. `ProjectSettings/`、`Packages/manifest.json` 与 `Packages/packages-lock.json`。
3. `README.md`、`README_EN.md` 和 `Docs/QuickStart.md`。
4. 运行日志和历史说明。

文档中的平台声明不等于已打包支持。当前仓库实际提供 Windows x86_64 DLL 和 Android ARMv7/ARM64 `.so`；没有看到可直接发布的 Linux、macOS、iOS 或 WebGL 原生实现。只有在补齐对应原生库、导入设置、构建流程并完成目标平台运行验证后，才可扩大支持声明。特别注意：Windows DLL 的 `.meta` 勾选其他平台不能让该 DLL 在那些平台运行。

`Assets/Player.log`、`Logs/` 和外部客户端导出的日志只能作为指定运行的证据，不能自动代表当前仓库、当前提交或当前设备状态。

## 仓库结构

- `Assets/MSO-Player/Scripts/Core/`
  - `LibVLCWrapper.cs`：LibVLC 3.x P/Invoke、枚举、结构体和回调委托。
  - `VlcMediaPlayer.cs`：原生实例、媒体、播放器、视频回调、RGB24 缓冲、切流和释放。
  - `ExtendedVlcPlayer.cs`：时间、位置、音量、直播判断和分辨率等扩展控制；通过核心层受控 API 访问播放器状态。
- `Assets/MSO-Player/Scripts/Platform/`
  - `MediaPlayer.cs`：面向 Unity UI 的 2D 播放组件。
  - `MediaPlayer360.cs`：面向球体/材质的全景播放组件。
  - `MediaPlayerAndroid.cs`：Android 参数、硬解与低内存处理。
  - `PlatformManager.cs`：平台和设备能力判断。
- `Assets/MSO-Player/Scripts/Pool/`：播放器池、启动器和预热逻辑。
- `Assets/MSO-Player/Scripts/PlayerControl/`：示例播放器 UI、进度和交互控制。
- `Assets/MSO-Player/Scripts/Utils/`：加载提示、360 相机和调试监视器。
- `Assets/MSO-Player/Editor/`：自定义 Inspector 和 Windows 构建后处理。
- `Assets/MSO-Player/Plugins/`：Windows 与 Android 原生依赖。二进制及其 `.meta` 都是发布链路的一部分。
- `Assets/MSO-Player/Prefab/`、`Assets/MSO-Player/Scene/`：示例 Prefab 和场景。
- `Assets/MSO-Player/Materials/`、`Assets/MSO-Player/Sprites/`：示例渲染和 UI 资源。
- `Docs/`、`README.md`、`README_EN.md`：用户文档。
- `Packages/com.unity.asset-store-tools/`：随项目放置的第三方/工具包代码。除非任务明确涉及它，否则不要修改。
- `Library/`、`Temp/`、`Obj/`、`Logs/`、`Build/`、`Builds/`、`UserSettings/`：Unity 生成目录，不得提交或作为源码修改目标。

项目自身目前没有独立 asmdef，也没有一套第一方自动化测试。Unity/IDE 生成的 `.sln` 和 `.csproj` 被忽略，不能当作稳定的仓库输入。

## 运行时数据流

正常的 2D 播放链路如下：

1. `MediaPlayer.SetUrl()`/`Play()` 取得或创建 `VlcMediaPlayer`。
2. `VlcMediaPlayer` 创建 `libvlc_instance_t`、`libvlc_media_t` 和 `libvlc_media_player_t`。
3. `libvlc_video_set_callbacks()` 注册 lock、unlock 和 display 回调，`libvlc_video_set_format()` 要求输出 `RV24`。
4. LibVLC 解码线程把一帧写入 `_imageIntPtr` 指向的非托管内存。
5. display 回调将帧复制到托管后备缓冲，并在锁内交换前后缓冲。
6. Unity 主线程的 `Update()` 调用 `CheckForImageUpdate()`，再用 `LoadRawTextureData()` 和 `Apply(false)` 上传到 `Texture2D`。
7. `RawImage.uvRect` 或材质纹理缩放负责垂直翻转。

音频由 LibVLC 直接管理，不经过 Unity `AudioSource`。播放器状态由协程轮询 LibVLC 状态并转换为 Unity 事件。普通播放器可从 `MediaPlayerPool` 复用；360 播放器当前直接持有和释放自己的核心实例。

## 核心不变量

### 原生互操作

- P/Invoke 函数名、参数宽度、返回值、结构体字段顺序和回调签名必须与仓库实际 LibVLC 版本匹配。
- 不凭记忆修改 ABI。涉及 ABI 时，应核对对应版本的官方头文件或导出符号，并至少验证 Windows 和一个 Android ABI。
- 原生回调委托必须在播放器整个原生生命周期中保持强引用；不得把短生命周期 lambda 直接交给 LibVLC。
- `opaque`、`GCHandle` 和播放器实例映射必须同生共灭。解除映射或释放 `GCHandle` 前，应确保原生回调已经停止。
- 新增字符串互操作时明确字符编码和所有权；不得释放 LibVLC 所有的指针，也不得遗忘释放调用方所有的分配。

### 线程模型

- Unity 对象、场景、组件、材质、纹理、协程和 `Time` API 只在 Unity 主线程使用。
- LibVLC 视频回调和轨道读取线程不能直接修改 Unity 对象。
- 跨线程共享的字典、标志、媒体指针和缓冲区必须具备明确的同步或所有权。`volatile` 不能替代复合操作所需的锁。
- 不在持锁期间调用可能阻塞、回调用户代码或进入 Unity 的操作。
- 后台线程必须能取消，并在释放相关原生指针和托管缓冲前确认退出；不要依赖后台线程随进程退出。
- 主线程分发器不得假定托管线程 ID 永远等于 1。若修改这一部分，应在 Unity 启动时捕获真实主线程或同步上下文。

### 帧缓冲与纹理

- `RV24` 固定为每像素 3 字节，pitch 为 `width * 3`。缓冲区大小、pitch、纹理格式和上传长度必须一致。
- 对宽高和乘法做合法性/溢出检查，拒绝零值、负值和不合理的大分辨率。
- 分辨率发生变化时，必须作为一个受控生命周期操作同时更新：托管前后缓冲、非托管帧内存、LibVLC 视频格式和 Unity 纹理。不得让回调写入旧尺寸内存。
- 回调热路径和 `Update()` 中避免逐帧分配、LINQ、反射和重复日志。
- `Texture2D.Apply(false)` 仍有 CPU 到 GPU 上传成本。性能改动必须用 Profiler/目标设备数据验证，不能只凭代码注释声称“零开销”或“超高性能”。
- CPU 垂直翻转会原地修改共享帧数组；若其他消费者也读取该帧，必须先明确所有权。

### 媒体与资源所有权

- 每次成功创建 `libvlc_media_t` 都必须有清晰的释放点。
- 切换 URL 时要保持“播放器当前媒体、`_media` 字段、轨道元数据和扩展播放器指针”一致；旧媒体和轨道信息应在不再使用后及时释放。
- `libvlc_media_tracks_get()` 的每次成功返回都应与一次 `libvlc_media_tracks_release()` 配对，不能只保存最后一次返回值。
- 推荐的销毁顺序是：停止新的工作和恢复协程，停止播放，确保回调/工作线程静止，释放轨道结果，释放 media player，释放 media，释放 LibVLC 实例，释放非托管帧内存，最后移除实例映射和 `GCHandle`。
- `Dispose()` 必须幂等。释放后的实例不能再次进池或继续响应回调。
- Unity 资源使用 `Destroy()`，不要在运行时使用 `DestroyImmediate()`。

### 播放状态与事件

- 区分 `Play`、`Resume`、`Pause`、`TogglePause`、`Stop` 和 `Reload`；不要依赖模糊的“如果没在播放就切换”行为。
- 一个用户操作对应的 `OnPlayEvent`、`OnStopEvent` 和错误事件应只触发一次。修改播放流程时检查手动触发和状态监控是否重复。
- LibVLC 构造完成、`libvlc_media_player_play()` 返回成功、状态变为 Playing、收到首帧是四个不同阶段；日志和 UI 不能混为一谈。
- 网络流短暂处于 Opening/Buffering 不等于失败。恢复策略应有单实例互斥、退避、次数上限和取消条件。
- 不要从 `libvlc_errmsg()` 的全局/线程局部最后错误推断任意旧操作的失败原因；尽量在失败调用附近读取并记录上下文。

### 对象池

- 入池实例必须停止播放但仍保持可复用，且不能已经 `Dispose()`。
- 出池后应重置 URL、媒体、轨道、缓冲状态、静音/音量、错误计数和首帧时间等所有会跨使用者泄漏的状态。
- 池键必须覆盖影响实例兼容性的配置。若平台、色度、VLC 参数或硬解设置不同，应扩展池键或禁用复用。
- 活动列表和可用队列中同一实例只能出现一次。
- 清理池时不得销毁仍被组件、回调或后台线程使用的播放器。
- 对池的改动必须在多播放器场景中验证，而不只是单播放器示例。

## 平台约束

### Windows x86_64


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PengZeYan/MSO-Player](https://github.com/PengZeYan/MSO-Player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
