---
trigger: always_on
description: > 本文件是项目的技术索引文档，供 AI Agents 和开发者快速了解项目全貌。
---

# AGENTS.md — ipaDown for Mac

> 本文件是项目的技术索引文档，供 AI Agents 和开发者快速了解项目全貌。

---

## 项目概述

**ipaDown** 是一款使用 Swift + SwiftUI 构建的 **跨平台苹果应用下载工具**（支持 macOS 与 iOS）。它通过 Apple 的私有 plist API 实现用户认证、应用购买、历史版本查询、分块下载和签名注入。

- **支持平台**: 
  - **macOS**: 26.0+ (原生 App)
  - **iOS/iPadOS**: 26.0+ (提供侧载 IPA)
- **Swift 版本**: Swift 6（严格并发模式）
- **UI 框架**: SwiftUI (通用组件布局)
- **Bundle ID**: `com.shawnrain.ipaDown`
- **自动更新**: Sparkle (仅 macOS 端)

---

## 架构模式

项目采用 **MVVM (Model-View-ViewModel)** 架构，使用 `@Observable` 宏实现响应式数据绑定。

```
┌─────────────┐    ┌──────────────┐    ┌──────────────┐
│   Views     │◄───│  ViewModels  │◄───│   Services   │
│  (SwiftUI)  │    │ (@Observable)│    │ (enum/static)│
└─────────────┘    └──────────────┘    └──────────────┘
                          │                    │
                   ┌──────┴──────┐      ┌──────┴──────┐
                   │   Models    │      │  Utilities  │
                   │  (Codable)  │      │  (Helpers)  │
                   └─────────────┘      └─────────────┘
```

### 状态注入方式
- 所有 ViewModel 通过 `@Environment` 注入到 View 树
- App 入口 `ipaDownApp.swift` 创建 `@State` ViewModel 实例，并通过 `.environment()` 传递
- ViewModel 之间通过直接引用通信（如 `DownloadManager.accountManager`）

---

## 目录结构

```
ipaDown/
├── ipaDownApp.swift          # App 入口（WindowGroup、主题、Sparkle）
├── AppDelegate.swift         # NSApplicationDelegate（Sparkle 更新）
├── ContentView.swift         # NavigationSplitView + 页面路由
├── Info.plist                # Sparkle 配置（SUFeedURL、SUPublicEDKey）
├── ipaDown.entitlements      # 权限（App Sandbox 已关闭）
│
├── Models/
│   ├── Account.swift         # Apple 账号（含 HTTPCookieData）
│   ├── AppSoftware.swift     # iTunes Search API 搜索结果
│   ├── CountryCodes.swift    # 国家/地区 → Store Front ID 映射
│   ├── DownloadTask.swift    # 下载任务（@Observable + Codable）
│   └── VersionInfo.swift     # 版本信息
│
├── Services/
│   ├── AuthService.swift     # Apple 认证（plist API, 两步验证）
│   ├── DownloadService.swift # 下载（5MB 分块 × 10 并行）
│   ├── PurchaseService.swift # 购买（免费应用许可获取）
│   ├── SearchService.swift   # 搜索（iTunes Search API）
│   ├── SignatureService.swift# IPA sinf 签名注入（Process: unzip/zip）
│   ├── VersionService.swift  # 历史版本查询（Apple API + Bilin API）
│   ├── StoreClient.swift     # HTTP 客户端（plist POST / JSON GET）
│   ├── DeviceIdentifier.swift# 设备标识（IOKit MAC 地址）
│   └── NotificationService.swift # 系统通知（UNUserNotificationCenter）
│
├── ViewModels/
│   ├── AccountManager.swift  # 账号管理（Keychain 持久化）
│   ├── SearchManager.swift   # 搜索管理
│   ├── VersionManager.swift  # 版本管理（Bilin 自动降级 Apple API）
│   ├── DownloadManager.swift # 下载管理（JSON 持久化、任务队列）
│   └── NavigationManager.swift # 导航管理（前进/后退栈）
│
├── Views/
│   ├── SidebarView.swift     # 侧边栏导航
│   ├── AccountView.swift     # 账号管理页
│   ├── SearchView.swift      # 搜索页
│   ├── VersionView.swift     # 历史版本页
│   ├── DownloadView.swift    # 下载管理页
│   ├── SettingsView.swift    # 偏好设置
│   ├── AboutView.swift       # 关于页
│   ├── About/
│   │   ├── ChangelogView.swift # 更新日志
│   │   └── LicenseView.swift   # 开源许可
│   └── Components/
│       ├── TaskInspectorView.swift # 任务详情面板
│       ├── SearchBar.swift    # 搜索栏组件
│       └── GlassCard.swift    # 毛玻璃卡片组件
│
├── Utilities/
│   ├── IPAError.swift        # 统一错误类型
│   ├── KeychainHelper.swift  # Keychain 读写封装
│   ├── Logger.swift          # 日志系统（@Observable, os.Logger）
│   ├── MD5Helper.swift       # MD5 校验（CryptoKit）
│   ├── PlistHelper.swift     # Plist 序列化/反序列化
│   └── LockedValue.swift     # 线程安全值封装
│
└── Assets.xcassets/          # 图标和资源
```

---

## 核心流程

### 1. 认证流程
```
用户输入邮箱/密码 → AuthService.authenticate()
→ StoreClient.postPlist() → 处理 302 重定向
→ 是否需要两步验证 → 返回 Account（含 cookies、passwordToken、dsPersonId）
→ AccountManager 保存到 Keychain
```

### 2. 下载流程
```
用户选择版本 → DownloadManager.addTask()
→ PurchaseService.purchase() (获取免费应用许可)
→ DownloadService.requestDownload() (获取 URL + sinfs)
→ DownloadService.downloadFile() (5MB 分块 × 10 线程并行)
→ MD5Helper.calculateMD5() (校验)
→ SignatureService.signIPA() (注入 sinf 签名)
→ 完成 + 系统通知
```

### 3. Token 自动刷新
- 下载中遇到 `IPAError.tokenExpired` → 自动调用 `AuthService.refreshToken()` → 重试下载
- App 启动时调用 `AccountManager.refreshAllTokens()`

---

## 代码规范

### 命名约定
- **文件名**: 与主要类型同名（PascalCase）
- **Services**: 使用 `enum` + `static func`（无实例化需求）
- **ViewModels**: 使用 `class` + `@Observable` 宏
- **Models**: 使用 `struct`，遵循 `Codable, Identifiable, Hashable`

### 并发模式
- 使用 Swift 6 严格并发检查
- 使用 `async/await` 和 `Task`/`TaskGroup`
- 主线程隔离通过 `@MainActor` 标注
- 后台密集操作使用 `Task.detached(priority: .userInitiated)`
- 线程安全值封装使用 `LockedValue<T>` (NSLock)

### 数据持久化规范
- **卸载无痕约束**：所有敏感凭证（如登录账号、Token）及用户数据**必须**保存在 App 私有的可回收沙盒目录中（如 `Application Support/ipaDown/` 下存放 `.json`）。
- **禁止持久残留**：严禁使用脱离应用生命周期的系统级存储（如公共 `Keychain` 或无沙盒隔离的暴露版 `UserDefaults`）来存放账号等信息，确保在各系统（如 macOS）上卸载 App 时能实现**数据彻底销毁、零痕迹残留**。

### UI 规范
- 所有界面文本使用中文
- 使用系统 accent color 作为主色调
- 自定义按钮使用 `.buttonStyle(.plain)` + 手动实现 hover 效果
- 使用 `RoundedRectangle(cornerRadius: 10~12, style: .continuous)`

### 错误处理
- 统一使用 `IPAError` 枚举
- 实现 `LocalizedError` 协议提供中文错误描述
- Token 过期自动重试（最多 2 次）

---

## 注意事项与安全准则

### 1. 密钥与敏感信息管理
- **禁止提交私钥**: 项目相关的 `.key` (RSA/EdDSA 私钥) 已被 `.gitignore` 忽略。**严禁**手动强制提交任何敏感密钥文件到 Git 仓库。
- **Sparkle 签名**: 自动打包脚本 `build_all.sh` 会尝试从本地 Keychain 获取签名私钥。如果本地私钥丢失，请参考 `scripts/release.sh` 中的说明重新生成并配置，切勿在脚本中硬编码私钥。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShawnRn/ipaDown-for-Mac](https://github.com/ShawnRn/ipaDown-for-Mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
