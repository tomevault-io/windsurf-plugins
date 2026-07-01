---
trigger: always_on
description: 本项目是 AnyCAD Rapid.NET 框架的示例代码库，展示了三维建模、运动仿真和高级几何处理的核心功能。
---

# AnyCAD Rapid.NET Sample - AI 开发指南

## 📚 项目概述

本项目是 AnyCAD Rapid.NET 框架的示例代码库，展示了三维建模、运动仿真和高级几何处理的核心功能。

### 技术栈
- **语言**: C# .NET (.NET 6.0/8.0/10.0)
- **UI 框架**: WinForms / WPF / Avalonia (跨平台)
- **核心库**: AnyCAD.Foundation, AnyCAD.Simulate, AnyCAD.QuickSolid
- **坐标系**: 右手坐标系，Z 轴向上

---

## 🏗️ 项目结构

```
anycad.rapid.net.sample/
├── AnyCAD.Basic/              # 基础功能示例
│   ├── Analysis/             # 几何分析（曲线、曲面、距离等）
│   ├── Geometry/             # 几何造型（布尔运算、拉伸、旋转等）
│   ├── Graphics/             # 图形显示（材质、纹理、粒子等）
│   ├── Interaction/          # 交互操作（拾取、测量等）
│   └── TestCaseLoader.cs     # 测试用例加载器
├── AnyCAD.Advanced/           # 高级功能示例
│   ├── Analysis/             # 高级分析
│   ├── Graphics/             # 高级图形（CAE、点云等）
│   ├── Interaction/          # 复杂交互
│   ├── Interop/              # 数据交换（DXF/DWG 导入）
│   ├── Simulation/           # 物理仿真（碰撞、光线等）
│   └── TestCaseLoader.cs
├── AnyCAD.QuickSolid/         # 快速建模工具
│   ├── Analysis/             # 曲率分析、孔洞检测
│   ├── Curve/                # 曲线处理
│   └── Geometry/             # 特殊造型（展开、投影等）
├── AnyCAD.WPF.App/            # WPF 应用程序入口
├── AnyCAD.WinForms.App/       # WinForms 应用程序入口
├── AnyCAD.AvaloniaApp/        # Avalonia 跨平台应用入口
└── data/                      # 资源数据文件
```

---

## 🎯 核心概念

### 1. TestCase 基类

所有示例都继承自 `TestCase` 抽象类，提供以下生命周期方法：

```csharp
class MyTestCase : TestCase
{
    // 初始化：创建场景、几何体、材质等
   public override void Run(IRenderView render)
    {
        // 在此处创建和显示几何体
    }
    
    // 动画回调（如果启用动画）
   public override void Animation(IRenderView render, float time)
    {
        // time: 毫秒，从 0 开始递增
        // 用于更新物体位置、变形等
    }
    
    // 清理资源
   public override void Exit(IRenderView render)
    {
        // 禁用动画、释放资源等
    }
    
    // 选择变化回调
   public override void OnSelectionChanged(IRenderView render, PickedResult result)
    {
        // 处理用户选择的物体
    }
}
```

### 2. 场景图结构

```
Scene
├── GroupSceneNode          # 组节点（可包含多个子节点）
├── BrepSceneNode           # 边界表示节点（精确几何体）
├── PrimitiveSceneNode      # 基本几何节点（网格、线、点）
├── ParticleSceneNode       # 粒子系统节点
└── SegmentsSceneNode       # 线段节点
```

### 3. 几何创建方式

#### 方式 1: GeometryBuilder (直接创建网格)
```csharp
// 创建球体
var sphere = GeometryBuilder.CreateSphere(radius, segmentsW, segmentsH);

// 创建平面
var plane = GeometryBuilder.CreatePlane(width, height);

// 创建线条
var line = GeometryBuilder.CreateLine(startPoint, endPoint);

// 创建点
var points = GeometryBuilder.CreatePoints(positions, colors, normals);
```

#### 方式 2: ShapeBuilder + BrepSceneNode (精确 CAD 几何)
```csharp
// 创建圆柱
var shape = ShapeBuilder.MakeCylinder(axis, radius, height, angle);

// 创建球体
var shape = ShapeBuilder.MakeSphere(center, radius);

// 创建立方体
var shape = ShapeBuilder.MakeBox(axis, length, width, height);

// 显示
var node = BrepSceneNode.Create(shape, material, edgeMaterial);
render.ShowSceneNode(node);
```

#### 方式 3: SketchBuilder (草图曲线)
```csharp
// 创建直线
var line = SketchBuilder.MakeLine(point1, point2);

// 创建圆
var circle = SketchBuilder.MakeCircle(center, radius, direction);

// 创建矩形
var rect = SketchBuilder.MakeRectangle(axis, length, width, cornerRadius, closed);

// 创建椭圆
var ellipse = SketchBuilder.MakeEllipse(center, majorRadius, minorRadius, xAxis, zAxis);
```

### 4. 材质系统

```csharp
// 基础材质（用于线框、点）
var basicMat = BasicMaterial.Create("name");
basicMat.SetColor(ColorTable.Red);
basicMat.SetLineWidth(2.0f);

// Phong 材质（光滑表面）
var phongMat = MeshPhongMaterial.Create("name");
phongMat.SetColor(ColorTable.Blue);
phongMat.SetSpecular(new Vector3(0.5f, 0.5f, 0.5f));
phongMat.SetShininess(50);

// 标准材质（PBR 渲染）
var standardMat = MeshStandardMaterial.Create("name");
standardMat.SetColor(ColorTable.Green);
standardMat.SetMetalness(0.5f);
standardMat.SetRoughness(0.5f);

// 点材质
var pointsMat = PointsMaterial.Create("name");
pointsMat.SetPointSize(10.0f);
pointsMat.SetSizeAttenuation(true);

// 虚线材质
var dashedMat = LineDashedMaterial.Create("name");
dashedMat.SetDashSize(1.0f);
dashedMat.SetGapSize(0.5f);

// 透明材质
material.SetTransparent(true);
material.SetOpacity(0.5f);

// 纹理贴图
var texture = ImageTexture2D.Create(GetResourcePath("textures/image.jpg"));
material.SetColorMap(texture);
```

---

## 📖 示例分类与模板

### 类型 1: 静态几何显示

**文件位置**: `AnyCAD.Basic/Graphics/` 或 `AnyCAD.Basic/Geometry/`

```csharp
using AnyCAD.Foundation;

namespace AnyCAD.Demo.Graphics
{
    class Graphics_MyExample: TestCase
    {
       public override void Run(IRenderView render)
        {
            // 1. 创建几何体
            var shape = ShapeBuilder.MakeCylinder(GP.XOY(), 10, 50, Math.PI * 2);
            
            // 2. 创建材质
            var material = MeshPhongMaterial.Create("my-material");
            material.SetColor(ColorTable.Blue);
            
            // 3. 创建场景节点并显示
            var node = BrepSceneNode.Create(shape, material, null);
            render.ShowSceneNode(node);
        }
    }
}
```

### 类型 2: 动画模拟

**文件位置**: `AnyCAD.Advanced/Simulation/` 或 `AnyCAD.Basic/Graphics/`

```csharp
using AnyCAD.Foundation;
using System.Collections.Generic;

namespace AnyCAD.Demo.Graphics
{
    class Graphics_AnimationExample : TestCase
    {
        List<BrepSceneNode> mObjects = new List<BrepSceneNode>();
        
       public override void Run(IRenderView render)
        {
            // 创建初始物体
            var shape = GeometryBuilder.CreateSphere(5, 32, 32);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anycad/anycad.rapid.net.sample](https://github.com/anycad/anycad.rapid.net.sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
