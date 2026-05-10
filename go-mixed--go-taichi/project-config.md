---
trigger: always_on
description: > **项目类型**: Golang封装的Taichi C-API跨平台绑定
---

# Go-Taichi 项目上下文

> **项目类型**: Golang封装的Taichi C-API跨平台绑定
> **最后更新**: 2026-02-19
> **Taichi版本**: v1.7.0 (C-API v1007000)
> **Go版本**: 1.25+

---

## 项目概述

Go-Taichi是Taichi C-API的Go语言绑定，使用**purego**实现跨平台支持（Windows/Linux/macOS），**无需CGo编译**。

### 核心特性

- ✅ **跨平台支持** - Windows(DLL) / Linux(SO) / macOS(Dylib)
- ✅ **无需CGo** - 使用purego实现，纯Go编译 (`CGO_ENABLED=0`)
- ✅ **简洁API** - 高级抽象层，自动资源管理
- ✅ **完整覆盖** - 覆盖所有Taichi C-API v1.7.0功能
- ✅ **类型安全** - 完整的类型系统映射
- ✅ **自动管理** - defer Release() 模式，无需手动清理

---

## 项目架构

### 双层设计

```
┌─────────────────────────────────────┐
│   用户代码 (import "go-taichi/taichi")  │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   高级抽象层 (taichi/)               │
│   • Runtime      - 运行时管理        │
│   • Memory       - 内存基类          │
│   • NdArray      - N维数组           │
│   • Image        - 图像处理          │
│   • AotModule    - AOT模块           │
│   • Kernel       - Kernel执行        │
│   • Sampler      - 采样器            │
│   • MemoryImport - 内存导入          │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   C-API绑定层 (taichi/c_api/)       │
│   • 纯C函数绑定                      │
│   • 类型定义                         │
│   • 跨平台加载                       │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   Taichi C动态库                     │
│   taichi_c_api.dll/.so/.dylib       │
└─────────────────────────────────────┘
```

### 目录结构

```
go-taichi/
├── taichi/                    # 公共API包（用户导入这个）
│   ├── c_api/                # C-API绑定（内部包）
│   │   ├── include/          # C头文件参考（仅用于生成Go API）
│   │   │   └── taichi/taichi_core.h
│   │   ├── init.go           # 跨平台初始化入口
│   │   ├── init_posix.go     # Linux/macOS加载 (purego.Dlopen)
│   │   ├── types.go          # 类型定义
│   │   ├── core.go           # 核心API
│   │   ├── memory_ops.go     # 内存操作
│   │   ├── image_ops.go      # 图像操作
│   │   ├── aot.go            # AOT模块
│   │   └── helpers.go        # 辅助函数
│   ├── taichi.go             # 包入口（导出常量、类型别名）
│   ├── runtime.go            # Runtime抽象
│   ├── memory.go             # Memory基类
│   ├── memory_import.go      # 内存导入（CPU/CUDA）
│   ├── ndarray.go            # NdArray抽象
│   ├── image.go              # Image抽象
│   ├── sampler.go            # Sampler抽象
│   └── aot.go                # AotModule/Kernel抽象
├── docs/                     # API文档
│   ├── high_api/             # 高级API文档
│   └── low_api/              # 低级C-API文档
├── examples/                 # 示例代码
│   ├── 01_runtime.go         # Runtime管理
│   ├── 02_ndarray_1d.go      # 1D数组
│   ├── 03_ndarray_2d.go      # 2D矩阵
│   ├── 04_image.go           # 图像处理
│   ├── 10_aot_kernel.go      # AOT Kernel
│   ├── 11_aot_async.go       # 异步执行
│   ├── 12_aot_batch.go       # 批量执行
│   ├── 13_compute_graph.go   # 计算图
│   ├── 20_memory_cpu.go      # CPU内存导入
│   ├── 21_memory_cuda.go     # CUDA内存导入
│   └── README.md             # 示例说明
├── CLAUDE.md                 # 本文件（完整文档）
└── README.md                 # 项目入口

**设计原则**：用户只导入 `go-taichi/taichi`，无需直接使用 `c_api`
```

---

## API使用方式

### 低级API

```go
archs := c_api.GetAvailableArchs()
runtime := c_api.CreateRuntime(archs[0], 0)
allocInfo := c_api.TiMemoryAllocateInfo{...}
memory := c_api.AllocateMemory(runtime, &allocInfo)
ptr := c_api.MapMemory(runtime, memory)
// ... 需要手动释放 ...
c_api.UnmapMemory(runtime, memory)
c_api.FreeMemory(runtime, memory)
c_api.DestroyRuntime(runtime)
```

### 高级API

```go
// 方式1：从系统PATH加载动态库
runtime, _ := taichi.NewRuntimeAuto("")
defer runtime.Release()

// 方式2：指定动态库目录
runtime, _ := taichi.NewRuntimeAuto("./lib")
defer runtime.Release()

arr, _ := taichi.NewNdArray1D(runtime, 1000, taichi.DATA_TYPE_F32)
defer arr.Release()

data, _ := arr.AsSliceFloat32()
// ... 使用数据 ...
arr.Unmap()
```

---

## 类型系统

### C API常量命名规范

**保持C语言风格**（约定俗成，不改为Go风格）：

- `TI_TRUE` / `TI_FALSE` - 布尔值
- `TI_ARCH_VULKAN` / `TI_ARCH_CUDA` - 架构类型
- `TI_FORMAT_RGBA8` / `TI_FORMAT_R32F` - 纹理格式
- `TI_DATA_TYPE_F32` / `TI_DATA_TYPE_I32` - 数据类型
- `TI_NULL_HANDLE` - 空句柄

**原因**：与官方C API文档保持一致，便于查阅和对照

### 句柄类型（7种）

```go
type TiRuntime      uintptr  // 运行时
type TiAotModule    uintptr  // AOT模块
type TiMemory       uintptr  // 内存
type TiImage        uintptr  // 图像
type TiSampler      uintptr  // 采样器
type TiKernel       uintptr  // Kernel
type TiComputeGraph uintptr  // 计算图
```

### 架构类型（8种）

```go
TI_ARCH_VULKAN    // Vulkan (推荐，跨平台)
TI_ARCH_CUDA      // NVIDIA CUDA
TI_ARCH_METAL     // Apple Metal
TI_ARCH_X64       // x64 CPU
TI_ARCH_ARM64     // ARM64 CPU
TI_ARCH_OPENGL    // OpenGL
TI_ARCH_GLES      // OpenGL ES
```

### 数据类型（14种）

```go
TI_DATA_TYPE_F16/F32/F64    // 浮点数
TI_DATA_TYPE_I8/I16/I32/I64 // 有符号整数
TI_DATA_TYPE_U1/U8/U16/U32/U64 // 无符号整数
```

### 纹理格式（44种）

```go
TI_FORMAT_RGBA8/RGBA16F/RGBA32F  // 常用格式
TI_FORMAT_R8/R16/R32F            // 单通道
TI_FORMAT_DEPTH16/DEPTH32F       // 深度格式
// ... 完整列表见 types.go
```

---

## 高级抽象API

### Runtime - 运行时管理

```go
// 自动选择最佳架构（从当前目录或系统PATH加载）
runtime, err := taichi.NewRuntimeAuto("")
defer runtime.Release()

// 自动选择最佳架构（从指定目录加载）
runtime, err := taichi.NewRuntimeAuto("./lib")
defer runtime.Release()

// 手动指定架构
runtime, err := taichi.NewRuntime(taichi.ArchVulkan, "")

// 查询信息
arch := runtime.Arch()
name := runtime.ArchName() // "Vulkan", "CUDA" 等
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-mixed/go-taichi](https://github.com/go-mixed/go-taichi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
