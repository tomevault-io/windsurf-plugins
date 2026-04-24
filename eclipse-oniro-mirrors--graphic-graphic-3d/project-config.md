---
trigger: always_on
description: 本文件为 Claude Code (claude.ai/code) 在处理此代码仓库时提供指导。
---

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在处理此代码仓库时提供指导。

## 项目概述

这是用于 OpenHarmony 的 **AGP (Ark Graphics Platform) 3D 引擎** —— 一个高性能、跨平台的实时 3D 渲染引擎。该引擎采用实体-组件-系统（ECS）架构，支持 OpenGL ES 和 Vulkan 后端。

## 构建命令

构建整个 graphic_3d 组件：
```bash
./build.sh --product-name rk3568 --build-target graphic_3d
```

构建输出（生成于 `out/` 目录）：
- `lib3dWidgetAdapter.z.so` - ArkUI 的 Widget 适配器
- `libAGPDLL.z.so` - 核心引擎 DLL
- `libPluginAGP3D.z.so` - 3D 渲染插件
- `libPluginAGPRender.z.so` - 渲染后端插件
- `libPluginAGPJpg.z.so` - JPG 图像解码插件
- `libPluginAGPPng.z.so` - PNG 图像解码插件
- `libPluginCamPreview.z.so` - 相机预览流插件，目前仅在AI Engine场景默认使能
- `libPluginDotfield.z.so` - 点云渲特效染插件
- `libPluginSceneWidget.z.so` - 场景渲染插件
- `libPluginMetaObject.z.so` - 元对象插件

构建类型在 `lume/lume_config.gni` 中配置（默认：Release）。

## 测试

测试使用 `ohos_unittest` 框架（Google Test）。每个主要组件都有 `test/` 子目录及其各自的 `BUILD.gn`。

运行所有测试：
```bash
./build.sh --product-name rk3568 --build-target graphic_3d_test
```

各个测试目标遵循每个测试 `BUILD.gn` 中定义的模式。

## 架构

代码库遵循分层架构：

```
应用层 (ArkUI / NAPI / ETS)
         ↓
适配层 (3d_widget_adapter, 3d_scene_adapter)
         ↓
绑定层 (kits/ets, kits/js)
         ↓
场景层 (LumeScene, LumeMeta)
         ↓
引擎核心层 (Lume_3D, LumeRender, LumeEngine, LumeBase)
         ↓
图形后端 (OpenGL ES / Vulkan)
```

### 核心组件 (`lume/`)

- **LumeBase** - 基础：数学库（向量、矩阵、四元数）、容器（vector、map、string）、工具类
- **LumeEngine** - 核心引擎：ECS 框架、资源管理、线程、插件系统、平台抽象
- **Lume_3D** - 3D 功能：GLTF 模型加载、动画（骨骼、变形目标）、PBR 材质、后处理（ToneMapping、Bloom、HDR、FXAA）、几何图元
- **LumeRender** - 渲染：延迟/前向管线、着色器管理、GPU 资源、多后端（GLES/Vulkan）
- **LumeMeta** - 反射：运行时类型信息、属性系统、关键帧动画、事件系统
- **LumeScene** - 场景图：分层场景表示、变换层级、相机系统、组件集成

### 集成层

- **3d_widget_adapter/** - ArkUI 集成：UI 框架集成、输入事件处理、Surface 管理、渲染配置
- **3d_scene_adapter/** - 场景管理：高级场景操作、离屏渲染
- **kits/ets/** - ETS（类 TypeScript）绑定 - 生成为静态 ABC 文件
- **kits/js/** - NAPI 绑定

### 关键架构模式

**ECS（实体-组件-系统）**：贯穿 LumeEngine 和 Lume_3D 的核心设计模式。组件是纯数据结构；系统处理具有特定组件的实体。

**插件系统**：扩展的动态加载（PNG/JPG 解码器、 场景/元数据 转发插件）。插件通过插件注册系统在运行时加载。

**多后端渲染**：着色器编译和 GPU 资源管理抽象了 OpenGL ES 和 Vulkan 的差异。

**属性系统**：LumeMeta 提供具有绑定支持的类型安全属性系统，广泛用于动画和数据流。

## 编程语言

- **C++** (98/11/14/17) - 核心引擎
- **GLSL** - 着色器
- **TypeScript** - 通过 NAPI/TAIHE 绑定

## 构建配置

关键配置文件：`lume/lume_config.gni`

## 依赖

OpenHarmony 组件（来自 `bundle.json`）：
- c_utils, hilog, graphic_2d, graphic_surface, hitrace, init, input, ipc
- napi, ability_runtime, bundle_framework, qos_manager
- 第三方：libpng, libjpeg-turbo, vulkan-loader, skia, freetype, zlib, runtime_core, meshoptimizer

---

## 深入探究：核心 Lume 模块

### 模块依赖层次

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层                                    │
│                 (ArkUI / NAPI / ETS / JS)                       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      适配层                                      │
│           (3d_widget_adapter, 3d_scene_adapter)                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      绑定层                                      │
│                  (kits/ets, kits/js)                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      场景层                                      │
│                  (LumeScene, LumeMeta)                          │
│       (场景图, 节点层级, 对象管理, 反射, 属性系统)               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     引擎核心层                                   │
│            (Lume_3D, LumeRender, LumeEngine, LumeBase)          │
│  (3D 渲染, GLTF, PBR 材质, 动画, 后处理, 渲染管线, ECS)         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    图形后端层                                    │
│                (OpenGL ES / Vulkan)                             │
└─────────────────────────────────────────────────────────────────┘
```

### LumeMeta (`lume/LumeMeta/`)

**用途**：提供反射、运行时类型信息和动态属性系统的基础层。

**目录结构**：
```
lume/LumeMeta/
├── include/meta/
│   ├── api/                  # 公共 API
│   ├── base/                 # 基础类型和宏
│   ├── ext/                  # 扩展功能
│   └── interface/            # 核心接口（30+）
```

**核心概念**：

1. **反射系统**：所有引擎对象的运行时类型内省
2. **属性系统**：具有更改通知的类型安全动态属性
3. **信号/槽**：事件处理机制
4. **对象工厂**：通过类型名称/ID 动态创建对象
5. **序列化**：对象保存/加载功能

### LumeScene (`lume/LumeScene/`)

**用途**：中层场景图，提供分层对象管理，并桥接高级场景概念与底层 ECS。

**目录结构**：
```
lume/LumeScene/
├── include/scene/
│   ├── api/                  # 公共场景 API
│   ├── base/                 # 基础类型和工具
│   ├── ext/                  # 扩展接口和辅助工具
│   └── interface/            # 核心接口（30+）
```

**核心类**：
- **Scene** - 所有场景对象的根容器
- **Node** - 分层场景对象的基类
- **Camera** - 带投影参数的相机节点
- **Light** - 光源节点
- **Geometry** - 网格/可渲染节点
- **Component** - 附加到节点的 ECS 组件包装器

**ECS 桥接模式**：
- 场景节点可以选择性地映射到 ECS 实体

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eclipse-oniro-mirrors) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
