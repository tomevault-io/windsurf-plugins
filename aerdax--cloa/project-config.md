---
trigger: always_on
description: 面向所有 AI 编码 agent 的唯一规范与架构文档：硬性规则 + 完整项目描述、架构与目录导航。
---

# AGENTS.md — Cloa 虚拟衣柜 iOS App

面向所有 AI 编码 agent 的唯一规范与架构文档：硬性规则 + 完整项目描述、架构与目录导航。

## Quick Reference
- **Platform**: iOS 26.0+ only（deployment target: 26.2）— 不向下兼容 iOS 25 及以下
- **Language**: Swift 6
- **UI Framework**: SwiftUI + Liquid Glass（Apple 液态玻璃设计语言）
- **Architecture**: MVVM with `@Observable`
- **Storage**: SwiftData + CloudKit
- **Testing**: Swift Testing + XCTest UI Tests
- **Package Manager**: Swift Package Manager
- **AI**: 火山方舟 Ark（Doubao-Seedream，图像生成 / 多图融合）

## 构建与验证（强制）
- **改完代码即停手，不构建、不运行、不截图、不测试**——一律由用户自行在 Xcode / 模拟器验证
- **禁止调用任何 XcodeBuildMCP 工具**（build / run / test / screenshot / snapshot_ui / 日志捕获等全部包含在内），也不要用 `xcodebuild`、`simctl` 等 shell 命令绕过
- 完成后只需说明改了哪些文件、改动的原理与可调参数，把验证交回用户
- **唯一例外**：用户在当次对话中明确要求「构建 / 运行 / 跑测试 / 截图看看」时才执行；该授权仅对当次请求有效，不延续到后续任务
- 若用户授权构建，Xcode 操作走 XcodeBuildMCP：Build `build_sim_name_proj`、Test `test_sim_name_proj`、Run `build_run_sim_name_proj`、Clean `clean`；单测通过 `test_plan` 或 `only_testing` 参数过滤

## 核心规则
- 项目使用 PBXFileSystemSynchronizedRootGroup（Xcode 16+）：`Cloa/` 目录下的新文件**自动纳入编译**，无需修改 .pbxproj
- 可以修改 .pbxproj（调整 Build Settings、添加 capability 等），但新增 Swift 源文件不需要
- **最低系统 iOS 26.0**，`IPHONEOS_DEPLOYMENT_TARGET = 26.2`；允许无条件使用 iOS 26 专属 API（含 Liquid Glass），禁止为 iOS 25 及以下做 `if #available` 兼容分支
- 禁止使用 UIKit，除非包装无 SwiftUI 替代的系统 API（`UIImage`、`UIGraphicsImageRenderer` 等除外）
- 禁止 force unwrap（`!`）
- 优先 struct 而非 class
- 使用 `@Observable` 而非 `ObservableObject`
- 使用 async/await，不使用 Combine
- 所有 public API 必须有文档注释
- 所有颜色使用 Asset Catalog 语义化 Color Set（Light/Dark Appearance），禁止硬编码颜色值
- API Key 一律经 `AppSecrets` 从本地 `Config/Secrets.xcconfig`（已 gitignore）读取，**禁止硬编码任何密钥**，也禁止把真实 Key 写回源码、文档或提交历史

### UI 规范：Liquid Glass（强制）
- 全局采用 Apple **Liquid Glass**（液态玻璃）设计语言，所有内容都要适配。容器、卡片、工具栏、TabBar、悬浮按钮、fullScreenCover 浮层、试穿结果操作条等一律使用 `.glassEffect(...)` / `GlassEffectContainer`，强调半透明玻璃质感与背景折射、明暗自适应。
- 浮层控件与导航层统一玻璃化；避免用不透明纯色大色块遮挡玻璃层次，让内容在玻璃之下透出。
- 语义化 Color Set 仍是唯一颜色来源，但需保证文字/图标在玻璃材质上于 Light/Dark 下均有足够对比度；玻璃之上的文字一律用语义 token（如 `CloaTextTertiary`），禁止硬编码颜色。
- 因最低系统为 iOS 26，Liquid API 直接使用，无需 `if #available` 回退。

## 架构概览

### 目录导航
```
Cloa/
├── App/                  应用入口与根导航（CloaApp.swift = Schema 注册 / MainTabView.swift = 四 Tab）
├── Core/
│   ├── Models/           SwiftData @Model（ClothingItem / Outfit / VirtualModel / TryOnRecord + 枚举）
│   ├── Services/         图像 I/O、Ark 生成、试穿执行器、感知哈希、展示背景常量
│   ├── Extensions/       UIImage 等扩展（flattened / resized / 像素尺寸对齐）
│   └── Localization/     多语言（AppLanguage.swift）
├── DesignSystem/         语义 token、字体/间距/圆角、Liquid Glass 复用助手（DesignTokens.swift）
├── Features/
│   ├── Scanner/          衣物拍摄 / 裁剪 / 预览 / 编辑（ScannerFlowCoordinator 状态机）
│   ├── Wardrobe/         衣柜网格 / 详情 / 编辑（WardrobeViewModel）
│   ├── Outfit/           搭配列表 / 详情 / 编辑 / 试穿历史（OutfitViewModel）
│   ├── VirtualModel/     虚拟形象两条创建路径 + 列表管理（VirtualModelFlowCoordinator 状态机）
│   └── TryOn/            试穿卡片与执行 UI（异步，与详情页共用 TryOnRunner）
├── Assets.xcassets/      语义 Color Set（唯一颜色真源）
└── Info.plist / Cloa.entitlements
```
- 新增 Swift 文件放入对应 `Features/<模块>` 或 `Core/<层>`，因 `PBXFileSystemSynchronizedRootGroup` 自动纳编，无需改 .pbxproj。

### 数据层
- **SwiftData models** (`Core/Models/`): `ClothingItem`, `Outfit`, `VirtualModel`, `TryOnRecord`
- Schema 注册在 `CloaApp.swift` — 新增 `@Model` 必须同步更新那里的 `Schema([...])` 数组
- 图像以文件存 `Documents/`，SwiftData 只存相对路径
- `ImageStorageService.shared` 是所有图像 I/O 的单一网关（Wardrobe PNG、VirtualModels PNG、TryOn JPEG）
- **服饰单品图不做抠图**：拍摄/导入后由用户裁剪，原样落盘；展示时统一垫在 `DisplayBackground.color`（`#F2F2F3`）卡片上（见 `ClothingHeroImage`），与 AI 生成图背景保持一致。该色号唯一真源是 `ImageBackground.swift` 的 `DisplayBackground`，AI 提示词模板同样引用它
- **虚拟形象「Use My Photo」直接使用拍摄/相册选择的整身原图**作为虚拟形象，**不调用 Ark、不替换背景、无 AI 提示词**（用户负责纯色背景取景）；确认预览后原样保存
- **虚拟形象支持创建多个**：Settings →「Manage Virtual Models」进入 `VirtualModelListView`（右上角「+」新建、左滑删除、仅展示预览图）。保存走 `VirtualModelSaving.save`（不再删除已有记录）；TryOn / Settings 取 `virtualModels.first`
- **所有 AI 生成图尺寸与原图完全一致**：调用 Ark 时按参考图宽高比设置输出 `size`（避免裁切）并在接口约束内取最高分辨率（清晰度最高），返回后再用 `UIImage.resized(toPixelSize:)` 对齐到原图**精确像素尺寸**（见 `VolcArkImageService.matchedSize` / `maxResSize` 与 `ImageBackground.swift`）

### AI 服务层 (`Core/Services/`)
云端图像生成走 **火山方舟 Ark**（VolcEngine Ark），不再使用 KlingAI / 阿里 Dashscope。**服饰扫描全程本地、不调用 Ark**；**虚拟形象「Use My Photo」直接用原图、也不调用 Ark**；仅虚拟形象「参数合成」（场景 1B）与试穿（场景 2）调用 Ark。

| Service | API | Purpose |
|---|---|---|
| `VolcArkImageService` | Ark `POST /api/v3/images/generations` | 云端图像生成入口：虚拟形象参数合成（场景 1B）+ 虚拟试穿（多图融合），**同步返回**；输出 `size` 按参考图宽高比动态计算，结果对齐原图精确像素。（「Use My Photo」不经此服务，直接用原图） |

- **Base URL**: `https://ark.cn-beijing.volces.com/api/v3`
- **Endpoint**: `POST /images/generations`（同步接口，无需轮询、无需 OSS 上传）
- **Auth**: `Authorization: Bearer <ARK_API_KEY>`。**Key 不写进源码与文档**：本地 `Config/Secrets.xcconfig`（已 gitignore）→ xcconfig 构建设置 → `Info.plist` 的 `$(ARK_API_KEY)` → 运行时 `AppSecrets.arkAPIKey` 读取（支持同名环境变量覆盖）。模板见 `Config/Secrets.xcconfig.example`
- **Model ID（可切换）**: 由 `VolcArkImageService.activeModel` 决定，当前 `.seedream50pro`（`doubao-seedream-5-0-pro-260628`）；回退改回 `.seedream45`（`doubao-seedream-4-5-251128`）即可，两者同一 API Key。两代差异（输出像素范围、是否支持 `sequential_image_generation`、最多参考图张数）已收敛进 `Model` 枚举，请求构造与尺寸计算按当前模型取值
- **多图输入**: `image` 字段传字符串或字符串数组，支持 base64 data URL（`data:image/jpeg;base64,...`）或公网 https URL；文生图省略 `image`，图生图 / 多图融合传入参考图（最多 10 张）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aerdax/Cloa](https://github.com/Aerdax/Cloa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
