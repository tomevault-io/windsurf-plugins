---
trigger: always_on
description: QtEasyTier 是一个基于 Qt C++ 框架开发的异地组网工具，提供直观的图形界面帮助用户配置和管理虚拟网络。其后端核心为 EasyTier —— 一个去中心化的组网方案，无需依赖中心服务器，每个节点平等独立，为用户提供安全、可靠、低成本的异地组网服务。
---

# QtEasyTier 项目上下文

## 项目概述

QtEasyTier 是一个基于 Qt C++ 框架开发的异地组网工具，提供直观的图形界面帮助用户配置和管理虚拟网络。其后端核心为 EasyTier —— 一个去中心化的组网方案，无需依赖中心服务器，每个节点平等独立，为用户提供安全、可靠、低成本的异地组网服务。

### 项目特点
- **快速**: 纯 Qt C++ 开发，无 Chromium/Webview，前端占用不超过 50MB
- **美观**: UI 样式移植自 KDE 的 Breeze 样式
- **简单**: 直观的网络配置管理界面
- **丰富**: 支持 EasyTier 大部分功能，按需定制虚拟网络
- **安全**: 基于 EasyTier 去中心化组网方案
- **高效**: 使用 FFI 直接调用核心库，无需子进程管理

### 技术栈
- **语言**: C++ 20
- **框架**: Qt 6（推荐 6.10.1），支持 Qt5 向后兼容
- **构建系统**: CMake 3.16+
- **UI 样式**: KDE Breeze Style（移植版）
- **核心绑定**: EasyTier FFI（Rust 库的 C 绑定）

### 当前版本
- 项目版本: 2.0.0（Beta）
- EasyTier FFI 版本: 2.6.0
- 平台支持: Windows 10/11、Linux

---

## 目录结构

```
QtEasyTier/
├── Qt_Gui/               # 主界面源码
│   ├── qtetmain.h/cpp    # 主窗口类
│   ├── qtetnetwork.h/cpp # 网络配置页面
│   ├── qtetoneclick.h/cpp    # 一键联机页面
│   ├── qtetservers.h/cpp # 服务器收藏页面
│   └── qtetsettings.h/cpp    # 设置页面
├── Qt_Items/             # 自定义控件
│   ├── qtetcheckbtn.h/cpp    # 自定义开关按钮
│   ├── qtetlabellist.h/cpp   # 标签列表控件
│   ├── qtetnodeinfo.h/cpp    # 节点信息显示控件
│   ├── qtetpushbtn.h/cpp # 自定义推送按钮
│   ├── qtettabwidget.h/cpp   # 自定义 Tab 控件
│   └── qtetpublicserverdialog.h/cpp  # 公共服务器对话框
├── Qt_QRC/               # 资源文件（图标等）
│   ├── icons.qrc         # 资源文件定义
│   ├── icon.ico          # 应用程序图标
│   ├── *.svg             # SVG 图标资源
│   └── *.ttf             # 自定义字体文件
├── SRC/                  # 核心源码
│   ├── main.cpp          # 程序入口、单实例检测、Breeze样式加载
│   ├── ETRunWorker.h/cpp # EasyTier FFI 运行工作类
│   └── networkconf.h/cpp # 网络配置数据结构
├── ThirdParty/           # 第三方依赖
│   ├── Breeze/           # Breeze 主题资源
│   │   ├── data/         # 配色方案、本地化
│   │   └── styles/       # 样式库
│   ├── EasyTier/         # EasyTier 核心库
│   │   ├── include/      # FFI 头文件 (easytier_ffi.h)
│   │   ├── library/      # Windows 静态库
│   │   ├── bin/          # 可执行文件
│   │   └── share/        # 平台特定文件
│   │       ├── linux/    # Linux 可执行文件
│   │       └── windows/  # Windows 可执行文件
│   └── toml++/           # TOML 解析库
├── CDIRCalculator/       # CIDR 计算器子项目
├── assets/               # 项目资源（图片、manifest等）
│   ├── app.manifest      # Windows 应用程序清单
│   ├── publicserver.json # 公共服务器列表
│   └── *.png/*.webp      # 图片资源
├── cmake-build-debug/    # Debug 构建目录
└── cmake-build-release/  # Release 构建目录
```

---

## 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                        QtETMain                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              m_mainStackedWidget                     │    │
│  │  ┌──────────┬──────────┬──────────┬──────────┐     │    │
│  │  │ HelloPage│NetworkPage│OneClick │ Servers  │     │    │
│  │  │          │           │  Page   │  Page    │     │    │
│  │  └──────────┴──────────┴──────────┴──────────┘     │    │
│  │                      Settings Page                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                           │                                  │
│                    SystemTrayIcon                            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      ETRunWorker                             │
│  (运行在独立 QThread 中)                                      │
│         │                                                    │
│         ▼                                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   EasyTierFFI                        │    │
│  │  parseConfig() | runNetworkInstance()                │    │
│  │  retainNetworkInstance() | collectNetworkInfos()     │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心类说明

### QtETMain
主窗口类，负责：
- 欢迎页面和功能导航
- 系统托盘管理
- 界面切换（欢迎页、网络配置页、一键联机页、服务器收藏页、设置页）
- 单实例运行检测
- 网络启动/停止通知处理

**关键成员：**
- `m_mainStackedWidget`: 主堆叠窗口
- `m_helloPage`: 欢迎页面
- `m_networkPage`: 网络配置页面
- `m_oneClickPage`: 一键联机页面
- `m_serversPage`: 服务器收藏页面
- `m_settingsPage`: 设置页面
- `m_trayIcon`: 系统托盘图标
- `m_trayMenu`: 托盘右键菜单

**信号槽：**
- `onNetworkStartedNotify()`: 网络启动完成通知
- `onNetworkStoppedNotify()`: 网络停止完成通知
- `onTrayIconActivated()`: 托盘图标激活处理

### QtETNetwork
网络配置页面，核心功能包括：
- **网络列表管理**: 新建、删除、切换网络配置
- **配置导入导出**: 支持 JSON 格式的配置文件
- **基础设置**: 用户名、网络名称、密码、IP 地址、DHCP
- **服务器管理**: 添加/删除服务器节点
- **高级设置**: KCP/QUIC 代理、TUN 模式、P2P、IPv6、加密等
- **运行管理**: 启动/停止网络实例
- **节点监测**: 实时显示网络中的节点信息

**关键成员：**
- `m_networkConfs`: 网络配置列表
- `m_networksList`: 网络列表控件
- `m_runThread`: 运行网络的工作线程
- `m_runWorker`: ETRunWorker 工作对象
- `m_monitorTimer`: 节点监测定时器
- `m_nodeInfoWidgets`: 节点信息控件列表

**信号：**
- `networkStarted(QString, bool, QString)`: 网络启动信号
- `networkStopped(QString, bool, QString)`: 网络停止信号

### QtETOneClick
一键联机页面，简化组网流程，支持房主/房客两种模式：
- **Base32 编解码**: 自定义联机码生成与解析算法
- **房主模式**: 自动生成网络ID和密码，生成联机码供房客使用
- **房客模式**: 输入联机码自动解码加入网络
- **服务器管理**: 内置默认公共服务器，支持添加自定义服务器
- **节点监测**: 定时收集网络信息，显示房主IP和联机人数
- **超时检测**: 连接超时自动停止并提示

**关键成员：**
- `m_currentRole`: 当前角色（None/Host/Guest/Stopping）
- `m_currentNetworkId`: 当前网络号
- `m_currentPassword`: 当前密码

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [myqfeng/qt-easy-tier](https://github.com/myqfeng/qt-easy-tier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
