---
trigger: always_on
description: 这是一个基于 Flutter 的 AI 聊天应用，主要面向 Android 和 iOS 移动端。应用支持多模态交互（文字、图片、音频），采用智能体（Agent）管理模式，用户可以配置多个 AI 智能体并进行对话。
---

# Copilot Instructions for xiaozhi_client_flutter

## 项目概述
这是一个基于 Flutter 的 AI 聊天应用，主要面向 Android 和 iOS 移动端。应用支持多模态交互（文字、图片、音频），采用智能体（Agent）管理模式，用户可以配置多个 AI 智能体并进行对话。

**核心功能：**
- 智能体列表管理（添加、编辑、删除智能体配置）
- 多模态消息发送/接收（文字、图片、音频）
- 底部导航栏：对话、设置两个主页面
- 智能体配置（URL、Token 等参数）

**技术栈：** Dart 3.9.2+, Flutter SDK, 主要目标平台 Android/iOS

---

## 项目架构设计

### 目录结构规范
```
lib/
├── main.dart                          # 应用入口
├── app/
│   ├── routes/                        # 路由配置
│   │   ├── app_routes.dart           # 路由常量定义
│   │   └── app_pages.dart            # 路由页面映射
│   ├── themes/                        # 主题配置
│   │   ├── app_theme.dart            # 主题定义
│   │   ├── app_colors.dart           # 颜色常量
│   │   └── app_text_styles.dart      # 文本样式
│   └── config/
│       └── app_config.dart           # 应用配置常量
├── core/
│   ├── network/                       # 网络层
│   │   ├── dio_client.dart           # Dio 客户端封装
│   │   ├── api_interceptor.dart      # 拦截器
│   │   └── api_exception.dart        # 异常处理
│   ├── storage/                       # 本地存储
│   │   └── storage_service.dart      # 封装 shared_preferences/hive
│   ├── utils/                         # 工具类
│   │   ├── logger.dart               # 日志工具
│   │   ├── toast_util.dart           # 提示工具
│   │   └── permission_util.dart      # 权限工具
│   └── constants/
│       └── api_constants.dart        # API 常量
├── data/
│   ├── models/                        # 数据模型
│   │   ├── agent_model.dart          # 智能体模型
│   │   ├── message_model.dart        # 消息模型
│   │   └── user_model.dart           # 用户模型
│   ├── repositories/                  # 数据仓库层
│   │   ├── agent_repository.dart     # 智能体数据仓库
│   │   └── chat_repository.dart      # 聊天数据仓库
│   └── providers/                     # 数据源
│       ├── remote/                    # 远程数据源（API）
│       │   ├── agent_api.dart
│       │   └── chat_api.dart
│       └── local/                     # 本地数据源
│           └── agent_local_storage.dart
├── presentation/
│   ├── pages/                         # 页面
│   │   ├── main/                      # 主页面（底部导航）
│   │   │   ├── main_page.dart
│   │   │   └── main_controller.dart
│   │   ├── conversation/              # 对话页面
│   │   │   ├── conversation_page.dart
│   │   │   ├── conversation_controller.dart
│   │   │   └── widgets/              # 对话页面组件
│   │   │       ├── agent_card.dart
│   │   │       └── agent_list.dart
│   │   ├── chat/                      # 聊天详情页
│   │   │   ├── chat_page.dart
│   │   │   ├── chat_controller.dart
│   │   │   └── widgets/
│   │   │       ├── message_bubble.dart
│   │   │       ├── input_bar.dart
│   │   │       ├── voice_recorder.dart
│   │   │       └── image_picker_widget.dart
│   │   ├── agent_config/              # 智能体配置页
│   │   │   ├── agent_config_page.dart
│   │   │   └── agent_config_controller.dart
│   │   └── settings/                  # 设置页面
│   │       ├── settings_page.dart
│   │       └── settings_controller.dart
│   └── widgets/                       # 全局共享组件
│       ├── common_button.dart
│       ├── loading_widget.dart
│       └── empty_state.dart
└── l10n/                              # 国际化（可选）
    └── app_localizations.dart
```

### 架构模式：Clean Architecture + MVVM + Repository Pattern
- **Presentation Layer**: 页面 + Controller（使用 GetX/Riverpod 管理状态）
- **Domain Layer**: 业务逻辑（可选，简单应用可省略）
- **Data Layer**: Repository + Data Source（Remote/Local）

---

## 核心依赖包（主流选型）

### 必需依赖
```yaml
dependencies:
  # 状态管理（选择其一，推荐 Riverpod 或 GetX）
  flutter_riverpod: ^2.5.0          # 推荐：类型安全，性能优秀
  # get: ^4.6.6                      # 备选：简单易用，路由+状态一体

  # 网络请求
  dio: ^5.4.0                        # HTTP 客户端
  retrofit: ^4.0.0                   # 类型安全的 API 定义（可选）
  retrofit_generator: ^8.0.0         # retrofit 代码生成
  
  # 本地存储
  shared_preferences: ^2.2.2         # 轻量键值存储
  hive: ^2.2.3                       # 高性能 NoSQL 数据库
  hive_flutter: ^1.1.0
  path_provider: ^2.1.2              # 文件路径

  # 路由导航
  go_router: ^13.0.0                 # 声明式路由（推荐）
  # get: ^4.6.6                      # 或使用 GetX 路由

  # JSON 序列化
  json_annotation: ^4.8.1
  freezed_annotation: ^2.4.1         # 不可变数据类（推荐）

  # UI 组件
  cached_network_image: ^3.3.1       # 图片缓存
  flutter_svg: ^2.0.10               # SVG 支持
  shimmer: ^3.0.0                    # 加载骨架屏
  
  # 多媒体
  image_picker: ^1.0.7               # 图片选择
  # record: ^5.0.4                   # 音频录制（暂时移除，存在 Linux 平台兼容性问题）
  audioplayers: ^5.2.1               # 音频播放
  permission_handler: ^11.3.0        # 权限管理
  
  # 工具类
  logger: ^2.0.2                     # 日志
  fluttertoast: ^8.2.4               # Toast 提示
  intl: ^0.19.0                      # 国际化和日期格式化
  uuid: ^4.3.3                       # UUID 生成

dev_dependencies:
  # 代码生成
  build_runner: ^2.4.8
  json_serializable: ^6.7.1
  freezed: ^2.4.7
  hive_generator: ^2.0.1
  retrofit_generator: ^8.0.0
  
  # 测试
  flutter_test:
    sdk: flutter
  mockito: ^5.4.4                    # Mock 测试
  
  # 代码质量
  flutter_lints: ^5.0.0
```

---

## 代码规范

### 命名规范
- **文件名**: `lowercase_with_underscores.dart`
- **类名**: `PascalCase` (例如: `AgentModel`, `ChatController`)
- **变量/方法**: `lowerCamelCase` (例如: `agentList`, `sendMessage()`)
- **常量**: `lowerCamelCase` 或 `SCREAMING_SNAKE_CASE`
- **私有成员**: 前缀 `_` (例如: `_counter`, `_initState()`)

### Widget 编写规范
```dart

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eiriksgata/xiaozhi-client-flutter](https://github.com/eiriksgata/xiaozhi-client-flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
