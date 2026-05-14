---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个 Unreal Engine 5.4 项目，整合了 NoesisGUI (XAML UI框架) 和 PuerTS (TypeScript 运行时)，通过 **PuerTS 的 `uclass_extends` 功能**实现 TypeScript 编写 NoesisGUI ViewMode 的完整方案。项目成功**完美复刻**了 NoesisGUI 官方的 **Buttons** 和 **QuestLog** 两个示例，证明了方案的可行性和完整性。

### 核心创新点

1. **TypeScript 代码化 ViewMode**：使用 PuerTS 的 `uclass_extends` 继承 UE 引擎类，通过 TypeScript 生成蓝图类
2. **动态 DataContext 设置**：自定义 UNoesisViewModeInstance 子类，解决官方 NoesisInstance 不允许动态设置 DataContext 的限制
3. **AI 友好的开发模式**：XAML 和 ViewMode 均为代码形式，易于版本控制和 AI 辅助开发
4. **避免蓝图合并冲突**：完全代码化的工作流，告别蓝图文件合并冲突的噩梦

## 核心架构

### 模块结构与依赖关系

#### NoesisDemo 模块（主游戏模块）
- **路径**: `Source/NoesisDemo/`
- **公共依赖**: Core, CoreUObject, Engine, InputCore, EnhancedInput, Puerts, JsEnv
- **私有依赖**: GameplayTags, StructUtils, NoesisViewMode
- **核心职责**: 管理 PuerTS 生命周期，提供 C++ 到 TypeScript 的调用接口

#### NoesisViewMode 模块（MVVM 框架）
- **路径**: `Source/NoesisViewMode/`
- **公共依赖**: Core, CoreUObject, Engine, GameplayTags
- **私有依赖**: Slate, SlateCore, UMG, Noesis, NoesisRuntime, Json, JsonUtilities
- **核心职责**: 提供 NoesisGUI 和 TypeScript 之间的数据绑定框架

### 关键组件

#### C++ 端
- **UNoesisDemoGameInstance**: 游戏实例，初始化和管理 PuerTS JsEnv
- **UNoesisDemoPuertsSubsystem**: PuerTS 调用子系统，提供 `CallTypeScript(GameplayTag, InstancedStruct)` 接口
- **UNoesisViewModeInstance**: **核心组件** - 继承自 UNoesisInstance，解决官方类不允许动态设置 DataContext 的问题，在 XamlLoaded 时自动设置 PendingDataContext
- **UNoesisNotifyHelperLibrary**: 核心属性通知 API，支持基础属性、TArray、TMap 的精细操作通知
- **NoesisViewModeFunctionLibrary**: ViewMode 工具函数库，提供创建 NoesisViewModeInstance 的便捷接口

#### TypeScript 端
- **ScriptCallHandler**: C++ 到 TypeScript 的调用路由器，基于 GameplayTag 分发
- **NoesisProxy**: 自动属性通知 Proxy，包装 ViewMode 实现自动 PropertyChanged
- **NoesisViewUtils**: 视图创建和管理工具类，封装常用操作
- **ContextManager**: 全局上下文管理器
- **BindingHelper**: 回调绑定辅助工具，处理事件绑定

### 数据绑定流程

```
1. TypeScript 类定义 (使用 PuerTS 的 uclass_extends)
   TS_ButtonsViewMode extends UE.Object
   使用 @uproperty、@ufunction 装饰器定义属性和方法
   ↓
2. 蓝图类生成
   PuerTS 根据 TypeScript 类定义自动生成蓝图类
   路径：/Game/BluePrints/TypeScript/ViewMode/Buttons/TS_ButtonsViewMode_C
   ↓
3. ViewMode 实例创建
   UE.Class.Load(classPath) → UE.NewObject(ViewModeClass)
   通过 PuerTS 的 NewObject 创建蓝图类实例
   ↓
4. NoesisViewModeInstance 创建与绑定
   创建 UNoesisViewModeInstance，设置 PendingDataContext = viewMode
   当 XAML 加载完成时，XamlLoaded 事件触发，自动设置 DataContext
   ↓
5. 数据绑定生效
   XAML Binding 表达式 → ViewMode 的 @uproperty 属性
   ↓
6. 属性更新通知
   NoesisProxy 拦截属性修改 → UNoesisNotifyHelperLibrary → NoesisGUI 刷新 UI
```

**关键点**：UNoesisViewModeInstance 的 XamlLoaded 回调是整个流程的核心，它解决了官方 NoesisInstance 不支持动态设置 DataContext 的限制。

## 项目目录结构

```
NoesisDemo/
├── Assets/                         # NoesisGUI 资源目录
│   ├── GUI/                        # XAML 界面文件
│   │   ├── MainPage.xaml
│   │   ├── Buttons/
│   │   │   ├── MainWindow.xaml
│   │   │   └── Resources.xaml
│   │   └── .noesis/                # NoesisGUI 生成的数据
│   └── NoesisGUI/                  # C# 设计时项目（Blend/VS）
├── Content/                        # UE 内容资源
│   ├── JavaScript/                 # 编译后的 JS（TypeScript 输出）
│   ├── GUI/                        # 导入的 NoesisGUI 资源
│   └── BluePrints/
│       └── TypeScript/ViewMode/    # PuerTS 生成的蓝图包装类
├── Plugins/
│   ├── NoesisGUI/                  # NoesisGUI 插件
│   └── Puerts/                     # PuerTS 插件
├── Source/                         # C++ 源代码
│   ├── NoesisDemo/                 # 主游戏模块
│   └── NoesisViewMode/             # MVVM 框架模块
├── TypeScript/                     # TypeScript 源代码
│   ├── main.ts                     # PuerTS 入口
│   ├── ScriptCallHandler.ts        # 调用路由器
│   ├── NoesisProxy.ts              # 属性通知代理
│   ├── NoesisViewUtils.ts          # 视图工具类
│   └── ViewMode/                   # ViewMode 实现
├── Typing/                         # TypeScript 类型定义
└── tsconfig.json                   # TypeScript 配置
```

## 常用命令

### 构建项目
```bash
# 生成 Visual Studio 项目文件
UnrealBuildTool.exe -ProjectFiles -Project="D:\GitDir\NoesisDemo\NoesisDemo.uproject"

# 编译 C++ 代码
UnrealBuildTool.exe NoesisDemo Win64 Development -Project="D:\GitDir\NoesisDemo\NoesisDemo.uproject"
```

### TypeScript 开发
```bash
# 在项目根目录编译 TypeScript（需要全局安装 tsc）
tsc

# 监视模式（自动编译）
tsc --watch

# 注意：项目根目录没有 package.json，不使用 npm 命令
```

### 运行项目
```bash
# 在编辑器中运行，PuerTS 会自动加载 TypeScript 代码
UnrealEditor.exe "D:\GitDir\NoesisDemo\NoesisDemo.uproject"
```

## 文件组织与映射

### XAML → ViewMode → TypeScript 映射

| XAML 文件 | UE 资源路径 | TypeScript ViewMode | 蓝图类路径 |
|-----------|-------------|---------------------|-----------|
| `Assets/GUI/MainPage.xaml` | `/Game/GUI/MainPage` | `TS_NoesisViewMode.ts` | `/Game/BluePrints/TypeScript/ViewMode/TS_NoesisViewMode` |
| `Assets/GUI/Buttons/MainWindow.xaml` | `/Game/GUI/Buttons/MainWindow` | `TS_ButtonsViewMode.ts` | `/Game/BluePrints/TypeScript/ViewMode/Buttons/TS_ButtonsViewMode` |

### TypeScript 编译配置
```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "commonjs",
    "experimentalDecorators": true,    // 支持装饰器
    "outDir": "Content/JavaScript",     // 输出到 Content/JavaScript
    "typeRoots": ["Typing", "./node_modules/@types"]
  },
  "include": ["TypeScript/**/*"]
}
```

## 关键类型映射

### C++ 属性类型到 TypeScript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [No-needto-recall/NoesisDemo](https://github.com/No-needto-recall/NoesisDemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
