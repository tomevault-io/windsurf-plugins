---
trigger: always_on
description: > 协作规范。所有贡献者（含 AI agent）必须遵守。
---

# AGENTS.md — App Audio Router

> 协作规范。所有贡献者（含 AI agent）必须遵守。

---

## 项目概述

Windows 平台「每应用音频路由」工具。Tauri v2 + React + TypeScript + TailwindCSS + Motion。

核心能力：枚举有音频会话的进程、枚举渲染设备、将**一个或多个（Ctrl+多选）进程**路由到一台或多台设备（多设备时第一台为主设备，其余通过进程回环复制，支持同步启动、按设备延迟补偿以对齐蓝牙、按设备音量以平衡响度）、自动记忆路由规则、两张列表随 Core Audio 变化实时更新、托盘常驻与开机自启、单实例（再次启动只唤出已有窗口）。

**硬性约束：无任何第三方 exe 依赖。** 所有音频操作由 Rust 直接调用 Windows Core Audio API。

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 桌面框架 | Tauri v2 |
| 前端 | React 19 + TypeScript (strict) |
| 样式 | TailwindCSS v3 (utility-first, CSS variables) |
| 动画 | Motion (`framer-motion`) |
| 构建 | Vite v6 |
| 包管理 | pnpm |
| Rust | edition 2021, `windows` crate (官方) |

---

## 目录结构

```
AppAudioRouter/
├── src-tauri/              # Rust 后端
│   ├── Cargo.toml
│   ├── tauri.conf.json     # Tauri 配置 + capabilities
│   └── src/
│       ├── main.rs         # 入口，注册命令 + 窗口关闭拦截（托盘常驻）+ 显示看门狗 + 退出前交还端点分配
│       ├── commands.rs     # Tauri 命令（invoke handler）
│       ├── tray.rs         # 托盘图标 + 菜单（左键开关窗口；菜单标签由前端下发以跟随语言）
│       ├── autostart.rs    # 开机自启：直接读写 HKCU\...\Run，启动时带 --hidden
│       ├── install.rs      # 启动提示：全新安装 / 装过旧版本（建窗口之前判定，install-state.json 记版本）
│       ├── single_instance.rs  # 单实例：命名互斥 + 命名事件（第二次启动唤出已有窗口后自行退出）
│       ├── audio/
│       │   ├── mod.rs
│       │   ├── devices.rs      # IMMDeviceEnumerator 设备枚举
│       │   ├── sessions.rs     # IAudioSessionEnumerator 会话枚举
│       │   ├── routing.rs      # 每应用端点：槽 25/26 写入·释放·读回（null HSTRING = 清除）+ 系统关键进程拦截 + PinnedRoutes（本进程写过的分配，停止/退出/重置时归还）
│       │   ├── duplication.rs  # WASAPI 进程回环 → 多设备复制引擎（源静默后停放；见「引擎的唤醒成本」）+ 每源电平估计与增益
│       │   ├── levels.rs       # 每源电平表：各引擎写自己的估算，对齐与诊断都从这里读（只回 1 秒内的读数）
│       │   └── notifications.rs # 变更通知线程 → audio-changed 事件（设备/会话实时刷新）
│       └── config.rs       # 配置持久化（route-memory.json: exe -> 设备列表；device-delays.json: 设备 -> 延迟补偿 ms + delay_range_ms 正负范围上限；device-volumes.json: 设备 -> 音量 %；source-volumes.json: exe -> 电平 %（可 >100）；app-settings.json: close_to_tray）
├── src/                    # React 前端
│   ├── main.tsx
│   ├── App.tsx
│   ├── components/         # UI 组件
│   │   ├── ConcentricStage.tsx   # 同心圆舞台，也是路由唯一被表达的地方：中心是被选中的那个程序（Hub），每台输出设备一枚环绕它的玻璃圆盘（Satellite），每枚参与路由的圆盘与中心之间一根辐条；内含只给副本用的 Tuner
│   │   ├── ProgramRail.tsx       # 左栏程序列表：每行一个正在出声的程序，行末那颗圆形开关「也一起改」取代了 Ctrl+点击
│   │   ├── ActionDock.tsx        # 工作区底部悬浮玻璃条：一句话 + Apply（舞台永远只是提案）；唯一的「回到系统默认」也在这里
│   │   ├── SourceLevelDial.tsx   # 中心圆盘下方的每程序电平环（0–400 %，套 ScrubReadout 的 lead 槽画环），只在多设备路由时挂载
│   │   ├── Toast.tsx             # 瞬态提示：路由后提供「撤销」（全项目唯一的撤销入口）
│   │   ├── SettingsPage.tsx      # 设置独立页面（主题/语言/路由开关/已记忆路由列表/每应用音频重置/后台开关/延迟范围·步进/一键对齐/关于）
│   │   ├── LogPanel.tsx          # 活动记录（下划线标签页的第二页）
│   │   ├── TitleBar.tsx    # 自定义标题栏（透明，极光从它底下过：品牌 + 「已路由 · 在响」计数 + 设置入口 + 窗口控制）
│   │   ├── StartupNoticeDialog.tsx  # 启动提示弹窗（欢迎语 / 旧版本提示 + 就地重置）
│   │   └── ui/             # 基础控件
│   │       ├── Ring.tsx                # 同心圆指示器：外环说"它是什么"，内芯说"它在不正在响"；全项目共用这一枚
│   │       ├── Switch.tsx              # 开关（h-5 w-9）
│   │       ├── UnderlineTabs.tsx       # 下划线标签页（不是胶囊；下划线用 layoutId 迁移）
│   │       ├── SegmentedControl.tsx    # 下划线式分段控件（不是胶囊，也不是 pill track）
│   │       ├── ScrubReadout.tsx        # 通用「数值即控件」（拖动/滚轮/方向键/键入），电平环在用；lead 槽可在数字前挂一个跟随预览值的图形
│   │       ├── StepButton.tsx          # 方形 ± 按钮（h-6 w-6，仅设置页在用）
│   │       ├── ConfirmButton.tsx       # 二次确认按钮（先「确认？」再执行；Escape/失焦/超时解除）
│   │       └── DelayStepper.tsx        # 设置页的延迟行控件（−/数值/+ 方框样式）
│   ├── hooks/              # 自定义 hooks
│   │   ├── useTheme.ts
│   │   ├── useLanguage.ts
│   │   ├── useDelayValue.ts  # 延迟编辑状态（草稿/提交/步进），两个延迟控件共用
│   │   ├── useLiveness.ts    # 声流闸门：窗口可见 + 持有焦点 + 未要求减少动效；三者任一不满足返回 false
│   │   ├── useGlassSpecular.ts # 玻璃高光跟指针：一个委托 pointermove 写 --gx/--gy 到悬停的面板
│   │   └── useBackendEvent.ts # 后端事件订阅（StrictMode 安全的 token 交接），四个事件共用
│   ├── stores/             # 状态管理
│   │   └── routerStore.ts  # Zustand store
│   ├── i18n/               # 国际化
│   │   ├── index.ts
│   │   ├── i18next.d.ts
│   │   └── locales/
│   │       ├── en.json
│   │       └── zh-CN.json
│   ├── lib/                # 工具函数
│   │   ├── invoke.ts       # Tauri invoke 封装
│   │   ├── delay.ts        # 延迟步进/钳制/范围换算
│   │   ├── stage.ts        # 舞台几何：中心/圆盘/辐条的尺寸与角度（辐条两端由此算出，不测量）+ 三种角色各自的配色类名
│   │   ├── motion.ts       # 全局共享动效曲线（tap / glide / route / fade）
│   │   ├── types.ts        # 共享类型定义
│   │   └── window.ts       # 窗口控制（懒加载 Tauri API）
│   └── styles/
│       └── index.css       # Tailwind 入口 + CSS variables
├── .github/workflows/      # CI/CD
│   └── release.yml
├── AGENTS.md               # 本文件
├── package.json
├── vite.config.ts
├── tsconfig.json
└── tailwind.config.ts
```

---

## 代码规范

### TypeScript / React

1. **严格模式**：`tsconfig.json` 启用 `strict: true`、`noUncheckedIndexedAccess: true`
2. **类型导出**：所有公共类型必须显式 `export type`
3. **函数组件**：使用 `React.FC<Props>` 或普通函数签名，禁止使用 `any`
4. **Hooks 规则**：遵守 Rules of Hooks，不在条件/循环中调用
5. **命名**：
   - 组件：`PascalCase` (`ProgramRail.tsx`)
   - Hooks：`use` 前缀 (`useTheme.ts`)
   - 工具函数：`camelCase`
   - 常量：`SCREAMING_SNAKE_CASE`
   - 类型/接口：`PascalCase`，**不**加 `I` 前缀
6. **Tailwind**：优先 utility class，禁止自定义 CSS 类（除非通过 `@apply` 或 CSS variables）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eververdants/AppAudioRouter](https://github.com/Eververdants/AppAudioRouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
