---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

HMusic 是一款智能音乐播放器,支持两种播放模式:
- **xiaomusic 模式**: 通过 xiaomusic 服务端控制小爱音箱(需部署服务端)
- **直连模式**: 直接通过小米 IoT API 控制小爱音箱(无需服务端)

## 常用命令

### 开发环境

```bash
# 获取依赖
flutter pub get

# 运行应用 (开发模式)
flutter run

# 代码生成 (用于 Riverpod 和 JSON 序列化)
flutter pub run build_runner build --delete-conflicting-outputs

# 清理构建缓存
flutter clean && flutter pub get
```

### 构建发布版本

```bash
# 使用自动化构建脚本 (推荐)
./build_release.sh

# 该脚本会:
# 1. 自动读取并可选更新版本号
# 2. 提供多种构建选项 (Android 通用版/分架构版/iOS)
# 3. 自动签名和混淆
# 4. 生成 SHA256 校验和
# 5. 输出到 build/release/ 目录

# 手动构建 Android APK (通用版)
flutter build apk --release --obfuscate --split-debug-info=build/symbols

# 手动构建 Android APK (分架构版,体积更小)
flutter build apk --release --split-per-abi --obfuscate --split-debug-info=build/symbols

# 手动构建 iOS IPA (无签名)
flutter build ios --release --no-codesign --obfuscate --split-debug-info=build/symbols
```

### 测试

```bash
# 运行所有测试
flutter test

# 运行单个测试文件
flutter test test/path/to/test_file.dart

# 查看测试覆盖率
flutter test --coverage
```

## 核心架构

### 状态管理

使用 **Riverpod** 进行状态管理,所有 Provider 定义在 `lib/presentation/providers/` 目录。

**关键 Provider:**

- `playbackProvider` - 播放控制总控制器
- `playbackModeProvider` - 播放模式选择 (xiaomusic/直连)
- `directModeProvider` - 直连模式状态管理
- `authProvider` - xiaomusic 模式认证
- `deviceProvider` - 设备选择
- `musicSearchProvider` - 音乐搜索

### 策略模式架构

播放控制使用**策略模式**,支持多种播放策略:

```
PlaybackProvider
    └── PlaybackStrategy (抽象接口)
            ├── RemotePlaybackStrategy (xiaomusic 模式)
            │       └── 通过 xiaomusic 服务端 API 控制
            ├── MiIoTDirectPlaybackStrategy (直连模式)
            │       └── 直接调用小米 IoT Cloud API
            └── LocalPlaybackStrategy (本地播放)
                    └── 使用 just_audio 本地播放
```

**核心文件:**
- `lib/data/services/playback_strategy.dart` - 策略接口定义
- `lib/data/services/remote_playback_strategy.dart` - xiaomusic 模式实现
- `lib/data/services/mi_iot_direct_playback_strategy.dart` - 直连模式实现
- `lib/data/services/local_playback_strategy.dart` - 本地播放实现

### 双模式设计

**xiaomusic 模式:**
- ✅ 功能完整 (播放控制、进度、音量、播放列表)
- ✅ 支持本地音乐库管理
- ⚠️ 需要部署 xiaomusic 服务端

**直连模式:**
- ✅ 无需服务端,仅需小米账号
- ✅ 支持音乐搜索和播放
- ⚠️ 受小米 IoT API 限制,不支持进度查询和音量控制

**模式切换实现:**
1. `playbackModeProvider` 保存当前选择的模式
2. `PlaybackProvider._initializeStrategy()` 根据模式创建对应策略
3. 配置通过 `SharedPreferences` 持久化

### 路由管理

使用 **GoRouter** 进行路由管理,配置在 `lib/app_router.dart`。

**主要路由:**
- `/` - 首页 (通过 AuthWrapper 自动跳转)
- `/mode_selection` - 播放模式选择页
- `/direct_login` - 直连模式登录页
- `/settings` - 设置页面
- `/now-playing` - 正在播放页面

## 代码规范

### 导入顺序

```dart
// 1. Flutter SDK
import 'package:flutter/material.dart';

// 2. 第三方包 (按字母顺序)
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

// 3. 项目内部 (相对路径)
import '../models/music.dart';
import '../providers/playback_provider.dart';
```

### 命名规范

- **Provider**: `xxxProvider` (例: `playbackProvider`)
- **Notifier**: `XxxNotifier` (例: `PlaybackNotifier`)
- **State**: `XxxState` (例: `PlaybackState`)
- **Page**: `XxxPage` (例: `NowPlayingPage`)
- **Service**: `XxxService` (例: `MiIoTService`)
- **Strategy**: `XxxPlaybackStrategy` (例: `RemotePlaybackStrategy`)

### 日志规范

使用 emoji 前缀标识日志类型:

```dart
debugPrint('✅ [模块] 成功信息');
debugPrint('⚠️ [模块] 警告信息');
debugPrint('❌ [模块] 错误信息');
debugPrint('🔧 [模块] 调试信息');
debugPrint('📡 [模块] 网络请求');
```

### 状态管理模式

所有 Provider 使用 `StateNotifier` 模式:

```dart
// 1. 定义 State 类 (使用 sealed class 确保类型安全)
sealed class XxxState {}
class XxxInitial extends XxxState {}
class XxxLoading extends XxxState {}
class XxxSuccess extends XxxState {}
class XxxError extends XxxState {}

// 2. 定义 Notifier
class XxxNotifier extends StateNotifier<XxxState> {
  XxxNotifier() : super(XxxInitial());
}

// 3. 定义 Provider
final xxxProvider = StateNotifierProvider<XxxNotifier, XxxState>((ref) {
  return XxxNotifier();
});
```

## 重要设计模式

### 1. 策略模式 (Strategy Pattern)

**用途**: 播放控制支持多种策略

**实现位置**: `lib/data/services/playback_strategy.dart`

**如何扩展**:
1. 创建新策略类实现 `PlaybackStrategy` 接口
2. 在 `PlaybackProvider._initializeStrategy()` 中添加策略选择逻辑
3. 更新 `PlaybackMode` 枚举 (如需新模式)

### 2. Provider 依赖注入

**用途**: 管理依赖关系和状态共享

**核心文件**: `lib/presentation/providers/`

**注意事项**:
- 使用 `ref.read()` 读取一次性值
- 使用 `ref.watch()` 监听状态变化
- 避免循环依赖

### 3. 配置持久化

**使用 SharedPreferences 保存配置:**

```dart
// 保存
final prefs = await SharedPreferences.getInstance();
await prefs.setString('key', 'value');

// 读取
final value = prefs.getString('key');
```

**已持久化的配置:**
- 播放模式选择 (`playback_mode`)
- 直连模式账号密码 (`direct_mode_account`, `direct_mode_password`)
- xiaomusic 服务器配置 (在 `authProvider` 中)
- 设备选择 (在 `deviceProvider` 中)

## 小米 IoT API 说明

### 登录流程

```
1. GET https://account.xiaomi.com/pass/serviceLogin?sid=micoapi
   → 获取 _sign

2. POST https://account.xiaomi.com/pass/serviceLoginAuth2
   Body: {user, hash(MD5), sid, _sign}
   → 获取 location URL

3. GET location URL
   → 从 Cookie 获取 serviceToken 和 userId
```

### 播放音乐

```
POST https://api.mina.mi.com/remote/ubus
Query: ?deviceId=xxx&message=player_play_url&path=mediaplayer
Headers: Cookie: serviceToken=xxx; userId=xxx
Body: {"url": "音乐URL"}
```

**实现位置**: `lib/data/services/mi_iot_service.dart`

## 依赖说明

### 核心依赖


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hpcll/HMusic](https://github.com/hpcll/HMusic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
