---
trigger: always_on
description: 跨平台、面向非技术用户的 [frp](https://github.com/fatedier/frp) 桌面客户端，
---

# MoonProxy

跨平台、面向非技术用户的 [frp](https://github.com/fatedier/frp) 桌面客户端，
基于 Tauri v2 构建，支持 Windows 与 macOS。

> 用户视角的介绍请看 [README.md](./README.md)；本文档面向在仓库里改代码的工程师。

## 功能

- 用户填入自己的 frps 服务端（地址、端口、Token、用户名），或预置自定义服务商快速切换
- 可视化管理代理规则（TCP / UDP / HTTP / HTTPS），主页实时显示本地端口连通性
- 实时流量监控：主页上下行速率曲线、连接数与累计流量（内置 TCP 中转层计数）
- 一键启动 / 停止 frpc：启动按钮分 4 态（已停止 / 连接中 / 已连接 / 连接错误），
  连接状态由 frpc 自身证据支撑（`/api/status` 探测 + 30s 超时回退）
- 系统托盘常驻：关闭窗口默认隐藏到托盘，frpc 继续后台运行
- 开机启动 + 静默启动（自启时隐藏到托盘）
- 开机自动连接（仅 OS 自启路径触发，手动启动不会误连）
- 定时连接（按星期多选 + 起止时间，分钟对齐 + 启动补跑 + 热加载）
- frpc 引擎自更新：GitHub Release → SHA256 校验 → 原子替换，无需重装应用
- 应用本体自更新：基于 `tauri-plugin-updater` 的「重启并安装」
- 通过 Tauri sidecar 机制内置 frpc 二进制，用户无需单独安装

## 技术栈

- 前端：Vue 3 + TypeScript + Vite 6
- 后端：Rust + Tauri v2
- frpc 版本：v0.69.1（已内置；可通过 `pnpm sync:frpc` 同步其他平台）

## 目录结构

```
moonproxy-desktop/
├── src/                          # Vue 前端
│   ├── App.vue                   # 顶层壳：TitleBar + 视图路由 + 全局键盘 / 右键 + 事件订阅
│   ├── main.ts                   # 挂载入口（按 ?view= 区分主窗 / 日志窗）
│   ├── state/                    # 跨组件响应式状态（拆分到各文件）
│   │   ├── index.ts              # 统一导出
│   │   ├── runtime.ts            # 运行时：frpc 状态 / 日志 / 启动时间
│   │   ├── config.ts             # 服务商 / 代理配置
│   │   ├── prefs.ts              # 偏好：语言 / 启动项 / 定时
│   ├── styles.css                # HSL 设计令牌 + 通用组件类
│   ├── composables/              # 按主题拆分的可复用状态 / 命令
│   │   ├── useToast.ts
│   │   ├── useFrpcUpdate.ts      # frpc 引擎自更新
│   │   ├── useAppUpdate.ts       # 应用本体自更新
│   │   ├── useProxyHealth.ts     # 主页端点健康点 + 指数退避轮询（3→6→12→24s）
│   │   ├── useTraffic.ts         # 实时流量：累计 / 滚动窗口 / 瞬时速率
│   │   └── useAppEvents.ts       # 应用级事件订阅 + 启动初始化
│   ├── components/
│   │   ├── TitleBar.vue          # 跨平台标题栏
│   │   ├── BrandIcon.vue         # 单色品牌标识
│   │   ├── CloseConfirm.vue      # frpc 运行时的关闭确认弹窗
│   │   ├── Toast.vue             # 顶部 Toast 渲染
│   │   ├── home/                 # HomeView 子组件（启动按钮 / 流量图表 / 端点列表 …）
│   │   └── settings/             # 设置面板 Tab 子组件
│   │       ├── ProviderTab.vue   # 服务商
│   │       ├── ProxyTab.vue      # 代理规则
│   │       ├── InterfaceTab.vue  # 界面语言切换
│   │       ├── LaunchTab.vue     # 开机启动 / 静默启动 / 开机自动连接
│   │       ├── ScheduleSection.vue  # 定时连接（LaunchTab 内嵌）
│   │       ├── LogsTab.vue       # 运行日志
│   │       └── AboutTab.vue      # 关于（含软件更新 + 核心引擎）
│   └── views/
│       ├── HomeView.vue          # 主面板：流量图表 / 启动按钮 / 端点列表 / 引导卡片
│       ├── ServicesView.vue      # 「服务」视图：复用 ProviderTab + ProxyTab
│       └── SettingsView.vue      # 设置面板：分段控件 + Tab 切换
├── src-tauri/
│   ├── src/
│   │   ├── main.rs               # Windows release 抑制控制台
│   │   ├── lib.rs                # Tauri Builder 配置 + setup hook + invoke_handler + ExitRequested 兜底
│   │   ├── types.rs              # 共享类型：StartArgs / ProxyConfig
│   │   ├── config.rs             # frpc.toml 生成 + 客户端配置持久化
│   │   ├── process.rs            # frpc 子进程生命周期
│   │   ├── frpc_state.rs         # 连接状态机 + 日志环形缓冲
│   │   ├── proxy_health.rs       # 代理本地端口连通性探测
│   │   ├── proxy_relay.rs        # frpc↔本地服务 TCP 中转层 + 流量统计
│   │   ├── latency.rs            # 服务端 TCP 握手延迟探测
│   │   ├── frpc_update.rs        # frpc 自更新
│   │   ├── prefs.rs              # 应用偏好
│   │   ├── scheduler.rs          # 按星期定时启停 frpc
│   │   ├── tray.rs               # 系统托盘
│   │   └── assets/               # 编译期嵌入资源（托盘图标）
│   ├── binaries/                 # frpc sidecar 二进制（按平台目标命名；不入库，本地放置）
│   ├── capabilities/
│   │   ├── default.json          # 主窗权限
│   │   └── logs.json             # 独立日志窗权限
│   ├── tauri.conf.json           # 窗口 400×740、无装饰、sidecar 声明、updater 端点
│   └── tauri.macos.conf.json     # macOS 平台覆盖：系统交通灯 + Overlay + hiddenTitle
├── docs/app-icons/               # 图标设计源（APP Icon + 托盘图标），含 README 规范
└── scripts/                      # 仓库辅助脚本
    ├── sync-frpc.sh              # 按 .env 版本同步 frpc 二进制到 src-tauri/binaries/
    └── check-icons.py            # 图标规范校验（更新图标后必跑）
```

> 前端 / 后端开发约定详见各自目录下的 `AGENTS.md`（`src/AGENTS.md`、`src-tauri/AGENTS.md`）。

## 配置与数据存储

按语义拆成三套独立存储，全部落到 Tauri 标准 `app_config_dir()`
（macOS：`~/Library/Application Support/<bundle-identifier>/`；
Windows：`%APPDATA%\<bundle-identifier>\`），互不污染。

| 数据           | 存储后端                  | 文件                    | 键 / 格式       | 管理模块                                  | 写入时机                                                                 |
| -------------- | ------------------------- | ----------------------- | --------------- | ----------------------------------------- | ------------------------------------------------------------------------ |
| 应用偏好       | `tauri-plugin-store`      | `prefs.json`            | `auto_launch` / `silent_start` / `auto_connect`（bool）+ `schedule`（`Schedule` 对象：`enabled` / `weekdays[7]` / `start_time` / `stop_time`，按星期定时启停 frpc）+ `language`（字符串） | `src-tauri/src/prefs.rs`                  | `LaunchTab` / `InterfaceTab` 保存；`set_auto_launch` 先写 OS 启动项再以实际状态回填；`auto_connect` 仅在 `--auto-launched`（OS 自启）时触发 `start_frpc`；`schedule` 由 `scheduler.rs` 每分钟重读热加载 |
| 客户端配置     | `tauri-plugin-store`      | `config.store.json`     | 单键 `start_args`（整体序列化 `StartArgs`） | `src-tauri/src/config.rs` `save_config`/`load_config` | 设置面板「服务商」「代理」Tab 保存                                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonProxyHQ/moonproxy-desktop](https://github.com/MoonProxyHQ/moonproxy-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
