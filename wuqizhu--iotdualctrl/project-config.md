---
trigger: always_on
description: 基于 **NXP i.MX6ULL ARM Cortex-A7** 平台的嵌入式物联网(IoT)智慧环境监控系统。
---

# CLAUDE.md — 嵌入式物联网智慧环境监控系统

本文件提供项目完整的技术文档和开发指南。

## 项目概览

基于 **NXP i.MX6ULL ARM Cortex-A7** 平台的嵌入式物联网(IoT)智慧环境监控系统。
**五层分层架构**：内核驱动层 → HAL硬件抽象层 → RPC服务层 → 应用客户端层 → 云端层。

```
硬件外设 → 内核驱动(.ko) → rpc_server(端口1234) → MQTT Bridge → MQTT Broker → Python脚本 → InfluxDB/Grafana
                                                       → Web管理界面(HTTP 8080)
                                                       → RPC Client(命令行/Qt客户端)
```

### 硬件资源

| 外设 | 引脚/接口 | 说明 |
|------|----------|------|
| LED (板载) | GPIO131 | /dev/100ask_led 驱动控制 |
| DHT11 温湿度 | GPIO115 | /dev/mydht11 驱动，中断+定时器解析单总线协议 |
| PIR 人体红外 | GPIO116 | 数字输入 (0无人/1有人) |
| 烟雾传感器 DO | GPIO117 | 数字输入 (0检测到烟雾/1正常) |
| 继电器1 (风扇) | GPIO118 | 数字输出 |
| 继电器2 (LED灯) | GPIO119 | 数字输出 |
| 光敏电阻 ADC | ADC通道3 | /sys/bus/iio/devices/iio:device0/in_voltage3_raw |
| USB摄像头 | /dev/video1 | V4L2 MJPEG 格式，640x480 |

### 软件架构总览

| 层 | 模块 | 语言 | 说明 |
|----|------|------|------|
| 驱动层 | dht11_drv.ko, led_drv.ko | C (内核) | GPIO中断+定时器解析单总线协议 |
| HAL层 | hal.h/c | C | 硬件抽象接口，所有硬件操作经过此层 |
| RPC服务层 | rpc_server (lesson5) | C | JSON-RPC over TCP，端口1234 |
| HTTP服务 | http_server (lesson5) | C | 轻量级HTTP服务器，端口8080 |
| MQTT网关 | mqtt_bridge (lesson6) | C++ | 智能网关，事件驱动上报+自动控制 |
| 数据缓存 | data_cache (lesson6) | C | 环形缓冲区，断网缓存+重传 |
| 配置管理 | config (lesson6) | C | JSON配置文件+环境变量组合加载 |
| 日志系统 | log (lesson6) | C | 分级日志，文件轮转 |
| 系统监控 | system_monitor (lesson6) | C | CPU/内存/负载/运行时间 |
| 设备认证 | device_auth (lesson6) | C | MAC地址ID + Token管理 |
| OTA升级 | ota_manager (lesson6) | C | wget下载+校验+备份+回滚 |
| 消息队列 | msg_queue (lesson6) | C | 支持优先级和超时的线程安全队列 |
| 传感器管理器 | sensor_manager (lesson6) | C | 即插即用统一管理 |
| 摄像头管理 | camera_manager (lesson5/6) | C | V4L2 MJPEG 抓拍+Base64编码 |
| 安全审计 | security_audit (lesson6) | C | 事件记录+IP锁定+证书检查 |
| 数据安全 | crypto_utils (lesson6) | C | SHA-256+XOR加密+数据脱敏 |
| 内存池 | memory_pool (lesson6) | C | 固定大小池+泄漏检测 |
| 性能监控 | perf_monitor (lesson6) | C | 监控点+快照+阈值告警 |
| 插件管理器 | plugin_manager (lesson6) | C | dlopen动态加载 |
| 设备发现 | device_discovery (lesson6) | C | UDP广播发现 |
| 共享库 | shared_lib | C | cJSON + watchdog 公共模块 |
| 云端 | mqtt_to_influxdb.py | Python | MQTT→InfluxDB桥接+HTTP图片服务+钉钉告警 |
| 可视化 | Grafana | YAML/JSON | Docker部署，预置仪表板 |

---

## 目录结构

```
.
├── lesson5/                      # RPC服务器层
│   ├── rpc_server/               # 核心RPC服务 + HTTP管理
│   │   ├── rpc_server.c          # RPC服务器主程序，注册9个方法
│   │   ├── hal.h/c               # 硬件抽象层（核心设计）
│   │   ├── dht11.h/c             # DHT11设备驱动封装（/dev/mydht11）
│   │   ├── led.h/c               # LED设备驱动封装（/dev/100ask_led）
│   │   ├── http_server.h/c       # 轻量级HTTP服务器
│   │   ├── web_api.c             # Web API实现（传感器/继电器/摄像头/配置等）
│   │   ├── camera_manager.h/c    # V4L2摄像头管理
│   │   └── www/index.html        # Web管理前端
│   ├── rpc_client/               # 命令行RPC客户端（旧版）
│   ├── jsonrpc-c/                # JSON-RPC库（静态编译）
│   ├── libev/                    # libev事件循环库（静态编译）
│   └── LED_and_TempHumi/         # Qt客户端（旧版，Qt 4.8）
├── lesson6/                      # MQTT智能网关层
│   ├── mqtt_bridge.cpp           # MQTT桥接主程序（核心）
│   ├── rpc_client.h/cpp          # RPC客户端库（线程安全，带超时重连）
│   ├── config.h/c                # 配置管理
│   ├── log.h/c                   # 日志系统
│   ├── error.h/c                 # 统一错误码框架
│   ├── data_cache.h/c            # 数据缓存（环形缓冲区）
│   ├── system_monitor.h/c        # 系统监控
│   ├── device_auth.h/c           # 设备认证
│   ├── ota_manager.h/c           # OTA升级
│   ├── msg_queue.h/c             # 消息队列
│   ├── sensor_manager.h/c        # 传感器管理器
│   ├── camera_manager.h/c        # 摄像头管理（V4L2+log版）
│   ├── security_audit.h/c        # 安全审计
│   ├── crypto_utils.h/c          # 数据安全
│   ├── memory_pool.h/c           # 内存池
│   ├── perf_monitor.h/c          # 性能监控
│   ├── plugin_manager.h/c        # 插件管理器
│   ├── device_discovery.h/c      # 设备发现
│   ├── test_cases.c              # 单元测试
│   ├── test_framework.h          # 测试框架
│   └── include/                  # 第三方库头文件
│       ├── mqttclient/           # MQTT客户端库（C语言）
│       ├── mqtt/                 # MQTT协议核心
│       ├── mbedtls/              # mbedTLS加密库
│       ├── network/              # 网络层抽象
│       ├── platform/             # 平台适配层
│       └── common/               # 公共工具
├── cloud/                        # 云端服务
│   └── mqtt_to_influxdb.py       # MQTT→InfluxDB桥接+HTTP图片+钉钉告警
├── grafana/                      # Grafana可视化
│   ├── docker-compose.yml        # Docker部署（InfluxDB+Grafana+Telegraf）
│   └── grafana/provisioning/     # 自动配置
├── shared_lib/                   # 公共静态库
│   ├── include/                  # 公共头文件
│   │   ├── cJSON.h               # JSON解析器
│   │   ├── watchdog.h            # 软件看门狗
│   │   └── shared.h              # 统一包含头文件
│   └── src/
│       ├── cJSON.c               # cJSON实现
│       └── watchdog.c            # 看门狗实现
├── 驱动库源码/                   # 内核驱动源码
│   ├── dht11/dht11_drv.c         # DHT11驱动（中断+定时器）
│   └── led/led_drv.c             # LED GPIO驱动
├── config.json                   # 系统配置文件
├── .env.example                  # 环境变量模板
├── check_all.sh                  # 一键检查脚本
├── snapshot.sh                   # 快照管理脚本
└── CLAUDE.md / CURRENT.md        # 项目文档
```

---

## 核心技术细节

### 1. JSON-RPC 协议

基于 jsonrpc-c 库 + libev 事件循环，TCP 端口 1234。

**注册的9个RPC方法：**

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `led_control` | [status] | 0成功/-1失败 | LED控制 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wuqiZhu/IoTDualCtrl](https://github.com/wuqiZhu/IoTDualCtrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
