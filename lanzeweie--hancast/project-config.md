---
trigger: always_on
description: **HanCast** 是基于 [xfangfang/Macast](https://github.com/xfangfang/Macast) 的二次开发项目，跨平台投屏应用，支持将媒体文件/链接投屏到局域网设备，同时可作为 DLNA 接收端。
---

# CLAUDE.md — HanCast 项目指南

## 项目概述

**HanCast** 是基于 [xfangfang/Macast](https://github.com/xfangfang/Macast) 的二次开发项目，跨平台投屏应用，支持将媒体文件/链接投屏到局域网设备，同时可作为 DLNA 接收端。

- **项目名**: HanCast
- **作者**: lanzeweie@foxmail.com
- **版本**: 2.0.1
- **性质**: 二次开发（非原项目官方更新）
- **原始代码**: `Macast-main/`（原作者 xfangfang，最后更新 2022-01）
- **目标架构**: Tauri 2.0 前端 + Rust 桥接 + Python Sidecar
- **协议**: GPL-3.0（继承原项目）

## 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 前端框架 | Tauri 2.0 | ^2.0.0 |
| 前端 UI | Vue 3 + TypeScript + Vite | Vue ^3.4.0 |
| 状态管理 | Pinia | ^2.1.7 |
| 路由 | vue-router | ^4.2.5 |
| 国际化 | vue-i18n | ^9.9.0 |
| 桥接层 | Rust (Tauri Core) | — |
| 后端服务 | Python Sidecar | >= 3.11 |
| 协议支持 | DLNA/UPnP/SSDP | — |
| 媒体播放 | MPV（外部依赖） | — |

## 项目结构

```
G:\Code\Macast-Han\
├── CLAUDE.md                    # 本文件 — 项目核心约束
├── README.md
├── package.json                 # 前端依赖
├── vite.config.ts               # Vite 配置
├── tsconfig.json                # TypeScript 配置
├── index.html                   # Vite 入口 HTML
├── env.d.ts                     # Vue 类型声明
│
├── src/                         # Vue 3 前端源码
│   ├── main.ts                  # 应用入口（Pinia/Router/i18n 初始化）
│   ├── App.vue                  # 根组件（主题检测、设置加载）
│   ├── api/
│   │   └── commands.ts          # Tauri invoke 封装 + 浏览器 mock
│   ├── components/
│   │   ├── TitleBar.vue         # 自定义标题栏（最小化、关闭、设置）
│   │   ├── MediaInput.vue       # 拖拽 + URL 输入 + 剪贴板粘贴
│   │   ├── CastControl.vue      # 投屏状态栏（播放/暂停/停止指示）
│   │   ├── DeviceList.vue       # 设备列表（含刷新）
│   │   ├── DeviceCard.vue       # 单个设备卡片（投屏、重命名、移除、设默认）
│   │   └── Footer.vue           # "无设备" 提示底栏
│   ├── views/
│   │   ├── HomeView.vue         # 主视图
│   │   └── SettingsView.vue     # 设置页（语言、DLNA 名称、端口、关于）
│   ├── stores/                  # Pinia 状态管理
│   │   ├── cast.ts              # 投屏状态
│   │   ├── device.ts            # 设备列表（30s 自动刷新）
│   │   ├── media.ts             # 媒体输入状态机
│   │   └── settings.ts          # 应用设置
│   ├── types/                   # TypeScript 类型定义
│   │   ├── cast.ts
│   │   ├── device.ts
│   │   ├── media.ts
│   │   ├── settings.ts
│   │   └── index.ts
│   ├── locales/                 # 国际化资源
│   │   ├── zh-CN.json
│   │   └── en-US.json
│   ├── router/
│   │   └── index.ts             # Hash 路由：/ (Home), /settings (Settings)
│   └── styles/
│       ├── variables.css        # CSS 变量（亮色/暗色主题）
│       └── global.css           # 全局样式
│
├── src-tauri/                   # Tauri Rust 后端
│   ├── Cargo.toml               # Rust 依赖
│   ├── tauri.conf.json          # Tauri 配置
│   ├── build.rs                 # tauri_build::build()
│   ├── icons/                   # 应用图标
│   └── src/
│       ├── main.rs              # 入口，调用 hancast_lib::run()
│       ├── lib.rs               # Tauri 应用设置 + 20 个命令定义
│       └── sidecar.rs           # Python Sidecar 管理器（stdin/stdout JSON）
│
├── hancast-backend/              # Python Sidecar 后端
│   ├── pyproject.toml           # Python 项目配置
│   ├── requirements.txt
│   ├── uv.lock
│   ├── hancast_sidecar/
│   │   ├── main.py              # Sidecar 入口（stdin/stdout JSON 循环）
│   │   ├── commands.py          # CommandHandler — 命令路由
│   │   ├── ssdp.py              # SSDP 设备发现（~540 行）
│   │   ├── protocol/
│   │   │   ├── dlna.py          # DLNA 协议实现（~840 行）
│   │   │   └── server.py        # DLNA HTTP 服务（描述、SOAP、SUBSCRIBE）
│   │   ├── renderer/
│   │   │   ├── base.py          # 渲染器基类
│   │   │   └── mpv.py           # MPV 渲染器（IPC: named pipe/unix socket）
│   │   ├── media/
│   │   │   ├── parser.py        # 媒体文件/URL 解析器
│   │   │   └── server.py        # 本地文件 HTTP 服务（支持 Range）
│   │   ├── types/
│   │   │   ├── cast.py          # CastState dataclass
│   │   │   ├── device.py        # Device dataclass
│   │   │   └── media.py         # MediaInfo dataclass
│   │   ├── utils/
│   │   │   ├── config.py        # 配置管理（AppData JSON 文件）
│   │   │   └── logger.py        # 日志设置
│   │   └── xml/                 # UPnP 描述 XML
│   │       ├── Description.xml
│   │       ├── AVTransport.xml
│   │       ├── ConnectionManager.xml
│   │       ├── RenderingControl.xml
│   │       ├── SinkProtocolInfo.csv
│   │       └── setting.html
│   ├── scripts/
│   │   ├── build_sidecar.py     # PyInstaller 构建脚本
│   │   └── run_sidecar.py       # 独立 Sidecar 运行器
│   └── tests/
│       ├── test_commands.py
│       └── test_imports.py
│
├── Macast-main/                 # 原始 1.x Python 源码（参考用）
├── Macast-plugins-main/         # 原始 Macast 插件（参考用）
├── mpv/                         # 捆绑的 MPV 二进制文件（Windows, ~120MB）
├── docs/                        # 设计文档
│   ├── HanCast-Backend-Spec.md
│   ├── Macast-Frontend-API.md
│   ├── Macast-Frontend-Spec.md
│   └── 原型图.png
├── _bmad/                       # BMad 方法论配置
├── _bmad-output/                # BMad 规划/实现产物
└── .claude/                     # Claude Code 配置
    └── memory/                  # 项目记忆与进度追踪
```

## 核心架构

### 进程通信架构

```
┌─────────────────┐     Tauri invoke()     ┌─────────────────┐
│  Vue 3 Frontend  │ ←────────────────────→ │  Rust Backend    │
│  (WebView)       │                        │  (Tauri Core)    │
└─────────────────┘                        └────────┬────────┘
                                                    │ stdin/stdout JSON
                                                    ▼

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lanzeweie/HanCast](https://github.com/lanzeweie/HanCast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
