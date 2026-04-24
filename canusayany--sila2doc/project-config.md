---
trigger: always_on
description: Generator文件夹是受保护的尽量不要修改
---



# SiLA2 D3驱动生成工具 - 项目文档

Generator文件夹是受保护的尽量不要修改
---

## 目录

1. [项目概述](#一项目概述)
2. [核心功能](#二核心功能)
3. [UI设计](#三ui设计)
4. [数据模型](#四数据模型)
5. [关键技术点](#五关键技术点)
6. [实施历史与关键改进](#六实施历史与关键改进)
7. [测试覆盖](#七测试覆盖)
8. [使用指南](#八使用指南)

---

## 一、项目概述

### 1.1 项目定位

本工具是"**一站式设备驱动生成工具**"的SiLA2/gRPC协议模块，用于从SiLA2服务器或本地特性文件自动生成符合D3系统规范的驱动代码。

### 1.2 核心架构

```
┌────────────────────────────────────────────────────────────┐
│  SiLA2 特性源                                               │
│  • 在线服务器（mDNS扫描）                                    │
│  • 本地 .sila.xml 文件                                      │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  Tecan Generator (第三方工具)                               │
│  • 生成接口定义 (Interface)                                 │
│  • 生成客户端实现 (Client)                                  │
│  • 生成数据传输对象 (DTOs)                                  │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  代码分析服务 (ClientCodeAnalyzer)                          │
│  • 反射分析生成的代码                                       │
│  • 提取方法、参数、返回值信息                               │
│  • 提取XML文档注释                                          │
│  • 检测数据类型支持情况                                     │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  方法预览与特性调整 (用户交互)                              │
│  • 用户勾选"调度方法" (MethodOperations)                    │
│  • 用户勾选"维护方法" (MethodMaintenance)                   │
│  • 提供批量操作功能                                         │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  D3驱动代码生成 (CodeDOM)                                   │
│  • AllSila2Client.cs - 多特性整合、方法平铺                │
│  • D3Driver.cs - 驱动主类、方法分类、JSON转换               │
│  • Sila2Base.cs - 基类实现                                  │
│  • CommunicationPars.cs - IP/Port配置                       │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  项目编译                                                   │
│  • 生成 .csproj 和 .sln 文件                                │
│  • 复制依赖库 (reflib)                                      │
│  • 执行 dotnet build                                        │
│  • 生成 DLL 和 XML 文档                                     │
└────────────────────────────────────────────────────────────┘
```

### 1.3 依赖工具

| 工具 | 用途 | 来源 |
|------|------|------|
| **Tecan Generator** | 从.sila.xml生成C# gRPC客户端代码 | 第三方 (Tecan) |
| **BR.PC.Device.Sila2Discovery** | SiLA2服务器的mDNS扫描和连接管理 | Bioyond |
| **Microsoft CodeDOM** | 生成D3驱动封装层代码 | .NET Framework |
| **Newtonsoft.Json** | JSON序列化/反序列化 | NuGet |
| **.NET SDK 8.0** | 项目编译和DLL生成 | Microsoft |

---

## 二、核心功能

### 2.1 完整工作流程

1. **特性选择**
   - 扫描在线服务器或导入本地.sila.xml文件
   - 支持多特性选择（限同一服务器）
   - 父节点三态显示（未选/半选/全选）

2. **设备信息输入**
   - 品牌（Brand）
   - 型号（Model）
   - 设备类型（DeviceType）
   - 开发者（Developer）

3. **客户端代码生成**
   - 使用Tecan Generator生成强类型代码
   - 自动复制必需的DLL（Tecan.Sila2、gRPC、protobuf等）
   - 自动去重重复的类型定义

4. **方法预览与分类**
   - 显示所有检测到的方法
   - 用户勾选"调度方法"和"维护方法"
   - 支持批量操作（全部设为调度/维护、清除特性）

5. **D3驱动代码生成**
   - 自动生成命名空间：`BR.ECS.DeviceDrivers.{DeviceType}.{Brand}_{Model}`
   - 生成4个核心文件：`AllSila2Client.cs`、`D3Driver.cs`、`Sila2Base.cs`、`CommunicationPars.cs`
   - 自动处理不支持的数据类型（JSON序列化/反序列化）

6. **项目编译**
   - 生成.csproj和.sln文件
   - 配置XML文档生成
   - 执行dotnet build
   - 输出DLL和XML文档文件

### 2.2 关键特性

#### 2.2.1 灵活的方法标记系统

- **IsIncluded**：是否包含在D3Driver中（默认true）
- **IsOperations**：是否为调度方法（默认false）
- **IsMaintenance**：是否为维护方法（默认根据方法名判断）

方法可以：
- ✅ 同时是调度方法和维护方法
- ✅ 只是调度方法
- ✅ 只是维护方法
- ✅ 两者都不是（不会生成到D3Driver中）

#### 2.2.2 智能类型转换

**支持的类型**（直接使用）：
- 基础类型：int, byte, sbyte, string, DateTime, double, float, bool, byte[], long, short, ushort, uint, ulong, decimal, char
- 枚举类型
- 基础类型的数组和列表

**不支持的类型**（自动转换为JsonString）：
- 复杂的自定义类型
- 嵌套的复合类型
- Tecan.Sila2.DynamicClient.DynamicObjectProperty

**转换示例**：
```csharp
// AllSila2Client.cs - 保持原样
public virtual SetAnyTypeValueResponse SetAnyTypeValue(
    DynamicObjectProperty anyTypeValue) { ... }

// D3Driver.cs - 自动转换
[MethodOperations]
public virtual string SetAnyTypeValue(string anyTypeValueJsonString)
{
    var anyTypeValue = JsonConvert.DeserializeObject<DynamicObjectProperty>(anyTypeValueJsonString);
    var result = this._sila2Device.SetAnyTypeValue(anyTypeValue);
    return JsonConvert.SerializeObject(result);
}
```

#### 2.2.3 代码去重机制

Tecan Generator在生成多个特性时可能产生重复的类型定义，系统会自动：
- 解析所有生成的C#文件
- 识别重复的顶层类型（类、枚举、结构、接口、委托）
- 注释掉重复的定义，保留第一次出现的
- 不影响嵌套类型和不同命名空间的同名类型

#### 2.2.4 DTO构造函数特性

自动为所有DTO类的默认构造函数添加 `[JsonConstructor]` 特性：
```csharp
[JsonConstructor()]
public SetAnyTypeValueRequestDto() { }
```

---

## 三、UI设计

### 3.1 主界面布局（三列式）

```
┌─────────────────────────────────────────────────────────────┐
│  [⋮] 侧边栏                │ │  主区域                       │
│  ┌──────────────────────┐  │ │  ┌─────────────────────────┐│
│  │ 🔍 扫描 📁 添加 📤 导出│  │ │  │ 生成过程信息             ││
│  ├──────────────────────┤  │ │  │ [日志显示区域]           ││

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/canusayany) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
