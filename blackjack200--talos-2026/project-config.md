---
trigger: always_on
description: > **代码定位**：Talos 是一个以 ML-style 建模、RAII 生命周期和类型安全数据流为核心的机器人视觉框架。
---

# Talos C++ 项目代码指南

> **代码定位**：Talos 是一个以 ML-style 建模、RAII 生命周期和类型安全数据流为核心的机器人视觉框架。
>
> **架构定位**：系统运行在自定义调度器之上，通过 5 级 FCS（Fire Control System）流水线组织计算；模块依赖由 DAG
> 显式描述，并在执行前完成分析，从结构上杜绝隐式共享状态与数据竞争。
>
> **核心原则**：用 `struct` 表达 product type，用 `std::variant` 表达 sum type，用 `std::expected<T, std::string>`
> 表达可恢复失败，用 RAII owner 表达资源生命周期；所有边界数据必须先解析为强类型，核心域中非法状态不可表示。

---

## 代码目录结构

```
talos-cpp/
├── crates/                      # 核心库
│   ├── primitive/               # 基础原语库（通道、ADT 辅助、线程/性能工具）
│   │   └── src/primitive/
│   │       ├── channel.hpp              # 统一通道抽象
│   │       ├── spsc_triple_buffer.hpp   # SPSC 三重缓冲
│   │       ├── spmc_triple_buffer.hpp   # SPMC 三重缓冲
│   │       ├── lazy.hpp                 # 延迟构造
│   │       ├── overloaded.hpp           # std::visit 重载辅助
│   │       ├── spin.hpp                 # 自旋等待原语
│   │       ├── performance_probe.hpp    # 性能探针
│   │       ├── system_info.hpp          # 系统信息查询
│   │       └── thread_affinity.hpp      # 线程亲和性设置
│   ├── scheduler/               # 调度器核心（World、System、DAG 依赖分析）
│   │   ├── src/scheduler/
│   │       ├── world.hpp      # World: 资源和通道容器
│   │       ├── scheduler.hpp/.cpp  # Scheduler: 调度器核心
│   │       ├── thin.hpp            # 轻量工具/薄封装
│   │       ├── demangle.*          # 类型名反混淆，错误诊断用
│   │       ├── error.*             # 调度器错误类型/错误传播
│   │       ├── error_formatter.*   # 调度器错误格式化
│   │   │   ├── system/             # System 执行模型与元信息
│   │   │   │   ├── components.hpp  # 通道/资源组件类型
│   │   │   │   ├── execution_policy.hpp # 执行策略
│   │   │   │   ├── system_meta.hpp # System 元信息与依赖描述
│   │   │   │   └── system.hpp      # System 执行模型
│   ├── fast_tf/                 # 类型安全坐标变换
│   ├── math/                    # 数学工具（SO2、Euler）
│   ├── toml/                    # TOML 配置解析
│   ├── log/                     # 日志工具
│   └── hardware/                # 硬件驱动
│       ├── at_gimbal/           # 云台控制
│       └── hik_camera_driver/   # HIK 相机驱动（x86_64）
├── src/
│   ├── fcs/                         # 火控系统主体（5 级流水线 + 标定 + 运行时）
│   │   ├── config.hpp               # FCS 总配置
│   │   ├── camera_config.hpp        # 相机配置
│   │   ├── foxglove_config.hpp      # 可视化配置
│   │   ├── core/                    # 共享领域类型、topics、时间、PNP、弹道
│   │   │   ├── channel_topics.hpp   # 调度器通道 topics
│   │   │   ├── armor_types.hpp      # 装甲板领域类型
│   │   │   ├── target_key.hpp       # 目标身份 key
│   │   │   ├── time.hpp             # 时间类型
│   │   │   ├── types.hpp/.cpp       # FCS 基础类型
│   │   │   ├── types_pnp.hpp        # PNP 类型
│   │   │   ├── math/                # FCS 数学辅助
│   │   │   └── trajectory/          # 弹道模型
│   │   ├── calibration/             # 相机内参、ChArUco、棋盘格、手眼标定
│   │   ├── chiral/                  # Chiral 数据采集/记录系统
│   │   ├── L1_sensor/               # 采集层：相机输入、输出接口、数据 parcel
│   │   ├── L2_perception/           # 感知层：armor、rune、ldm、common
│   │   ├── L3_estimation/           # 估计层：tracker、EKF、能量机关、LDM naive
│   │   ├── L4_planning/             # 轨迹规划层：aimer、gimbal planner、目标选择、弹道构建
│   │   ├── L5_weapon/               # 武器/火控层：fire decision、fire control
│   │   ├── runtime/                 # 启动、配置加载、采集器、L1/L2 注册
│   │   └── tests/                   # FCS 单元测试与算法回归测试
│   ├── fcs_visualization/           # Foxglove 可视化
│   │   ├── foxglove_server.*        # Foxglove WebSocket server
│   │   ├── foxglove_sink.hpp        # 可视化消息 sink
│   │   ├── foxglove_systems.*       # 可视化系统注册
│   │   ├── foxglove_types.hpp       # 可视化类型
│   │   ├── scene_builder.*          # 场景构建
│   │   ├── system_helpers.hpp       # 可视化系统辅助
│   │   ├── tactical_palette.hpp     # 战术配色
│   │   ├── utility.hpp              # 可视化工具
│   │   └── systems/                 # 各流水线层的可视化系统
│   ├── main.cpp                     # 主程序入口
│   ├── playground.cpp               # 实验入口
│   └── quanta_ipc_demo.cpp          # Quanta IPC demo
├── config/                      # 配置文件
├── 3dparty/                     # 第三方库
└── cmake/                       # CMake 模块
```

---

## 库结构

| 库名                | 类型        | 说明                       |
|-------------------|-----------|--------------------------|
| toml              | INTERFACE | TOML 配置解析辅助              |
| log               | INTERFACE | 日志工具封装                   |
| math              | INTERFACE | 数学工具（SO2、Euler）          |
| primitive         | SHARED    | 原语库（通道、lazy、spin、性能探针）   |
| scheduler         | SHARED    | 调度器                      |
| fast_tf           | SHARED    | 类型安全坐标变换                 |
| at_gimbal         | INTERFACE | 云台控制                     |
| hik_camera_driver | SHARED    | HIK 相机驱动（仅 Linux x86_64） |
| hardware_daedalus | SHARED    | 硬件抽象层（共享内存）              |
| fcs               | SHARED    | 火控系统（使用 PCH 加速 50-70%）   |
| fcs_visualization | SHARED    | Foxglove 可视化             |

---

## 调度器编程模型

### System 定义

```cpp
// System 通过通道 (Channel) 和资源 (Resource) 访问数据
struct MySystem {
  // 通道访问
  spsc<Input, TagInput> reader;       // SPSC 只读
  spmc_mut<Output, TagOutput> writer; // SPMC 只写

  // 资源访问
  res<Config> config;                 // 只读资源（版本追踪）
  res_mut<State> state;               // 可写资源（写后版本号自增）
  local<Temp> temp;                   // 系统本地变量（不参与依赖分析）

  // bind(): 预创建通道（初始化期调用）
  void bind(World& world);

  // run(): 执行系统逻辑
  void run(World& world);
};
```

### 执行策略


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blackjack200/talos_2026](https://github.com/Blackjack200/talos_2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
