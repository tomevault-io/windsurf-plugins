---
trigger: always_on
description: 本文档面向 AI 编程助手，提供项目背景、架构概览和开发规范。
---

# HomeBot AGENTS.md

本文档面向 AI 编程助手，提供项目背景、架构概览和开发规范。

## 项目概述

HomeBot 是一个面向家庭场景的轻量级机器人项目，采用**分层模块化架构**和 **ZeroMQ** 通信总线，支持手机遥控、语音交互、模仿学习、人体跟随等多种应用。

**核心特性：**
- 纯 Python 实现，跨平台支持（Windows、Linux、macOS、树莓派）
- ZeroMQ 通信总线，低延迟轻量级（~1MB vs ROS2 ~1GB）
- 网页遥控端，支持手机/平板/PC，实时视频流显示
- 紧急停止锁定机制，触发后需手动归位解锁
- 硬件抽象层设计，易于适配不同硬件

## 技术栈

| 组件 | 技术 |
|------|------|
| 语言 | Python 3.11+ |
| 通信 | ZeroMQ (pyzmq) |
| Web 框架 | Flask + Flask-SocketIO |
| 计算机视觉 | OpenCV, Ultralytics YOLO |
| 语音识别 | sherpa-onnx |
| 语音合成 | 火山引擎 TTS |
| LLM对话 | DeepSeek API / OpenAI |
| MCP框架 | fastmcp |
| 前端 | HTML5 + JavaScript (nippleJS 虚拟摇杆) |
| 硬件驱动 | pyserial, ftservo-python-sdk |
| 其他 | numpy, filterpy |

## 项目结构

```
homebot/
├── docs/                      # 中文文档
│   ├── 软件架构与开发规划.md
│   ├── 技术方案选型.md
│   ├── 人体跟随使用指南.md
│   ├── 人体检测与跟随方案.md
│   └── 网页控制端使用指南.md
├── hardware/                  # 硬件设计文件（SolidWorks, STL）
│   └── structure/
├── software/                  # 软件代码
│   ├── src/
│   │   ├── common/            # 公共工具、消息定义、ZeroMQ 辅助
│   │   ├── configs/           # 运行时配置 (config.py)
│   │   ├── applications/      # 应用层
│   │   │   ├── remote_control/    # 网页遥控端 (Flask + WebSocket)
│   │   │   ├── gamepad_control/   # 游戏手柄控制 (Xbox手柄)
│   │   │   ├── human_follow/      # 人体跟随 (YOLO + 视觉伺服)
│   │   │   ├── speech_interaction/# 语音交互
│   │   │   └── imitation_learning/# 模仿学习
│   │   ├── services/          # 服务层
│   │   │   ├── motion_service/    # 运动控制服务
│   │   │   │   ├── chassis_service.py   # 底盘服务（含仲裁器）
│   │   │   │   └── chassis_arbiter/     # 仲裁器核心
│   │   │   ├── vision_service/    # 视觉服务（图像采集发布）
│   │   │   └── speech_service/    # 语音服务
│   │   ├── hal/               # 硬件抽象层
│   │   │   ├── camera/        # 摄像头驱动
│   │   │   ├── chassis/       # 底盘驱动（三轮全向轮）
│   │   │   ├── arm/           # 机械臂驱动
│   │   │   ├── audio/         # 音频驱动
│   │   │   └── ftservo_driver.py  # 飞特舵机底层驱动
│   │   ├── examples/          # 示例代码
│   │   └── tests/             # 测试代码
│   ├── tools/                 # 辅助脚本（模型下载等）
│   ├── models/                # 机器学习模型（YOLO 等）
│   ├── start_system.py        # 跨平台系统启动器
│   ├── start_chassis_service.py
│   ├── start_human_follow.py
│   └── start_system.bat       # Windows 一键启动
├── requirements.txt           # Python 依赖
├── pyproject.toml             # 构建系统配置
├── setup.py                   # 包安装配置
└── README.md                  # 项目说明
```

## 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                        Application Layer                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Remote   │ │ Gamepad  │ │  Human   │ │  Voice   │       │
│  │ Control  │ │ Control  │ │  Follow  │ │Interaction│      │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘       │
│  ┌──────────┐ ┌──────────┐                                   │
│  │Imitation │ │  ...     │                                   │
│  │Learning  │ │          │                                   │
│  └────┬─────┘ └────┬─────┘                                   │
└───────┼────────────┼──────────────────────────────────────────┘
        │            │            │            │
        └────────────┴──────┬─────┴────────────┘
                            │ ZeroMQ
┌───────────────────────────┼─────────────────────────────────┐
│                      Service Layer                            │
│  ┌──────────────┐ ┌──────┴──────┐ ┌──────────────┐          │
│  │ Motion       │ │   Vision     │ │   Speech     │          │
│  │ Service      │ │   Service    │ │   Service    │          │
│  │(Chassis+Arm) │ │(Camera Pub)  │ │              │          │
│  └──────┬───────┘ └──────┬──────┘ └──────┬───────┘          │
└─────────┼────────────────┼───────────────┼──────────────────┘
          │                │               │
          └────────────────┼───────────────┘
                           │
┌──────────────────────────┼──────────────────────────────────┐
│                     HAL (Hardware Abstraction Layer)          │
│  ┌──────────┐ ┌──────────┼──┐ ┌──────────┐ ┌──────────┐     │
│  │ Chassis  │ │   Arm    │  │ │  Camera  │ │  Audio   │     │
│  │  Driver  │ │  Driver  │  │ │  Driver  │ │  Driver  │     │
│  └──────────┘ └──────────┘  │ └──────────┘ └──────────┘     │
│  ┌──────────┐                                             │
│  │ Gamepad  │                                             │
│  │  Driver  │                                             │
│  └──────────┘                                             │
└─────────────────────────────┴─────────────────────────────────┘
```

## 核心模块说明

### 1. 硬件抽象层 (HAL)

**底盘驱动** (`hal/chassis/driver.py`):
- 基于飞特 ST3215 舵机
- 三轮全向底盘（左前、右前、后轮）
- 实现逆运动学：速度 (vx, vy, omega) → 各轮速度
- 支持轮式模式控制

**舵机驱动** (`hal/ftservo_driver.py`):
- 封装 ftservo-python-sdk (scservo_sdk)
- 支持位置模式、速度模式（轮式）
- 自动模拟模式（SDK 未安装时）

**摄像头驱动** (`hal/camera/driver.py`):
- OpenCV 封装
- 支持分辨率、帧率配置

**游戏手柄驱动** (`hal/gamepad/`):
- 基于 Windows XInput API
- 支持 Xbox 360/One/Series X|S 手柄
- 按键、摇杆、扳机键读取
- 震动控制反馈

### 2. 服务层 (Services)

**底盘服务** (`services/motion_service/chassis_service.py`):
- ZeroMQ REP 模式监听控制指令
- **仲裁器核心逻辑**：优先级-based 控制权管理
- 紧急停止锁定机制（触发后需归位解锁）
- 1秒超时自动释放控制权

控制源优先级（从高到低）：
```python
PRIORITIES = {
    "emergency": 4,  # 紧急停止（最高）
    "auto": 3,       # 自动模式（人体跟随）
    "gamepad": 2,    # 游戏手柄控制

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [choco-robot/HomeBot](https://github.com/choco-robot/HomeBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
