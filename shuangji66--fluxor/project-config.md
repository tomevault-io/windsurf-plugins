---
trigger: always_on
description: 本文件为后续接手的 AI 编码助手或开发者提供项目的系统架构、核心逻辑、通信接口及开发规约，以便于快速理解项目并进行无缝维护与扩展。
---

# Fluxor 项目 AI 代理指南 (AGENTS.md)

本文件为后续接手的 AI 编码助手或开发者提供项目的系统架构、核心逻辑、通信接口及开发规约，以便于快速理解项目并进行无缝维护与扩展。

---

## 1. 项目定位与架构概述

`Fluxor` 是一个轻量级、无冗余的 Mihomo 内核管理面板与订阅生成系统。它采用**前后端不分离**的架构设计：
- **后端 (Go)**：使用 Go 1.26 标准库（仅引入 `gorilla/websocket` 作为唯一外部依赖）。后端托管在 Unix Socket (`/var/apps/Fluxor/target/app.sock`) 上，对外通过前端反向代理暴露，内嵌了前端的所有静态资源。
- **前端双版本并存与条件编译**：
  为了维护旧版的前端原生加载独立性，同时满足现代客户端的体验，项目支持**双版本前端分流编译**：
  1. **Vanilla JS 版（旧版，默认）**：无需任何构建步骤，直接嵌入 `static/` 目录的静态原生 HTML/JS 源码。
  2. **Vue 3 TypeScript 版（新版，主维护）**：源码位于 `web/` 目录，由 Vue 3 (Composition API / Setup) + Vite + TailwindCSS + Pinia + TypeScript 构成。
  - **分流机制**：后端利用 Go 条件编译标签进行控制：
    - `assets_vanilla.go` (go:build !vue)：默认构建 Vanilla JS 前端。
    - `assets_vue.go` (go:build vue)：使用 `-tags vue` 参数时，自动嵌入 `web/dist` 中的 Vue 3 前端。
  - **开发环境编译**：任何新功能或漏洞修复请**首选在 Vue 3 版本中维护**，修改 Vue 代码后，需在 `web` 目录下执行 `npm run build`。

### 核心功能职责

1. **内核进程生命周期管理**：负责本地 Mihomo 二进制文件的启动、停止、状态查询及配置热重载（不中断长连接）。
2. **配置文件订阅与生成**：读取用户的订阅链接及自定义规则集，生成内核可运行的 `config.yaml`。
3. **通信中转代理 (Bridge)**：由于 Mihomo 运行在本地 Unix Socket 上，Fluxor 后端作为前端与本地内核之间的“双向桥梁”，代理所有的 HTTP API 请求与 WebSocket 数据流（流量、内存、连接、日志等），并自动附加 `Bearer Token` 认证。

---

## 2. 目录结构与索引

```text
fluxor/
├── main.go                 # 程序入口，路由注册，Unix Socket 监听，WebSocket 双向代理 (wsProxyHandler)
├── assets_vanilla.go       # 嵌入旧版静态资源 (go:build !vue)
├── assets_vue.go           # 嵌入 Vue 版编译产物 (go:build vue)
├── handlers_core.go        # 内核进程生命周期（start/stop/restart/coreRequest/cancelableReadCloser）
├── handlers_api.go         # 代理内核 HTTP API（流量、内存、连接、代理、规则、配置、DNS、GEO、升级等）
├── handlers_tproxy.go      # TProxy 防火墙与路由规则管理，例外 IP/端口过滤，本地出站代理控制
├── handlers_index.go       # 主页入口 index.html 模板渲染
├── handlers_utils.go       # JSON 错误响应工具 (writeJSONError/respondJSON) 及后端地址正则校验
├── subscribe.go            # 订阅配置 CRUD、config.yaml 生成、模板替换、MetaCubeXD config.js 修改
├── build/                  # 跨平台自动化编译与打包工具链（含 config_lite/base/full.yaml 模板）
├── static/                 # 旧版原生 Vanilla JS 前端目录
└── web/                    # 主维护 Vue 3 前端源码目录
    ├── package.json        # Vue 3.4 + Pinia + vue-i18n 9 + Vite 5 + Tailwind CSS 3 + TypeScript 5 + @vicons/ionicons5
    ├── vite.config.js      # 自定义 fluxorBuildPlugin：构建后将 index.html 移至 static/html/ 适配 embed.FS
    ├── tailwind.config.js  # data-theme 暗黑模式 + 扩展 accent/success/danger/warning 颜色
    ├── postcss.config.js   # Tailwind + Autoprefixer
    ├── index.html          # HTML 入口
    └── src/
        ├── main.ts         # 挂载 Pinia + vue-i18n (Composition API, legacy:false)
        ├── App.vue         # 根组件：响应式侧边栏/移动端底部 Tab、亮暗/跟随系统主题、中英切换、Toast 队列、Promise 确认框、统一轮询 coreStatus 状态
        ├── env.d.ts        # .vue 类型声明 & Window.BASE_URL 接口扩展
        ├── i18n.ts         # 全站国际化（zh/en），从 localStorage 读取语言偏好，禁止硬编码中文
        ├── index.css       # Tailwind 基础指令 + CSS 变量亮暗主题（data-theme 选择器）+ 自定义滚动条
        ├── components/     # 公共及细粒度组件 (ProxyGroupCard, FormSwitch)
        ├── composables/    # 全局解耦组合式函数 (useTheme, useLanguage)
        ├── utils/
        │   ├── api.ts      # withBase() 拼接 BASE_URL、apiFetch() HTTP 封装、wsConnect() WebSocket 封装（自动 ws/wss 协议选择）
        │   └── mock.ts     # 前端离线开发模拟器：拦截 HTTP/WS 请求提供 mock 数据，支持脱离后端独立测试
        ├── store/
        │   ├── global.ts   # 标签页激活状态、侧边栏折叠、亮暗/跟随系统主题、Toast 队列（3s 自动消失）、Promise 驱动确认框
        │   ├── config.ts   # 内核常规配置参数（allow-lan/ipv6/mode/log-level/tun/端口等，通过 app.vue 统一轮询 coreStatus，与订阅解耦）
        │   ├── subscription.ts # 订阅管理 Pinia Store：负责订阅配置 CRUD、解析及状态更新，由 config.ts 中拆分解耦而来
        │   ├── overview.ts # 仪表盘实时统计（速度/流量/内存/连接数/版本/当前节点）、60 点流量历史、3 路 WS + 1 路 HTTP 轮询
        │   ├── proxies.ts  # 代理组列表、节点延迟字典、手风琴展开状态、并发受限（10）批量测速
        │   ├── connections.ts # 活跃/已关闭连接列表、汇总统计、排序/搜索、WS 瞬时速率计算（快照差分）
        │   ├── rules.ts    # 规则列表、规则提供商列表、fetch/refresh 方法
        │   └── logs.ts     # 日志缓冲区（上限 2000 条）、自动滚动、暂停/继续、指数退避重连（1s~30s）
        └── views/
            ├── Overview.vue     # 概览：4 指标卡（上传/下载速度+总量）+ 4 信息卡（内存/连接数/版本/外部面板）+ Canvas 自绘折线图（max 60 点）
            ├── Proxies.vue      # 代理：手风琴展开/折叠、点击切换选择、单节点/组/全部测速、延迟着色（绿≤150 / 黄≤300 / 红>300ms/超时）
            ├── Rules.vue        # 规则：搜索过滤、启用/禁用开关（乐观更新+回滚）、规则提供商单个/全部更新
            ├── Connections.vue  # 连接：活跃/已关闭双标签、多列排序、搜索过滤、单条/全部断开（乐观更新移入 closed）、清空已关闭
            ├── Logs.vue         # 日志：暗色终端风格、级别过滤（Debug/Info/Warning/Error）、搜索、暂停/继续、智能自动滚动
            ├── Config.vue       # 配置：内核状态卡（启动/停止/升级）、常规参数、端口校验（1025-65535+重复检测）、TUN（gVisor/System/Mixed）、高级运维（重载/清缓存/GEO）、内置 DNS 查询
            └── Subscription.vue # 订阅：代理/面板端口、密钥显隐切换、规则集（lite/base/full）、UI 面板选择、订阅 CRUD 模态框（zoomIn 动画，支持订阅名称、链接、检测间隔、节点前缀）、流量/健康度/有效期卡片、「保存并应用」
```

> **页面路由机制**：未使用 vue-router，通过 `globalStore.activeTab` 与 `<component :is="..." />` 动态组件切换视图。在此基础上，外层包裹了 `<KeepAlive :max="6">` 进行视图缓存，以长效留存页面各交互状态（如滚动进度与折叠状态）并规避切换页面时的重复连接请求。

---

## 3. 前后端通信与代理机制 (重要规避点)

### 3.1 统一路由前缀 (BASE_URL)
所有的请求均有统一的基本路径前缀：`baseURL = "/app/Fluxor"`。
在 Vue 源码中，所有 `apiFetch` 或 WebSocket 通信必须调用 [api.ts](web/src/utils/api.ts)，它会自动且妥善地完成前缀拼接。

### 3.2 HTTP 代理流过早截断修复与 Context 释放
前端向后端发起管理请求时，后端通过 Unix Socket 拨号并发 Do(req) 请求内核。为了防止大 JSON 数据（例如代理组数据、连接历史）在传输中因超时 Context 被提前取消导致流被中断（抛出 `Unterminated string in JSON` 错误），后端在 [handlers_core.go](handlers_core.go) 实现了：
```go
type cancelableReadCloser struct {
	io.ReadCloser
	cancel context.CancelFunc
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuangji66/fluxor](https://github.com/shuangji66/fluxor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
