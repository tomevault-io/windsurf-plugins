---
trigger: always_on
description: > **规则映射说明**：本文档仅记录本项目特定的技术规则与上下文。关于 Git 工作流、PR 规范、文档原则、Agent 执行规范、编码与工程原则等用户级通用规则，请查阅 ~/.kimi/AGENTS.md（dlcv_mcp 仓库）。
---

# OpenIVS Agent 规则

> **规则映射说明**：本文档仅记录本项目特定的技术规则与上下文。关于 Git 工作流、PR 规范、文档原则、Agent 执行规范、编码与工程原则等用户级通用规则，请查阅 ~/.kimi/AGENTS.md（dlcv_mcp 仓库）。

## 项目概览

OpenIVS 是一个 .NET WPF 工业视觉框架。**本 AGENTS.md 聚焦 API 层（C++ DLL + C# 封装层）与测试程序**，不展开 WPF 框架本身（相机/PLC/主循环）。WPF 主框架构建在 API 层之上。

- **技术栈**：C# .NET Framework 4.7.2 + C++17 + Qt + WPF
- **平台**：Windows 10+，x64
- **API 层**：
  - C++ API：`dlcv_infer_cpp_dll`（头文件 `dlcv_infer.h`、`dlcv_sntl_admin.h`、`flow/FlowGraphModel.h`）
  - C API：`dlcv_infer_c_dll`（头文件 `dlcv_infer_c_api.h`，通过 `dlcv_infer_cpp_dll.lib` 依赖 C++ API）
  - C# API：`DlcvCsharpApi`（`Model.cs`、`Utils.cs`、`FlowGraphModel.cs`、`DvsModel.cs`、`DllLoader.cs`）
- **测试程序**：
  - C++：`dlcv_infer_cpp_qt_demo`（Qt 桌面应用）
  - C#：`DlcvDemo` / `DlcvDemo2` / `DlcvDemo3`（WinForms 桌面应用）
- **控制台测试**：`Test/DlcvCSharpTest`、`Test/dlcv_infer_cpp_test`、`Test/dlcv_infer_c_test`

## 构建方式

- **编译时优先使用项目 skill**：`.cursor/skills/vs-build/scripts/build.py`
- **禁止直接调用** `msbuild`、`dotnet`、`devenv` 或其他本地 shell 编译命令
- **默认构建参数**：`Debug`、`x64`、`Build`、`minimal`
- 用户明确指定 `Release`、`Rebuild`、`Clean` 等参数时，按指定值执行
- .NET 项目通过 Visual Studio 或 `dotnet build` 编译
- C++ 项目通过 Visual Studio 编译（x64 Release）
- Qt 项目需配置 Qt 路径和 OpenCV 路径
- 构建前需确保深度视觉 SDK 已正确安装（`dlcv_infer.dll` 可用）
- WPF 框架额外需要海康 MVS 安装

## 统一运行与验证输入规则

- 本仓库所有程序、测试程序、自测入口、验证入口和临时排查入口均**禁止使用命令行传参**覆盖模型路径、图片路径、batch、阈值、运行次数、线程数或其他业务输入。
- 需要切换验证输入时，只修改源码中的固定变量、常量字符串或配置对象字段。
- 控制台测试程序也遵守该规则；验证时不通过 shell 命令行追加参数。

## 核心模块与入口

| 关注点 | 文件 | 说明 |
|--------|------|------|
| C++ API 头文件 | `dlcv_infer_cpp_dll/dlcv_infer.h` | `Model`、`SlidingWindowModel`、`Utils`、`DllLoader`、`GetAllDogInfo` |
| C++ API 实现 | `dlcv_infer_cpp_dll/dlcv_infer.cpp` | 模型加载、推理、DVS 解包、结果解析 |
| C++ 加密狗 | `dlcv_infer_cpp_dll/dlcv_sntl_admin.cpp` | Sentinel/Virbox 设备与 feature 查询 |
| C++ 流程图 | `dlcv_infer_cpp_dll/flow/FlowGraphModel.h` | `FlowGraphModel` 类 |
| C# 封装层 | `DlcvCsharpApi/Model.cs` | `Model`：构造、加载、推理、释放 |
| C# 工具类 | `DlcvCsharpApi/Utils.cs` | 结果类型、编码转换、DLL 释放 |
| C# DLL 加载器 | `DlcvCsharpApi/DllLoader.cs` | 加密狗自动检测、DLL 路径解析、函数代理 |
| C# 流程图 | `DlcvCsharpApi/flow/FlowGraphModel.cs` | `FlowGraphModel`：加载、推理、JSON 输出 |
| C# DVS 模型 | `DlcvCsharpApi/flow/DvsModel.cs` | `.dvst/.dvso/.dvsp` 归档解包与加载 |
| C# 结果类型 | `DlcvCsharpApi/DataTypes.cs` | `CSharpObjectResult`、`CSharpSampleResult`、`CSharpResult` |
| C# 加密狗工具 | `DlcvCsharpApi/sntl_admin_csharp.cs` | `DogUtils`、`DogProvider` |
| C++ 图像输入 | `dlcv_infer_cpp_dll/ImageInputUtils.h` | 图像预处理与格式转换 |
| C++ 测试程序 | `dlcv_infer_cpp_qt_demo/MainWindow.cpp` | 模型加载、推理、压力测试、加密狗检测 |
| C# 测试程序 | `DlcvDemo/Form1.cs` | WinForms 测试程序主窗口 |
| C# 压力测试 | `PressureTestRunner/PressureTestRunner.cs` | 多线程/一致性测试框架 |

## 常见修改点

- **API 层修改**：
  - 新增结果字段 → 同步修改 `C++ API文档.md`、`C# API文档.md`
  - 修改图像预处理逻辑 → 同步检查 C++ `dlcv_infer.cpp` 与 C# `Model.cs` 的 `PrepareInferImages`
  - 新增模型格式支持 → 同步更新 `DllLoader`（C++ 与 C#）的 `ResolveProviderFromHeader`
- **测试程序修改**：
  - 新增推理参数 → 同步更新 C++ `MainWindow.cpp` 与 C# `Form1.cs` 的参数 JSON 构建
  - 修改可视化规则 → 同步检查 C++ `ImageViewerWidget` 与 C# `ImageViewer`

## 关键依赖路径

- **DLCV 推理 DLL（必须）**
  - `dlcv_infer.dll`（Sentinel）或 `dlcv_infer_v.dll`（Virbox）
  - 路径：`C:\dlcv\Lib\site-packages\dlcvpro_infer\dlcv_infer.dll`
  - 未安装时测试程序启动会弹窗提示"需要先安装 dlcv_infer"
- **海康 MVS DLL（WPF 框架使用）**
  - `C:\Program Files (x86)\MVS\Development\DotNet\win64\MvCameraControl.Net.dll`
  - 未安装 MVS 会出现找不到 `MvCameraControl` 的问题
- **Sentinel Admin API 库（Linux）**
  - `dlcv_infer_cpp_dll` 的加密狗检测依赖 `libsntl_adminapi.so`
  - 固定查找路径：`/usr/local/dlcv/lib/libsntl_adminapi.so`
  - 该路径不存在时，Sentinel 加密狗检测返回空列表

## 模型文件类型

对外使用时，模型文件只分为两类：

1. **普通模型文件**：`.dvt`、`.dvo`。由 `Model` 直接加载，适合单模型推理。
2. **流程模型文件**：`.dvst`、`.dvso`。由 `FlowGraphModel` 或 `DvsModel` 加载，适合把多步处理组织成一条完整流程。

调用端不需要为这两类模型准备两套完全不同的调用方式。传入模型路径、设备和请求参数后，入口对象会完成对应的加载与执行。

## API 速查表

### C++ API 速查表

| 能力 | 类/函数 | 关键接口 |
|------|---------|----------|
| 普通模型 | `dlcv_infer::Model` | 构造（`std::string`/`std::wstring` + `device_id`）、`Infer()`、`InferBatch()`、`InferOneOutJson()`、`GetModelInfo()`、`FreeModel()` |
| 滑动窗口模型 | `dlcv_infer::SlidingWindowModel` | 继承 `Model`，构造参数含 `small_img_width/height`、`horizontal/vertical_overlap`、`threshold`、`iou_threshold`、`combine_ios_threshold` |
| 工具类 | `dlcv_infer::Utils` | `FreeAllModels()`、`GetDeviceInfo()`、`GetGpuInfo()`、`KeepMaxClock()`、`OcrInfer()`、`JsonToString()` |
| DLL 加载器 | `dlcv_infer::DllLoader` | `Instance()`、`EnsureForModel()`、`GetDogProvider()` |
| 流程图模型 | `dlcv_infer::flow::FlowGraphModel` | `Load()`、`InferInternal()`、`GetModelInfo()` |
| 加密狗查询 | `dlcv_infer::GetAllDogInfo()` | 返回 Sentinel + Virbox 设备与 feature 列表 |
| 编码转换 | `dlcv_infer::convert*()` | `Utf8↔Gbk`、`Utf8↔Wide`、`Ansi↔Wide` |

### C API 速查表

| 能力 | 函数 | 关键接口 |
|------|------|----------|
| 加载模型 | `dlcv_infer_cpp_load_model_c` | `const char* model_path, int device_id` → 返回 `model_index` |
| 释放模型 | `dlcv_infer_cpp_free_model_c` | `int model_index` → 返回 `0`/` -1` |
| 推理 | `dlcv_infer_cpp_infer_c` | `int model_index, const DlcvCImageList* image_list` → 返回 `DlcvCResult` |
| 释放结果 | `dlcv_infer_cpp_free_model_result_c` | `DlcvCResult* result` |

**数据结构**：复用底层 `dlcv_infer/dlcv_data_type_c.h` 中的 `DlcvCImage`、`DlcvCImageList`、`DlcvCObjectResult`、`DlcvCSampleResult`、`DlcvCResult`。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dl-cv/OpenIVS](https://github.com/dl-cv/OpenIVS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
