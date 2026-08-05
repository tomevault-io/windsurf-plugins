---
trigger: always_on
description: - **项目名称**：S1er（s1er）
---

# 项目宪法
# 本文件补充全局 GEMINI.md，相同条目以本文件为准。

## 项目信息

- **项目名称**：S1er（s1er）
- **核心目标**：第三方 Stage1st（S1）论坛客户端，基于 Flutter 构建，支持多平台运行
- **创建日期**：2026-07-07

---

## 技术栈锁定

> 所有 AI 生成的代码必须严格遵守以下版本，不得引入未列出的主要依赖。

| 层级 | 技术 | 版本 |
|:---|:---|:---|
| 语言 | Dart | >=3.4 <4.0 |
| 框架 | Flutter | >=3.4 |
| 状态管理 | flutter_riverpod | 3.2.1（临时固定；规避 #4765；Notifier / AsyncNotifier；禁长期依赖 `legacy.dart`） |
| HTTP 客户端 | dio | ^5.4.0 |
| 路由 | go_router | ^17.0.0 |
| 本地结构化存储 | drift / drift_flutter | ^2.34.1 / ^0.3.1 |
| 图片磁盘缓存 | flutter_cache_manager / cached_network_image | ^3.4.1 / ^3.4.1 |
| 分享卡导出编码 | ironpress（方案 C：mozjpeg / oxipng / libwebp；默认 WebP，可选 JPEG/PNG）；Web 走 canvas / Skia PNG | ^0.2.0 |
| 测试夹具位图 | image | ^4.2.0 |
| 网络状态 | connectivity_plus | ^6.1.4 |
| 设备机型标签 | device_info_plus（小尾巴细机型；失败回退平台名） | ^13.2.0 |
| 桌面窗口管理 | window_manager（自绘标题栏；仅 Windows / macOS / Linux） | ^0.5.1 |
| 备份（L1 ZIP） | archive / file_selector / share_plus | ^4.0.9 / ^1.1.0 / ^13.2.0 |
| 回复插图 | file_selector + p.sda1.dev 外链图床 | 已有 file_selector；不做 Discuz attach |
| 麻将脸表情 | `assets/emoticons/` + packs/list/ATTRIBUTION | 自 s1emoticon Release；无 LICENSE 已声明 |
| WebView | webview_flutter | ^4.7.0 |
| HTML 渲染 | flutter_html | ^3.0.0 |
| Cookie 管理 | dio_cookie_manager / cookie_jar | ^3.1.1 / ^4.0.8 |
| 安全存储 | flutter_secure_storage | ^10.x |
| 加密 | cryptography | ^2.x |
| 崩溃监控 | sentry_flutter（可选，通过 --dart-define 注入 DSN 启用） | ^9.0.0 |
| 包管理 | flutter pub | — |
| Lint | flutter_lints | ^6.0.0 |
| 运行环境 | Flutter SDK >=3.4 | 支持 Web / Android / iOS / Windows / macOS / Linux |

> Cookie 走 `PersistCookieJar` + 加密存储，不进 Drift、不进 `s1-backup`。不使用 Hive。

---

## Material Design 3 规范

> 所有 Flutter UI 代码必须遵守以下 M3 规则，违者视为 bug。

- **主题**：`useMaterial3: true` + `ColorScheme.fromSeed(seedColor: ...)`，禁手写色板
- **色彩**：UI 绘制一律从 `Theme.of(context).colorScheme` 取语义色
  - **允许**（见下方「M3 允许模式」）：`themeSeeds` 种子色、`Colors.transparent`、API 数据驱动色
  - **禁止**：`screens/` / `widgets/` 中 `Color(0xFF...)`、`Colors.red` 等语义替代
- **排版**：一律 `textTheme.*`；HTML 渲染须从 `textTheme` 桥接到 `flutter_html` 的 `FontSize`
  - **禁止**：裸写 `fontSize: 14` 等常量（头像 fallback 用 `FittedBox` + `textTheme`）
- **组件映射**：`NavigationBar` / `FilledButton` / `SegmentedButton` / 原生 `Badge`
  - 计数/楼层 → `Badge`；可点击标签/分页 → `Chip` / `ActionChip`
  - `SegmentedButton` 仅用于 2–5 个彼此相关、需要并列比较的选项，并保持至少 48dp 触控目标
  - **选中勾与前置 icon 二选一**：要么 `showSelectedIcon: false`（仅靠容器色表达选中态），要么保留勾且每段 `ButtonSegment` 必须提供语义 `icon`（选中时勾替换该 icon，避免文案左右跳动）。禁止「有勾、无前置 icon」
  - 紧凑屏上的五段短标签应设 `showSelectedIcon: false`，以容器色表达选中态；文案仍放不下时响应式改用下拉选择，禁止压缩字号或触控区域
- **层级**：M3 以色调高度（tonal）为主、阴影为辅。静止表面（`Card` / `AppBar` / Nav / SearchBar）必须显式 `elevation: 0`，靠 `S1Surface` 色阶分层；浮层（Menu / Dialog / Sheet / FAB）按 M3 使用低 elevation 或框架默认阴影
- **透明度**：一律用 `S1Alpha.*` token，禁止内联 `withValues(alpha: 0.x)`
- **排版常量**：`S1Typography.defaultBodySize` 为字号设置标准档，HTML 渲染通过 `S1Typography.bodySize(textTheme)` 桥接
- **Modal sheet 关闭**：标准高度抽屉 / `showS1AdaptiveSheet` **不放**顶栏关闭按钮；靠 drag handle（紧凑屏已由 API 提供）、点 scrim、系统返回 / Escape。禁止内容区再画一套自定义 drag handle。例外：`AlertDialog` 的取消/关闭 **actions**；全屏 modal 的顶栏关闭；内容错误/空态且无其它主操作时的内容区 CTA（如「关闭」）

**审计**：`dart run scripts/audit_m3.dart --fail-on-error`（CI / 本地均需通过）

---

## 命名约定

- 文件命名：snake_case（如 `api_service.dart`、`thread_card.dart`）
- 函数/变量命名：camelCase（如 `fetchThreadList`、`isLoggedIn`）
- 类命名：PascalCase（如 `ApiService`、`ThreadCard`）
- 常量命名：camelCase（Dart 惯例，如 `maxRequestsPerSecond`）；全局常量类使用 PascalCase（如 `S1Constants`、`ApiConfig`）
- Provider 命名：camelCase + `Provider` 后缀（如 `authProvider`、`httpClientProvider`）
- Screen 命名：PascalCase + `Screen` 后缀（如 `HomeScreen`、`LoginScreen`）
- Widget 命名：PascalCase，按功能命名（如 `PostItem`、`QuoteBlock`）

---

## 架构边界

- **目录结构**：

```
lib/
├── config/       # 配置常量（API 地址、应用常量、环境变量）
│   ├── api_config.dart       # API 地址与模块名
│   ├── constants.dart        # 应用常量（UA、限速等）
│   ├── env_config.dart       # --dart-define 环境配置（日志、超时等）
│   └── resource_domains.dart # 资源域名规则（代理、认证、公开）
├── models/       # 纯数据模型（不含业务逻辑）
├── providers/    # Riverpod 状态管理（连接 services 与 UI）
├── screens/      # 页面级 Widget（路由目标）
├── services/     # 服务层（HTTP、API 调用、认证）
├── theme/        # Material 3 主题定义
├── utils/        # 工具函数（BBCode 解析等）
├── widgets/      # 可复用 UI 组件
│   ├── s1_error_view.dart    # 统一错误视图（维护/登录/通用）
│   └── ...
├── app.dart      # 应用入口与路由配置
└── main.dart     # 主入口（初始化 Drift / AppLocalData、HTTP 客户端）
```

- **模块职责**：
  - `config/`：只放静态配置，不含逻辑
  - `models/`：纯数据类，包含 `fromJson` 工厂方法，不依赖 Flutter
  - `services/`：封装所有外部交互（HTTP、Cookie、认证、本地 Drift、备份编解码），不依赖 UI 层
  - `providers/`：桥接 services 与 UI，管理状态生命周期，不含直接 HTTP 调用
  - `screens/`：页面组合层，调用 providers 获取数据，不直接调用 services
  - `widgets/`：可复用的 UI 片段，通过参数接收数据，不持有状态逻辑

- **标准数据流**：Screen / Widget → Provider / Notifier → Service → `S1HttpClient` / Dio → Discuz! API，单向，禁止跨层直调
  - Provider / Notifier 负责业务编排及 `loading/data/error`、缓存失效和关联刷新，不能退化为 Service 的无状态透传层
  - 纯 UI 状态（动画、展开收起、输入框、滚动）可留在 Widget；网络、持久化、认证、缓存和业务错误必须下沉
  - 保持最少必要分层，禁止为单一动作叠加无实际职责的 Controller / UseCase / Repository 包装

---

## 禁止模式

> 以下模式在本项目中绝对禁止，发现即指出，不得生成。

- 硬编码 secrets / API Key / 密码（必须走环境变量或配置文件）
- `eval()` 或类似的动态代码执行
- 裸 `catch` 吞掉异常（必须至少记录日志或向用户展示错误信息）
- 在循环内发起 HTTP 请求（N+1 问题）
- 外部 HTTP 请求不设 timeout
- 直接绕过 `S1HttpClient` 发起网络请求（必须走统一的限速与 Cookie 管理）
- 在 Widget 中直接调用 `ApiService`（必须通过 Provider）
- 在 Model 中引入 Flutter 依赖（models 层必须保持纯 Dart）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shirolin/s1er](https://github.com/Shirolin/s1er) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
