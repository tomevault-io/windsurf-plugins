---
trigger: always_on
description: 这是一个基于SIMPLE（Semi-Implicit Method for Pressure-Linked Equations）算法的二维CFD求解器，使用有限体积法离散化Navier-Stokes方程。项目采用现代C++17开发，使用结构化网格处理二维流体力学问题，如顶盖驱动流（lid-driven cavity flow）。
---

# CFD SIMPLE Solver - 项目架构文档

## 项目概述

这是一个基于SIMPLE（Semi-Implicit Method for Pressure-Linked Equations）算法的二维CFD求解器，使用有限体积法离散化Navier-Stokes方程。项目采用现代C++17开发，使用结构化网格处理二维流体力学问题，如顶盖驱动流（lid-driven cavity flow）。

## 技术栈

- **语言**: C++17
- **构建系统**: CMake 3.14+
- **核心依赖**:
  - `Eigen3`: 线性代数运算（矩阵求解器）
  - `Boost`: `multi_array`（多维数组）、`math`（数学常量）
  - `fmt`: 格式化输出

## 项目结构

```
CPP-SIMPLE/
├── src/
│   ├── config/              # 配置参数
│   │   └── simulationConfig.hpp  # 全局仿真参数（几何、物理、边界条件等）
│   ├── grid/                # 网格模块
│   │   ├── structuredMesh.hpp
│   │   └── structuredMesh.cpp
│   ├── field/               # 场数据模块
│   │   ├── scalarField.hpp/cpp    # 标量场
│   │   ├── vectorField.hpp/cpp    # 矢量场（速度）
│   │   ├── boundaryField.hpp/cpp  # 边界条件场
│   │   └── fluidProperties.hpp/cpp # 流体物性场
│   ├── equation/            # 方程离散模块
│   │   ├── scalarEquation.hpp/cpp # 标量输运方程
│   ├── main.cpp             # 程序入口
│   ├── test.cpp/hpp         # 测试和示例
├── utils/                   # 工具函数
│   └── formatter4eigen.h    # Eigen矩阵的fmt格式化支持
├── data/                    # 输出数据目录
└── CMakeLists.txt           # 构建配置
```

## 核心模块详解

### 1. 配置模块 (`src/config/`)

**`simulationConfig.hpp`** 包含所有全局配置参数：

- **几何参数**: `Lx`, `Ly`（腔体尺寸）、`ncx`, `ncy`（网格数量）
- **物理参数**: `Re`（雷诺数）、`density`（密度）、`mu_fluid`（动力粘度）、`k_fluid`（导热率）、`cp_fluid`（比热）
- **边界条件**: `boundaryInfo[4]`（四面墙的边界条件配置）
- **求解控制**: `max_iterations`, `continuity_tolerance`, `momentum_tolerance`

**枚举类型**:
- `FACE_POSITION`: `INTERIOR`, `X_MIN`, `X_MAX`, `Y_MIN`, `Y_MAX`
- `BOUNDARY_TYPE_U`: `NONE`, `WALL`, `INLET`, `OUTLET`
- `BOUNDARY_TYPE_T`: `DIRICHLET`, `NEUMANN`, `ROBIN`

### 2. 网格模块 (`src/grid/`)

**`StructuredMesh`** 类：管理结构化网格的几何信息和拓扑

**核心数据**:
- `dx_`, `dy_`: 单元尺寸
- `cellCentersX_`, `cellCentersY_`: 单元中心坐标
- `nodeCoordinatesX_`, `nodeCoordinatesY_`: 节点坐标
- `cellFaceID_`: 每个单元的面位置信息（使用`boost::multi_array`）

**重要方法**:
- `createVolumeMesh()`: 创建体网格，计算单元中心和节点坐标
- `createBoundaryMesh()`: 识别边界单元的面位置
- `getMeshSize()`: 返回`{dx_, dy_}`
- `saveMeshInfo()`: 将网格信息导出为JSON格式到`data/mesh_info.json`

**数据布局**: `cellFaceID_[j][i]` 表示第j行第i列的单元

### 3. 场数据模块 (`src/field/`)

#### 3.1 `ScalarField` - 标量场

**用途**: 存储标量数据（温度、压力等），不处理边界条件

**核心特性**:
- 使用 `boost::multi_array<float, 2>` 存储数据
- 索引映射：`operator()(i, j)` → `data_[j][i]`（确保x方向连续访问）
- 支持初始化为配置参数中的初始温度或其他值

**接口**:
```cpp
ScalarField field;                                    // 使用默认配置初始化
ScalarField field(ncx, ncy, init_value);              // 指定尺寸和初值
field(i, j) = value;                                   // 写入
float val = field(i, j);                              // 读取
field.fill(value);                                     // 全局填充
field.data_ptr();                                      // 获取原始指针
field.ncx(), field.ncy();                             // 获取尺寸
```

#### 3.2 `VectorField` - 矢量场

**用途**: 存储矢量数据（速度场），由两个标量场组成

**核心结构**:
```cpp
struct Velocity { float u; float v; };
```

**组成**: 包含两个`ScalarField`成员`u_`和`v_`，分别存储x和y方向的速度分量

**接口**:
```cpp
VectorField velocity;
Velocity vel = velocity(i, j);                         // 读取 (u, v)
velocity.u()(i, j) = 1.0;                              // 直接访问u分量
velocity.v()(i, j) = 0.5;                              // 直接访问v分量
```

#### 3.3 `BoundaryField` - 边界条件场

**用途**: 存储和访问边界条件，为方程离散提供边界信息

**存储布局**:
- `west_[j]`, `east_[j]`: 左右边界的条件（j从0到ncy-1）
- `south_[i]`, `north_[i]`: 上下边界的条件（i从0到ncx-1）

**初始化**: 从`simulationConfig.hpp`中的`boundaryInfo`数组读取配置

**接口**:
```cpp
BoundaryField bc(ncx, ncy, boundaryInfo, 4);
bc.west(j).velocityType = WALL;                        // 访问左侧边界j位置
bc.north(i).VelocityValue = {1.0, 0.0};                // 访问上侧边界i位置
```

**边界条件结构** (`BoudaryCondition`):
- `position`: 边界位置
- `velocityType`: 速度边界类型（WALL/INLET/OUTLET）
- `VelocityValue[2]`: 速度值 (u, v)
- `temperatureType`: 温度边界类型（DIRICHLET/NEUMANN/ROBIN）
- `TemperatureValue`: 温度值
- `heatFluxValue`: 热流值

#### 3.4 `FluidPropertyField` - 流体物性场

**用途**: 存储流体物性（密度、粘度、导热率、比热），支持变物性

**核心结构**:
```cpp
struct FluidValues {
    float rho;      // 密度
    float mu;       // 动力粘度
    float k;        // 导热率
    float cp;       // 定压比热
    float nu;       // 运动粘度 = mu / rho
    float alpha;    // 热扩散率 = k / (rho * cp)
};
```

**组成**: 包含四个`ScalarField`成员：`rho_`, `mu_`, `k_`, `cp_`

**初始化**: 使用`simulationConfig.hpp`中的默认值填充

**接口**:
```cpp
FluidPropertyField props;
FluidValues p = props(i, j);                          // 读取单点物性
props.rho()(i, j) = 1.2;                               // 修改密度
props.fill(rho, mu, k, cp);                            // 全局填充常物性
props.updateFluidProperties();                         // 根据温度场更新物性（待实现）
```

### 4. 方程离散模块 (`src/equation/`)

#### 4.1 `ScalarEquation` - 标量输运方程

**用途**: 离散化标量输运方程（如动量方程、温度方程、压力修正方程）

**离散系数结构**:
```cpp
struct COEF {
    float aE;      // 东侧邻居系数
    float aW;      // 西侧邻居系数
    float aN;      // 北侧邻居系数
    float aS;      // 南侧邻居系数
    float aP;      // 中心系数
    float bsrc;    // 源项
};
```

**核心方法**（大部分待实现）:
- `resetCoefficients()`: 重置所有系数为零
- `addConvectionTerm()`: 添加对流项贡献
- `addDiffusionTerm()`: 添加扩散项贡献
- `addSourceTerm()`: 添加源项
- `addPressureGradient()`: 添加压力梯度项（仅动量方程）
- `applyBoundaries()`: 应用边界条件
- `setRelaxation(float)`: 设置欠松弛因子

**数据成员**:
- 持有网格、标量场、矢量场、边界场、物性场的引用

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HatoriChise) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
