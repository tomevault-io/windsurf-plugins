---
trigger: always_on
description: <!-- 本文件由 .agents/bin/sync-agents-md.sh 从 .agents/rules/*.md 生成，请勿手动编辑。 -->
---

<!-- 本文件由 .agents/bin/sync-agents-md.sh 从 .agents/rules/*.md 生成，请勿手动编辑。 -->
<!-- 修改规则请编辑 .agents/rules/ 下对应文件后重跑该脚本。 -->

# SkyPlayer — 项目规则（始终生效）

> 技能（skills）位于 `.agents/skills/`，Codex 会原生扫描该目录，无需在此声明。


# SkyPlayer 代码风格规范

本项目遵循以下代码风格规范和约定：

## Kotlin 代码风格

### 命名约定
- **类名**: 使用 PascalCase，如 `SkyMediaPlayer`, `AudioFocusManager`
- **函数名**: 使用 camelCase，如 `setDataSource()`, `prepareAsync()`
- **常量**: 使用 UPPER_SNAKE_CASE，如 `MEDIA_PREPARED`, `MEDIA_ERROR`
- **变量**: 使用 camelCase，如 `mediaPlayer`, `surfaceHolder`

### 接口设计
- 播放器接口使用 `I` 前缀，如 `IMediaPlayer`
- 监听器接口使用 `On...Listener` 命名，如 `OnPreparedListener`, `OnErrorListener`

### 代码组织
- 每个类一个文件
- 相关类放在同一包下
- 包结构按功能划分：`player/`, `audio/`, `widget/`, `utils/`

## C/C++ 代码风格

### 命名约定
- **文件名**: 使用小写加下划线，如 `skymediaplayer.cpp`, `sky_msg_queue.cpp`
- **类名**: 使用 PascalCase，如 `SkyMediaPlayer`, `SkyRenderer`
- **函数名**: 使用小写加下划线，如 `sky_display_image()`, `sky_open_audio()`
- **宏定义**: 使用 UPPER_SNAKE_CASE，如 `SKY_MSG_PREPARED`
- **结构体**: 使用 PascalCase 或带 `Sky` 前缀，如 `SkyAudioSpec`

### 头文件规范
- 使用 `#pragma once` 或 include guards
- 公共头文件放在 `include/` 目录
- 私有头文件与源文件放在一起

### JNI 规范
- JNI 函数使用 `_` 前缀，如 `_native_setup`, `_setDataSource`
- 使用弱全局引用管理 Java 对象
- 使用 TLS 管理 JNIEnv
- 自动 attach/detach 线程

### 内存管理
- 使用 RAII 机制管理资源
- 智能指针优先于裸指针
- 正确处理 JNI 引用（Local/Global/Weak）

## 注释规范

### Kotlin/Java
```kotlin
/**
 * 播放器接口定义
 * 兼容 Android MediaPlayer API
 */
interface IMediaPlayer {
    /**
     * 设置数据源
     * @param path 文件路径或网络 URL
     */
    fun setDataSource(path: String)
}
```

### C/C++
```cpp
/**
 * 显示视频帧
 * @param player 播放器实例
 * @param frame 解码后的视频帧
 * @return 成功返回 true
 */
bool sky_display_image(void *player, AVFrame *frame);
```

## 异常处理

### Kotlin 层
- 使用 try-catch 捕获异常
- 通过 OnErrorListener 回调错误信息
- 资源释放使用 try-finally 或 use 扩展

### Native 层
- 函数返回错误码或布尔值
- 通过消息队列传递错误事件
- 使用 RAII 确保资源释放

## 线程安全

### 多线程设计
- 使用互斥锁保护共享资源
- JNI 层使用 TLS 管理 JNIEnv
- 消息队列实现异步事件处理

### 线程模型
- 读取线程：读取数据包
- 视频解码线程：解码视频帧
- 音频解码线程：解码音频帧
- 渲染线程：音画同步和渲染

## 性能考虑

### 视频渲染
- 使用 OpenGL ES 2.0 硬件加速
- 支持 5 种像素格式优化渲染
- Shader 实现 YUV → RGB 转换

### 音频输出
- 使用 OpenSL ES 实现低延迟（< 20ms）
- 多缓冲区设计保证流畅播放
- 实时线程优先级

### 内存优化
- 避免频繁内存分配
- 使用对象池复用资源
- 及时释放不再使用的资源

## 代码审查要点

1. **接口兼容性**: 确保与 Android MediaPlayer API 兼容
2. **线程安全**: 检查多线程访问的正确性
3. **资源管理**: 确保资源正确释放
4. **错误处理**: 检查错误路径的处理
5. **性能影响**: 评估对播放性能的影响


# SkyPlayer 主要功能模块

SkyPlayer 是一个基于 FFmpeg 8.0 官方 ffplay 深度改造的 Android 音视频播放器，提供从解封装、解码、同步到渲染的完整实现。

## 核心功能

### 1. 播放器核心

播放器核心是 SkyPlayer 的基础，提供标准的媒体播放 API。

**相关代码目录**:
- `skymediaplayer/src/main/java/imt/zw/skymediaplayer/player/`: Java/Kotlin 播放器接口和实现
- `skymediaplayer/src/main/cpp/player/skymediaplayer.cpp`: Native 播放器核心
- `skymediaplayer/src/main/cpp/ffplay/ffplay.c`: FFmpeg ffplay 引擎

**主要特点**:
- 兼容 Android MediaPlayer API
- 支持本地文件和网络流媒体
- 异步准备和播放控制
- 完整的播放状态管理
- 精确的 Seek 功能

**核心接口**:
```kotlin
interface IMediaPlayer {
    fun setDataSource(path: String)
    fun prepareAsync()
    fun start()
    fun pause()
    fun stop()
    fun seekTo(msec: Long)
    fun release()
}
```

### 2. 视频渲染

视频渲染模块使用 OpenGL ES 2.0 实现硬件加速渲染，支持多种像素格式。

**相关代码目录**:
- `skymediaplayer/src/main/cpp/player/skyrenderer.cpp`: 渲染器管理
- `skymediaplayer/src/main/cpp/player/sky_egl2_renderer_*.cpp`: 5 种像素格式渲染器

**支持的像素格式**:
- **YUV420P**: 最常见的视频格式
- **YUV422P**: 高质量视频格式
- **NV12**: Android 硬件解码常用格式
- **NV21**: Android Camera 常用格式
- **RGBA**: 通用图像格式

**主要特点**:
- OpenGL ES 2.0 硬件加速
- Shader 实现 YUV → RGB 色彩空间转换
- 10-100x 比 CPU 渲染更快
- 支持 4K+ 分辨率
- 低功耗

### 3. 音频输出

音频输出模块使用 OpenSL ES 实现超低延迟音频播放。

**相关代码目录**:
- `skymediaplayer/src/main/cpp/player/skyaudio.cpp`: OpenSL ES 音频输出
- `skymediaplayer/src/main/java/imt/zw/skymediaplayer/audio/`: 音频焦点管理

**主要特点**:
- OpenSL ES 低延迟（< 20ms）
- 实时线程优先级
- 多缓冲区设计
- 音频焦点管理

**支持的音频格式**:
- AAC、MP3、Opus、Vorbis
- 采样率：8kHz - 192kHz
- 声道：单声道/立体声

### 4. JNI 桥接层

JNI 层实现 Java/Kotlin 与 Native 代码的通信。

**相关代码目录**:
- `skymediaplayer/src/main/cpp/skymediaplayer_jni.cpp`: JNI 桥接实现

**主要特点**:
- 12 个 Native 方法注册
- TLS 管理 JNIEnv
- 自动 attach/detach 线程
- 弱全局引用防止内存泄漏
- 线程安全设计

**JNI 方法列表**:
```cpp
_native_setup()
_setDataSource(String)
_prepareAsync()
_start()
_pause()
_stop()
_seekTo(long)
_release()
// ... 更多方法
```

### 5. 消息队列

消息队列实现异步事件处理和状态通知。

**相关代码目录**:
- `skymediaplayer/src/main/cpp/player/sky_msg_queue.cpp`: 消息队列实现
- `skymediaplayer/src/main/cpp/include/sky_messages.h`: 消息定义

**主要特点**:
- 异步事件处理
- 解耦播放控制和状态通知
- 线程安全的事件传递
- 支持多种消息类型

**消息类型**:
- `MEDIA_PREPARED`: 准备完成
- `MEDIA_PLAYBACK_COMPLETE`: 播放完成
- `MEDIA_BUFFERING_UPDATE`: 缓冲更新
- `MEDIA_SEEK_COMPLETE`: Seek 完成
- `MEDIA_ERROR`: 错误通知

## 网络支持

### 支持的协议
- **HTTP/HTTPS**: 标准在线视频点播
- **HLS (m3u8)**: 自适应码率直播流
- **本地文件**: 支持本地存储的所有格式

### 技术特性
- OpenSSL 集成支持 HTTPS 加密传输
- HLS 自适应码率切换
- 智能网络缓冲策略
- 断点续播支持

## 支持的格式

### 视频容器
- MP4、AVI、MKV、WebM、MOV

### 视频编码
- H.264、H.265/HEVC、MPEG-4、MPEG-2、VP8、VP9

### 音频编码
- AAC、MP3、Opus、Vorbis

## UI 组件

### SkyVideoView

完整的视频播放视图组件。

**相关代码目录**:
- `skymediaplayer/src/main/java/imt/zw/skymediaplayer/widget/SkyVideoView.kt`

**主要特点**:
- 封装播放器和渲染视图
- 自动处理 Surface 生命周期
- 支持多种缩放模式

### SurfaceRenderView

Surface 渲染视图。

**相关代码目录**:
- `skymediaplayer/src/main/java/imt/zw/skymediaplayer/widget/SurfaceRenderView.kt`

**主要特点**:
- 管理 SurfaceHolder
- 处理 Surface 创建和销毁
- 视频尺寸适配

## 示例应用

app 模块提供完整的使用示例。

**相关代码目录**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhiwei-wu/SkyMediaPlayer](https://github.com/zhiwei-wu/SkyMediaPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
