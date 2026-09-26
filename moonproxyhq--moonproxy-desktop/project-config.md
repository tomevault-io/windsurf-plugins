---
trigger: always_on
description: > 本文件面向在该目录（`src/`）下做前端开发的工程师。涉及前后端协议的部分
---

# 前端开发指南（MoonProxy）

> 本文件面向在该目录（`src/`）下做前端开发的工程师。涉及前后端协议的部分
> 也给出对应后端命令 / 事件的引用，便于一眼对齐。

## 1. 技术栈

- **框架**：Vue 3（`<script setup lang="ts">`）
- **构建**：Vite 6
- **类型**：TypeScript 5.6（`strict`、`noUnusedLocals`、`noUnusedParameters`）
- **桥接**：`@tauri-apps/api`（`core` / `event` / `window`）+ 官方插件
  - `@tauri-apps/plugin-dialog`：原生对话框
  - `@tauri-apps/plugin-fs`：文本 / 目录读写
  - `@tauri-apps/plugin-opener`：外部链接
  - `@tauri-apps/plugin-shell`：副作用留作扩展位（侧车执行由后端封装）
- **状态**：原生 `ref` / `reactive`，**不引入** Pinia / Vuex
- **样式**：手写 CSS（无 Tailwind / UnoCSS），遵循 `styles.css` 中的 HSL 令牌
- **图表**：`chart.js` + `vue-chartjs`（仅主页实时流量曲线，按需 register 模块）

## 2. 目录结构与文件职责

```
src/
├── main.ts                       # 挂载入口；按 URL `?view=logs` 分流到 LogsWindow，否则挂 App
├── App.vue                       # 主窗顶层壳：TitleBar + 视图路由 + 全局键盘/右键监听；事件订阅已抽到 useAppEvents
├── types.ts                      # 前后端共享类型（ProxyConfig / FrpcConfig / Prefs / FrpcStatus / LogEntry / TrafficPayload）
├── i18n.ts                       # vue-i18n 实例 + AppLocale 类型 + setLocale / normalizeLocale
├── state/                        # 核心响应式状态，按主题拆分（详见 §3.2）
│   ├── index.ts                  # 聚合 barrel：仅做 `export * from "./xxx"`，不放任何状态
│   ├── config.ts                 # config + isConfigured + toArgs
│   ├── prefs.ts                  # prefs（应用偏好）
│   ├── runtime.ts                # frpcStatus / frpcError / running / logs
├── commands/                     # 按职责拆分的 invoke 封装（全部吞异常、不向前端抛）
│   ├── config.ts                 # loadConfig / saveConfig
│   ├── contextMenu.ts            # showEditMenu（输入框原生右键编辑菜单）
│   ├── frpc.ts                   # startFrpc / stopFrpc
│   ├── latency.ts                # probeServerLatency（服务端 TCP 握手延迟探测）
│   └── prefs.ts                  # loadPrefs / savePrefs / setAutoLaunch / refreshAutoLaunch
├── styles.css                    # 设计令牌（HSL）+ 通用组件类（.btn / .input / .card / .badge）
├── vite-env.d.ts                 # *.vue 模块声明
├── composables/
│   ├── useToast.ts               # 轻量 Toast（showToast / dismiss timer）
│   ├── useFrpcUpdate.ts          # frpc 自更新：版本 / updateInfo / 下载 / 横幅相关
│   ├── useAppUpdate.ts           # 应用本体自更新
│   ├── useProxyHealth.ts         # 主页端点健康点：proxyHealth + 指数退避轮询（3→6→12→24s）
│   ├── useAppEvents.ts           # 应用级事件订阅 + 启动初始化（App.vue 已委托）
│   ├── useLogsWindow.ts          # 打开/聚焦独立日志窗口（WebviewWindow label="logs"）
│   └── useTraffic.ts             # 实时流量：累计字节 / 60s 滚动窗口 / 瞬时速率 + 格式化
├── components/
│   ├── TitleBar.vue              # 跨平台标题栏：mac 交通灯避让、Win 最小化/关闭、拖动区；「服务」「设置」（仅 home）与「返回」（仅非 home）均按 OS 分槽（macOS 右 / Windows 左，远离系统窗口控件）
│   ├── BrandIcon.vue             # 单色品牌标识（currentColor SVG），跟随标题文字色
│   ├── CloseConfirm.vue          # frpc 运行时的关闭确认弹窗（最小化 / 退出）
│   ├── Toast.vue                 # 顶部 Toast 渲染
│   ├── home/                     # HomeView 拆出的子组件（详见 §5.4）
│   │   ├── StartButton.vue       # 底部药丸形启动按钮 + CSS 双层涟漪 + 4 态文案
│   │   ├── TrafficChart.vue      # 实时流量曲线（chart.js）：连接数 / 上下行速率 / 累计
│   │   ├── ProxyList.vue         # 公网访问地址列表 + 健康点 + 复制按钮 + 指数退避健康轮询
│   │   ├── GuideCard.vue         # 未配置引导卡片
│   │   └── SystemStatus.vue      # 底部只读系统状态栏（开机启动 / 定时连接）
│   ├── banners/
│   │   └── UpdateBanners.vue     # 顶部 4 类横幅：frpc 错误条 / 软件本体更新 / 引擎已应用 / 引擎待应用
│   └── settings/                 # 设置面板 Tab 子组件
│       ├── ProviderTab.vue
│       ├── ProxyTab.vue
│       ├── InterfaceTab.vue      # 界面语言切换
│       ├── LaunchTab.vue         # 开机启动 / 静默启动 / 开机自动连接（ScheduleSection 抽出独立子件）
│       ├── ScheduleSection.vue   # 定时连接：主开关 + 星期选择 + 起止时间 + 校验 + 保存
│       ├── LogsTab.vue           # 运行日志
│       └── AboutTab.vue          # 关于（含软件更新 + 核心引擎）
└── views/
    ├── HomeView.vue              # 主面板：纯组装（TrafficChart + GuideCard + ProxyList + StartButton + SystemStatus + 错误条 + 启停逻辑）
    ├── ServicesView.vue          # 「服务」视图：复用 ProviderTab + ProxyTab 的分段控件
    ├── SettingsView.vue          # 设置面板：分段控件 + Tab 切换
    └── LogsWindow.vue            # 独立日志窗口根组件：get_logs 拉历史 + listen 实时；不复用 App.vue 的关闭/快捷键逻辑
```

## 3. 状态层

> 设计原则：**单例、扁平、纯响应式**。所有跨视图共享状态按主题拆分到独立模块，
> 视图组件只读 + 通过封装的命令函数修改。
>
> - `types.ts`：前后端共享类型（snake_case，与 Rust 一一对应）
> - `state/` 子目录：核心响应式状态，按主题拆为 `config` / `prefs` / `runtime`
>   三个模块；统一经 `state/index.ts` barrel 暴露；
>   `isConfigured` / `toArgs` 留在 `state/config.ts`（仅服务 config）
> - `commands/config.ts` / `commands/frpc.ts` / `commands/prefs.ts`：按职责拆分的 invoke 封装
> - `composables/useFrpcUpdate.ts`：frpc 引擎自更新相关状态
> - `composables/useAppUpdate.ts`：应用本体自更新相关状态
> - `composables/useProxyHealth.ts`：代理本地端口连通性
> - `composables/useTraffic.ts`：实时流量（累计 / 滚动窗口 / 瞬时速率）
> - `composables/useAppEvents.ts`：应用级 Tauri 事件订阅 + 启动初始化（App.vue 已委托）

### 3.1 类型

`types.ts`：

```ts
// ProxyConfig 是按 `type` 拆分的 discriminated union——每种 frp 代理类型
// 有独立的 schema（TCP/UDP 走 remotePort，HTTP/HTTPS 走 customDomain 且
// 不接受 remotePort）。聚合在扁平结构里会让 build_toml / URL 生成路径
// 都需按字符串 type 分叉，且无法在编译期排除非法字段。
type ProxyConfig =
  | { type: "tcp" | "udp"; name: string; local_ip: string;
      local_port: number; remote_port: number }
  | { type: "http" | "https"; name: string; local_ip: string;
      local_port: number; custom_domain: string };
interface FrpcConfig {
  custom_name: string;   // 自定义服务商显示名称
  server_addr: string; server_port: number;
  token: string; user: string;
  proxies: ProxyConfig[];
}
interface Prefs {
  auto_launch: boolean;  // 开机启动（OS 实际状态）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonProxyHQ/moonproxy-desktop](https://github.com/MoonProxyHQ/moonproxy-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
