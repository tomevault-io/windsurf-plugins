---
trigger: always_on
description: 本文档为 AI 编程助手提供 ScreenPresenter 项目的开发规范和上下文指南。
---

# AGENTS.md - AI 助手开发指南

本文档为 AI 编程助手提供 ScreenPresenter 项目的开发规范和上下文指南。

## 📋 项目概述

**ScreenPresenter** 是一款 macOS 原生设备投屏工具，支持同时展示 iOS 和 Android 设备屏幕，具备仿真设备边框渲染效果。

### 核心特性
- 📱 **iOS 投屏**: QuickTime 同款路径 (CoreMediaIO + AVFoundation)
- 🤖 **Android 投屏**: scrcpy 码流 + VideoToolbox 硬件解码
- 🖥️ **Metal 渲染**: CVDisplayLink 驱动的 60fps 高性能渲染
- 🔄 **双设备展示**: 支持同时展示两台设备
- 📐 **仿真边框**: 根据真实设备型号绘制设备外观
- 🎛️ **纯 AppKit**: 零 SwiftUI 依赖
- 🌐 **多语言**: 中英文双语支持

### 系统要求
- macOS 13.0+
- Xcode 16.4+
- Swift 6.0+

### 发布版本约定
- 项目版本号与 Git tag 均不带 `v` 前缀，例如使用 `1.2.0`，不要使用 `v1.2.0`。

---

## 🏗️ 架构风格

### 技术栈

| 层级 | 技术 |
|------|------|
| UI 框架 | **AppKit** (NSWindow / NSView) |
| 渲染引擎 | **Metal** (CAMetalLayer + CVMetalTextureCache) |
| 帧同步 | CVDisplayLink |
| iOS 捕获 | CoreMediaIO + AVFoundation |
| Android 捕获 | scrcpy-server + Socket + VideoToolbox |
| 音频播放 | AVAudioEngine |
| 状态管理 | Combine |
| 并发模型 | Swift Concurrency (async/await) |

### 关键设计模式

1. **协议驱动设计**: `DeviceSource` 协议统一 iOS/Android 设备接口
2. **单例模式**: `AppState.shared`, `AppLogger.shared`
3. **观察者模式**: Combine 的 Publisher/Subscriber
4. **策略模式**: 不同编解码器的处理（AAC/OPUS/RAW）

---

## 📁 目录结构

```
ScreenPresenter/
├── AppDelegate.swift              # 应用入口，菜单和工具栏配置
├── main.swift                     # 程序入口点
├── Core/
│   ├── AppState.swift             # 全局状态管理 (@MainActor 单例)
│   ├── Audio/
│   │   ├── AudioPlayer.swift      # AVAudioEngine 播放器
│   │   ├── AudioRegulator.swift   # 音频缓冲调节器 (参考 scrcpy)
│   │   └── RingBuffer.swift       # 环形缓冲区
│   ├── DeviceDiscovery/
│   │   ├── IOSDevice.swift        # iOS 设备模型
│   │   ├── IOSDeviceProvider.swift
│   │   ├── AndroidDevice.swift    # Android 设备模型
│   │   └── AndroidDeviceProvider.swift
│   ├── DeviceSource/
│   │   ├── DeviceSource.swift     # 设备源协议与基类
│   │   ├── IOSDeviceSource.swift  # iOS 实现 (AVCaptureSession)
│   │   ├── ScrcpyDeviceSource.swift # Android 实现
│   │   └── Scrcpy/                # scrcpy 相关组件
│   │       ├── ScrcpyServerLauncher.swift
│   │       ├── ScrcpySocketAcceptor.swift
│   │       ├── ScrcpyVideoStreamParser.swift
│   │       ├── ScrcpyAudioStreamParser.swift
│   │       └── Scrcpy*Decoder.swift
│   ├── Preferences/
│   │   └── UserPreferences.swift  # 用户偏好设置
│   ├── Process/
│   │   ├── ProcessRunner.swift    # 进程管理
│   │   └── ToolchainManager.swift # 工具链管理 (adb, scrcpy)
│   ├── Rendering/
│   │   ├── MetalRenderer.swift    # Metal 渲染器
│   │   ├── MetalRenderView.swift  # 渲染视图
│   │   ├── VideoToolboxDecoder.swift # H.264/H.265 解码器
│   │   ├── FramePipeline.swift    # 帧管道 + CVDisplayLink
│   │   └── ColorCompensation/     # 色彩补偿
│   └── Utilities/
│       ├── Logger.swift           # 日志框架
│       ├── Localization.swift     # 本地化
│       └── ...
├── Views/
│   ├── MainViewController.swift   # 主视图控制器
│   ├── PreferencesWindowController.swift
│   └── Components/
│       ├── DevicePanelView.swift  # 设备面板
│       ├── DeviceBezelView.swift  # 设备边框绘制
│       ├── DeviceModel.swift      # 50+ 设备型号定义
│       └── ...
└── Resources/
    ├── Tools/                     # 捆绑的工具 (adb, scrcpy)
    ├── en.lproj/                  # 英文本地化
    └── zh-Hans.lproj/             # 中文本地化
```

---

## 📝 代码规范

### 文件头模板

每个 Swift 文件必须包含标准文件头：

```swift
//
//  FileName.swift
//  ScreenPresenter
//
//  Created by Sun on YYYY/MM/DD.
//
//  功能简述
//  详细说明（可选）
//
```

### MARK 注释

使用 `// MARK: -` 分隔代码区域：

```swift
// MARK: - 属性

private var someProperty: String

// MARK: - 初始化

init() { }

// MARK: - 公开方法

func publicMethod() { }

// MARK: - 私有方法

private func privateMethod() { }
```

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 类/结构体/枚举 | 大驼峰 | `DeviceSource`, `CapturedFrame` |
| 属性/变量/函数 | 小驼峰 | `isPlaying`, `startCapture()` |
| 常量 | 小驼峰或全大写 | `maxBuffering`, `PACKET_HEADER_SIZE` |
| 协议 | 大驼峰 + 动词/形容词 | `Sendable`, `Identifiable` |

### 访问控制

- 默认 `private`，需要时才放开
- 用 `private(set)` 暴露只读属性
- 避免使用 `open`

### Swift Concurrency

```swift
// ✅ 正确: 使用 @MainActor 标注主线程类
@MainActor
final class AppState {
    // ...
}

// ✅ 正确: 异步函数使用 async/await
func connect() async throws {
    // ...
}

// ❌ 避免: 在 async 上下文中使用 DispatchQueue.main.async
```

### 错误处理

```swift
// 定义专用错误类型
enum DeviceSourceError: LocalizedError, Equatable {
    case connectionFailed(String)
    case permissionDenied
    // ...
    
    var errorDescription: String? {
        switch self {
        case .connectionFailed(let msg): return L10n.error.connectionFailed(msg)
        // ...
        }
    }
}
```

---

## 📊 日志系统

使用 `AppLogger` 的分类日志，基于 `os.log`：

```swift
// 可用的日志分类
AppLogger.app.info("应用启动")
AppLogger.device.info("发现设备: \(deviceName)")
AppLogger.capture.error("捕获失败: \(error)")
AppLogger.rendering.debug("渲染帧: \(frameCount)")
AppLogger.connection.warning("连接不稳定")
AppLogger.process.info("进程已启动: \(pid)")

// 日志级别: debug < info < warning < error
```

### 日志规范

- **info**: 正常业务流程的关键节点
- **debug**: 详细调试信息（生产环境可关闭）
- **warning**: 可恢复的异常情况
- **error**: 需要关注的错误

---

## 🌐 本地化

使用 `L10n` 结构体获取本地化字符串：

```swift
// ✅ 正确
label.stringValue = L10n.device.connecting
throw DeviceSourceError.connectionFailed(L10n.error.noDevice(L10n.platform.ios))

// ❌ 错误: 硬编码字符串
label.stringValue = "连接中..."
```

本地化文件位于:
- `Resources/en.lproj/Localizable.strings`
- `Resources/zh-Hans.lproj/Localizable.strings`

---

## 🧪 测试规范

使用 XCTest 框架：

```swift
import XCTest
@testable import ScreenPresenter


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HapticTide/ScreenPresenter](https://github.com/HapticTide/ScreenPresenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
