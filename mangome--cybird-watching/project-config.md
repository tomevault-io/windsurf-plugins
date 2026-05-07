---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个名为"CybirdWatching"的嵌入式项目，基于ESP32平台开发。该项目是一个智能显示设备，集成了多种传感器、显示模块、网络连接和用户界面功能。项目使用Arduino框架和PlatformIO进行开发。

## 开发环境

- **平台**: ESP32 (pico32)
- **框架**: Arduino
- **构建系统**: PlatformIO
- **串口速度**: 115200
- **上传端口**: COM3（在 platformio.ini 中配置）

## 构建和开发命令

### 常用PlatformIO命令
```bash
# 编译项目
platformio run

# 上传到设备
platformio run --target upload

# 监控串口输出
platformio device monitor

# 清理构建文件
platformio run --target clean

# 上传并监控（推荐）
platformio run --target upload && platformio device monitor
```

### Windows快捷脚本（在 scripts/ 目录）
```bash
# 编译项目
.\pio_run.bat

# 上传并监控
.\upload_and_monitor.bat

# CLI工具（交互式串口命令）
.\cybird_cli.bat
```

## 项目架构

### v3.0 双核FreeRTOS架构

项目采用双核分离设计，充分利用ESP32的双核优势：

```
┌─────────────────────────────────────────────────────┐
│                     ESP32                           │
├─────────────────────┬───────────────────────────────┤
│   Core 0 (UI)       │   Core 1 (System)             │
│   优先级: 2         │   优先级: 1                    │
│   栈: 8KB           │   栈: 8KB                      │
│   频率: 200Hz       │   频率: 100Hz                  │
├─────────────────────┼───────────────────────────────┤
│ • LVGL GUI          │ • IMU 传感器                   │
│ • Display 刷新      │ • 串口命令                     │
│ • 小鸟动画          │ • WiFi 通信                    │
│ • 图片解码          │ • SD 卡操作                    │
│                     │ • 业务逻辑                     │
└─────────────────────┴───────────────────────────────┘
           ↕                        ↕
    ┌──────────────────────────────────────┐
    │   消息队列 + LVGL 互斥锁              │
    └──────────────────────────────────────┘
```

**关键要点**:
- UI任务运行在Core 0，专注于渲染，保证界面流畅
- 系统任务运行在Core 1，处理传感器、IO和业务逻辑
- 通过LVGL互斥锁保证线程安全
- 通过消息队列实现任务间通信
- 详见: [双核架构文档](docs/DUAL_CORE_ARCHITECTURE.md)

### 目录结构
```
src/
├── main.cpp                           # 主程序入口
├── drivers/                           # 硬件驱动层
│   ├── display/                       # 显示驱动 (TFT_eSPI)
│   ├── sensors/                       # 传感器驱动
│   │   ├── imu/                       # IMU传感器 (MPU6050)
│   │   └── ambient/                   # 环境传感器
│   ├── communication/network/         # 网络通信 (WiFi)
│   ├── storage/sd_card/               # SD卡存储
│   └── io/rgb_led/                    # RGB LED控制 (FastLED)
├── system/                            # 系统服务层
│   ├── logging/                       # 日志管理系统
│   ├── commands/                      # 串口命令系统
│   └── lvgl/ports/                    # LVGL端口层
└── applications/                      # 应用层
    └── gui/                           # 图形用户界面
        ├── core/                      # 核心GUI组件
        └── screens/                   # GUI屏幕
```

### 核心组件

#### 1. 显示系统 (`src/drivers/display/`)
- 基于TFT_eSPI库的LCD显示控制
- LVGL图形库集成
- 支持背光PWM控制
- 负责屏幕初始化和日常刷新

#### 2. 传感器系统 (`src/drivers/sensors/`)
- **IMU**: MPU6050加速度计和陀螺仪，用作用户输入设备
- **环境传感器**: 环境光线、温度等传感器支持

#### 3. 网络通信 (`src/drivers/communication/network/`)
- WiFi网络连接管理
- 从SD卡读取WiFi配置 (`/wifi.txt`)
- 支持网络数据传输功能

#### 4. 存储系统 (`src/drivers/storage/sd_card/`)
- SD卡文件系统支持
- 日志文件存储
- 配置文件读取

#### 5. 用户界面 (`src/applications/gui/`)
- 基于LVGL的图形用户界面
- 包含主屏幕和场景屏幕
- 支持触摸和IMU输入

#### 6. 日志系统 (`src/system/logging/`)
- 单例模式的日志管理器
- 支持串口和SD卡输出
- 多级日志 (DEBUG, INFO, WARN, ERROR, FATAL)
- 日志文件自动轮转
- 专为嵌入式环境优化

#### 7. 命令系统 (`src/system/commands/`)
- 串口命令解析和执行
- 支持系统控制和调试命令
- 非阻塞式命令处理

### 外部依赖库

#### 核心库 (`lib/`)
- **FastLED**: 高性能LED控制库，用于RGB LED
- **MPU6050**: IMU传感器驱动库
- **TFT_eSPI**: 高性能TFT LCD驱动库
- **LVGL**: 轻量级图形用户界面库

### 系统初始化流程

v3.0双核架构的初始化顺序（详见 src/main.cpp:30）：

1. 看门狗配置 (10秒超时，防止图像加载时触发)
2. 串口初始化 (115200 baud)
3. 日志系统初始化 (先串口输出)
4. 串口命令系统初始化
5. **SD卡初始化** (必须在显示屏之前，避免SPI冲突)
6. 日志系统切换为SD卡输出（保持CLI输出清洁）
7. 显示屏初始化和背光设置 (0.2亮度)
8. LVGL文件系统初始化
9. LVGL输入设备初始化
10. IMU (MPU6050) 初始化
11. RGB LED初始化
12. GUI界面创建 (setup_ui)
13. **Task Manager初始化** (创建LVGL互斥锁)
14. **双核任务启动** (Core 0: UI任务 200Hz, Core 1: 系统任务 100Hz)
15. Logo显示 (lv_init_gui)
16. **Bird Watching系统初始化** (扫描小鸟资源，耗时操作)
17. Logo关闭，切换到小鸟界面

**⚠️ 关键时序要求**:
- SD卡必须在显示屏之前初始化（避免SPI总线冲突）
- Task Manager必须在双核任务启动前初始化（提供LVGL互斥锁）
- 双核任务必须在logo显示前启动（UI任务负责渲染）
- 小鸟资源扫描在logo显示期间进行（提升用户体验）

### 主循环逻辑（v3.0双核架构）

在v3.0双核架构中，主loop已经不再承担核心功能：

```cpp
void loop() {
    // 所有核心功能已经在FreeRTOS任务中运行：
    // - Core 0: UI Task (200Hz - LVGL + Display + Animation)
    // - Core 1: System Task (100Hz - Sensors + Commands + Business Logic)

    // 可选：每60秒打印一次任务统计信息
    if (taskManager) {
        taskManager->printTaskStats();
    }

    delay(1000);  // 让出CPU给FreeRTOS调度器
}
```

**实际任务分配**:
- **UI任务** (Core 0, 200Hz): `screen.routine()` + `lv_timer_handler()` + 动画播放
- **系统任务** (Core 1, 100Hz): `mpu.update(200)` + `SerialCommands::handleInput()` + BirdManager逻辑

### 配置文件

- **WiFi配置**: `/wifi.txt` (第1行SSID, 第2行密码)
- **小鸟配置**: `/configs/bird_config.csv` - 小鸟ID、名称、权重配置
- **项目配置**: `platformio.ini` - 包含构建标志和库依赖
- **包含路径**: 所有模块的包含路径已在platformio.ini中配置

### 常用串口命令参考

使用 `cybird_cli.bat` 或串口工具连接后，常用命令：

```bash
# 系统命令
help                    # 显示所有命令
status                  # 显示系统状态
clear                   # 清空终端

# 小鸟系统
bird trigger [id]       # 触发小鸟动画（可选指定ID）
bird list               # 列出所有小鸟
bird stats              # 查看观鸟统计

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mangome/cybird-watching](https://github.com/Mangome/cybird-watching) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
