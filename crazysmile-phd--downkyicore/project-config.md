---
trigger: always_on
description: 本文件为 AI 编码代理在此代码库中高效工作提供所需的全部信息。
---

# AGENTS.md — DownKyi 代码库 AI 编码代理指南

本文件为 AI 编码代理在此代码库中高效工作提供所需的全部信息。

## 强制架构入口

> 在分析或修改 DownKyi 程式碼前，必須先閱讀 `ai-knowledge-graph.md`，確認受影響的節點、依賴關係、穩定契約、風險與對應測試。

本仓库中的实际路径是 `docs/ai-knowledge-graph.md`。任何新增、删除、移动或重新导向模块责任与依赖关系的变更，都必须在同一个分支、commit 组和 Pull Request 中同步更新该文件。执行重构时还必须阅读 `docs/refactoring-live-plan.md`，遵守其中的分支分组与未完成工作清单。

---

## 目录

1. [项目概述](#项目概述)
2. [仓库结构](#仓库结构)
3. [构建与运行](#构建与运行)
4. [测试](#测试)
5. [架构](#架构)
6. [命名规范](#命名规范)
7. [代码风格](#代码风格)
8. [MVVM 模式](#mvvm-模式)
9. [异步模式](#异步模式)
10. [错误处理](#错误处理)
11. [核心工具类](#核心工具类)
12. [包管理](#包管理)

---

## 项目概述

**DownKyi** 是一个跨平台的 Bilibili 视频下载器，使用以下技术构建：

- **Avalonia 12** — 跨平台 XAML UI（Windows、Linux、macOS）
- **Prism.DryIoc.Avalonia** — MVVM 框架、依赖注入、导航、发布/订阅事件
- **Microsoft.Data.Sqlite** — SQLite 原生 SQL 存储（下载队列、历史记录）
- **FFMpegCore** — 音视频混流
- **Downloader** — 内置 HTTP 下载引擎
- **Newtonsoft.Json** — 设置项与 API 响应的序列化

当前旧生产代码包含两个项目，此外还有四个测试项目与一个 benchmark 项目。PR 02 会新增目标分层项目：

| 项目 | 类型 | 目标框架 |
|---|---|---|
| `DownKyi` | WinExe（Avalonia UI） | `net10.0` |
| `DownKyi.Core` | 类库 | `net10.0` |
| `tests/*` | xUnit v3 测试 | `net10.0` |
| `benchmarks/DownKyi.Benchmarks` | BenchmarkDotNet | `net10.0` |

目标运行时：`win-x64`、`win-x86`、`linux-x64`、`linux-arm64`、`osx-x64`、`osx-arm64`。

---

## 仓库结构

```
DownKyi/
├── AGENTS.md                       ← 本文件
├── DownKyi.sln
├── Directory.Build.props           ← 全局配置：Nullable=enable，CPM=true
├── Directory.Build.targets         ← 空文件
├── Directory.Packages.props        ← 所有 NuGet 包版本（CPM）
├── version.txt                     ← 版本号的唯一来源
├── .github/workflows/build.yml     ← CI：全平台构建与发布
│
├── DownKyi/                        ← UI 项目
│   ├── App.axaml.cs                ← Prism DryIoc 引导、DI 注册
│   ├── Program.cs                  ← Avalonia AppBuilder 入口点
│   ├── AppConstant.cs              ← 全局应用常量
│   ├── ViewModels/                 ← MVVM ViewModel（Prism BindableBase）
│   │   ├── ViewModelBase.cs        ← 基类，含属性变更辅助方法
│   │   ├── Dialogs/                ← 对话框 VM（IDialogAware）
│   │   └── DownloadManager/        ← 下载列表项 VM
│   ├── Views/                      ← Avalonia .axaml 视图（CompiledBindings）
│   ├── Services/                   ← IInfoService、IDownloadService 抽象
│   │   └── Download/               ← DownloadService 及内置/Aria 实现
│   ├── Events/                     ← Prism PubSubEvent<T> 定义
│   ├── Models/                     ← 数据库实体（纯 POCO，无 ORM 注解）
│   ├── PrismExtension/             ← 自定义异步 IDialogService 扩展
│   └── Utils/                      ← DictionaryResource（国际化）
│
└── DownKyi.Core/                   ← 领域/API 项目
    ├── BiliApi/                    ← Bilibili HTTP API
    │   ├── WebClient.cs            ← 单例 HttpClient 封装
    │   ├── VideoStream/            ← playURL / 字幕 API
    │   └── BiliUtils/              ← URL 解析、BvId <-> AvId 转换
    ├── Settings/                   ← SettingsManager 单例（partial 类）
    ├── Storage/                    ← StorageManager（路径解析）、SQLite DB
    ├── Logging/                    ← LogManager（异步文件写入器）
    ├── FFMpeg/                     ← FFMpegCore 封装
    ├── Aria2cNet/                  ← aria2 RPC 客户端
    └── Utils/                      ← Format、HardDisk、Debugging.Console、Encryptor
```

---

## 构建与运行

### 前置条件

- .NET 10 SDK
- 系统 PATH 中存在 FFmpeg 二进制文件（开发时混流操作所需）
- `PupNet` 工具（仅打包时需要，开发构建无需）

### 常用命令

```bash
# 还原包
dotnet restore

# Debug 构建
dotnet build

# Release 构建
dotnet build -c Release

# 运行应用
dotnet run --project DownKyi/DownKyi.csproj

# 自包含发布（示例：macOS arm64）
dotnet publish DownKyi/DownKyi.csproj \
  --self-contained \
  -r osx-arm64 \
  -c Release \
  -p:DebugType=None \
  -p:DebugSymbols=false
```

`-r` 支持的值：`win-x64`、`win-x86`、`linux-x64`、`linux-arm64`、`osx-x64`、`osx-arm64`。

---

## 测试

本仓库使用 xUnit v3，并包含以下测试层：

- `tests/DownKyi.Core.Tests`：HTTP、aria2、FFmpeg 与 Core 行为。
- `tests/DownKyi.Tests`：下载流程、储存相容性与应用服务。
- `tests/DownKyi.Desktop.Tests`：Avalonia headless UI smoke tests。
- `tests/DownKyi.Architecture.Tests`：项目依赖方向与循环引用。

提交前至少依序执行：

```bash
dotnet restore ./DownKyi.sln
dotnet build ./DownKyi.sln -c Release --no-restore --no-incremental
dotnet test ./DownKyi.sln -c Release --no-restore --no-build
dotnet format ./DownKyi.sln --verify-no-changes --no-restore
git diff --check
```

不要并行执行同一工作树的 `dotnet build` 与 `dotnet test`，两者可能同时写入 PDB 而产生假失败。测试必须通过 `DOWNKYI_DATA_DIR` 使用隔离目录，禁止读取真实 Cookie、设置或下载数据库。

---

## 架构

`docs/ai-knowledge-graph.md` 是当前代码责任、调用关系、稳定契约、风险和测试入口的权威索引。下述 Prism 架构属于迁移中的旧实现；新代码必须遵守 `DownKyi.Desktop -> DownKyi.Application -> DownKyi.Domain`，以及 `DownKyi.Infrastructure -> Application/Domain` 的目标依赖方向。

### 依赖关系

```
DownKyi（UI） → DownKyi.Core（领域/API）
```

`DownKyi.Core` 不依赖 `DownKyi`。所有 Bilibili API 调用、设置、日志和存储均在 `Core` 中实现。

### Prism DI 注册

所有服务在 `App.axaml.cs` 的 `RegisterTypes` 方法中注册，接口通过 DryIoc 容器解析。ViewModel 中使用构造函数注入，尽量少用 `IContainerProvider`。

### 导航

使用 Prism 区域导航。视图通过区域名称注册自身。导航通过 `IRegionManager.RequestNavigate(regionName, viewName, parameters)` 触发。ViewModel 实现 `INavigationAware`（`OnNavigatedTo`、`OnNavigatedFrom`、`IsNavigationTarget`）。

### 事件（发布/订阅）

事件定义在 `DownKyi/Events/`。在 ViewModel 中注入 `IEventAggregator` 使用。示例：

```csharp
_eventAggregator.GetEvent<NavigationEvent>().Publish(param);
_eventAggregator.GetEvent<MessageEvent>().Publish(message);
```

---

## 命名规范

| 目标 | 规范 | 示例 |
|---|---|---|
| 私有实例字段 | `_camelCase` | `_inputText`、`_downloadService` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crazysmile-PhD/downkyicore](https://github.com/crazysmile-PhD/downkyicore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
