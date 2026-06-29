---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

Komari Web UI 的「Editorial Paper」主题。设计语言参考 Stripe Press / Maggie Appleton / Are.na: 暖白纸面 + Fraunces 衬线印刷骨架 + 极克制的 Caveat 手写批注 + JetBrains Mono 等宽数据。保持 Komari 原有数据层 (RPC2 + Context) 不变。

## Development Commands

- `npm install` — 安装依赖
- `npm run dev` — 启动 Vite 开发服务器
- `npm run build` — TypeScript 类型检查 + Vite 生产构建（输出到 `dist/`）
- `npm run lint` — ESLint
- `npm run preview` — 预览生产构建

## Architecture

### 数据层 (保留 Komari 原架构)
- `src/contexts/`: `PublicInfoContext`、`LiveDataContext` (实时数据)、`NodeListContext` (节点元信息)、`RPC2Context` (RPC 调用)
- `src/lib/api.ts`、`src/lib/rpc2.ts`: API / RPC 客户端
- `src/types/LiveData.tsx`: `LiveData`、`Record` 实时数据类型
- `src/contexts/NodeListContext.tsx`: `NodeBasicInfo` 节点元信息类型

### 视觉层 (Editorial Paper)

#### 全局样式 `src/global.css`
- **设计令牌**: 单一 light 主题, 暖白纸 + 暖灰墨, 不支持暗色
  - `--paper`: 主纸张 (#f4efe6, 暖白偏 Stripe Press 取色)
  - `--paper-card`: 卡片本体 (#fbf7ee, 近米白)
  - `--paper-cool`: 冷调副纸 (#ebe9e1, 用于侧栏 / 二级面板, 与主纸对比)
  - `--paper-soft`: hover / 内嵌格子色
  - `--ink` / `--ink-soft` / `--ink-mute`: 暖灰黑系文字 (#2a2622 / #4d463d / #7c7062)
  - `--ink-line` / `--ink-line-soft`: 边框 (主文字色实色 / 22% 透明软描边)
  - `--pen-red` / `--pen-blue`: 红蓝铅笔批注色 (其他笔色已删, 不堆多色)
  - `--data-1..5`: Recharts / 数据可视化色环 (与笔色同源, 偏冷静)
  - `--rule-line` (10% 暖墨色) / `--margin-line` (实红色, 装订线) / `--highlight` (荧光黄)
- **纸纹背景**: 单层 SVG `feTurbulence` 极淡噪点 (data URI 内联), 不再画笔记本横线 (=作业本廉价感)
- **字体栈**:
  - `--font-serif`: **Fraunces** (variable opsz/SOFT/WONK) + LXGW WenKai (中) — 主标题 / 正文 / UI
  - `--font-hand`: **Caveat** + LXGW WenKai — **仅** 用于批注 / 标签 / 空状态
  - `--font-mono`: **JetBrains Mono** + tabular-nums — 数字 / 流量 / 表格 / Recharts 坐标
  - 中文统一 LXGW WenKai (霞鹜文楷, jsDelivr CDN)
- **OpenType 特性**: `font-variant-numeric: oldstyle-nums proportional-nums` 默认; `.font-tabular` / `.font-mono` 切到 `tabular-nums lining-nums`
- **暗色变体**: 不存在。`@custom-variant dark` 指向 `.never-active`, shadcn / Tailwind 的 `dark:` 前缀永不命中
- **没有 backdrop-filter**: 整个项目不再使用毛玻璃 / SVG 折射

#### 卡片类
- `.paper-card`: 主卡片 (Node 卡 / Summary / 详情弹窗 / 图表块)
  - 暖白底 + 1px 软描边 (主文字色 22% 透明) + 几乎方角 (`2px 3px 2px 3px`)
  - 双层阴影: 1px 硬偏移 (纸厚) + 12-24px 大柔影 (悬浮)
  - **卡片不再歪斜**: `--tilt` 旋转 + `nth-child(6n+1..6)` 角度规则已全部移除
  - hover: 仅 translateY(-2px) + 阴影加深, 35ms cubic-bezier (无旋转回正)
  - 工具类 `.no-tilt`: 卡片歪斜已全局移除, 此类退化为 no-op 别名, 兼容历史标记
- `.paper-strip`: NavBar / 不可旋转的纸条 (单层硬阴影)

#### Editorial 装饰类 (按需用, 高度克制)
- `.eyebrow`: 杂志栏目名 (Fraunces italic + 字间距 + 全大写 + 极小号)
- `.drop-cap::first-letter`: 段首大字母, Fraunces 900 + opsz 144 + SOFT 100
- `.editorial-rule` / `.thick`: 极细 (1px) / 加粗 (2px) 水平分隔线
- `.editorial-margin::before`: 元素左侧 1px 红墨竖线 (装订线 / 强调段)
- `.corner-stamp`: 卡片右上角圆形红圈印章 (Caveat 字 + 旋转 -12deg, absolute)
- `.hand-underline`: 红波浪下划
- `.hand-highlight`: 荧光笔黄背景
- `.font-hand`: 强制 Caveat (用作批注/便签贴)

#### 动效组件 (手撸, 不引第三方动画库)
- `src/components/SplitText.tsx`: 字符级 stagger 入场, `Array.from` 拆 codepoint (兼容中文/emoji); 默认 step 35ms / duration 520ms / ease-out cubic + 微下移 + 2px blur; `[data-split-text] .split-text-char` keyframe 在 `global.css` 定义, `prefers-reduced-motion` 自动停。**只用于首屏封面感**: NavBar 站名 / 详情页 H1 服务器名
- `src/components/CountUp.tsx`: `requestAnimationFrame` + ease-out cubic 数字滚动, 600ms; 接收 `string | number`, 仅滚动第一段整数 (前后缀如 ` / 10` 原样穿过); `currentRef` 兜底动画中断; reduced-motion 直显; 用于首页 Summary 在线节点数等纯整数项, 不用于带单位/方向箭头的传输量

#### 关键组件
- `src/components/DynamicBackground.tsx`: 极简, 仅渲染极淡 vignette + 用户自定义背景图 (sepia + multiply 半透明叠层, "夹照片"风格); 不再画咖啡渍 / 墨点
- `src/components/Node.tsx`: 便签风格紧凑节点卡 (`paper-card + node-card`), 信息密度高且分三级层次; `NodeGrid` 响应式网格 (手机 1 / 平板 2 / 笔记本 3 / 大屏 `2xl` 4 列, `items-stretch` 同行等高)
  - **三级信息层次** (靠底色分层 + 字阶, 不堆边框): ① 资源块 (`.node-metric-block` 冷调副纸凹陷, 视觉重心) ② 网络/健康 (主纸平铺墨色) ③ 脚注 (`.node-card-meta` 淡色 + 顶部细线)
  - 卡内结构 (在线): 头 (旗 + 名 Fraunces 600/opsz48 + `#group` 红铅笔批注) → **资源 2×2** (CPU/内存/磁盘/负载, 每格 `eyebrow` 标签 + mono 数值 + `UsageBar compact` + 已用/总量小字) → **网络 2 列** (实时网速 ↑↓ / 总流量 ↑↓) → **健康 2 列** (延迟 / 丢包, 数字带阈值色 + `MiniBars` 迷你柱图) → `BillingBar` (价格文本 + 到期剩余进度条) → **脚注** (TCP·UDP·进程 + uptime / OS·arch·CPU型号, truncate + title) → tags (`PriceTags layout="grid2" showPrice={false}`, 仅自定义 tags, 无 tags 不占位)
  - 价格/到期不再走底部 badge: 改由 `BillingBar` 在脚注上方渲染 (价格 `Wallet` 图标 + 文本; 到期 = 已过周期占比进度条 + 剩余天数, 阈值色 ≤7 红 / ≤15 橙 / 其余绿; 免费/长期/一次性/无周期不画条)
  - 卡片 `flex flex-col h-full gap-2.5`; tags 区 `mt-auto` (有 tags 才渲染), 网格同行等高靠 `items-stretch`
  - 负载基准: `load1 / cpu_cores` 折百分比驱动 bar; 多 ping task 取延迟最低的 task
  - 离线: 红铅笔脉冲点 + 名 + 分组 + Caveat "offline" 批注 + 价格 (离线卡仍走 `PriceTags` 带价格); ping 不拉取
  - `formatUptime` 仍 export (被 `DetailsGrid` / `NodeTable` 引用)
- `src/hooks/useNodePing.ts`: 便签卡 ping 数据 hook —— RPC2 `common:getRecords` 按 uuid 拉最近 1h, 汇总成 `{latest, loss, values[]}`
  - **控制首页 N 节点请求量**: 模块级 `cache` (同 uuid 多组件共享一份, 不重复请求) + 60s 节流 + 按 uuid 错峰 0~800ms 首拉 + 离线节点 `enabled=false` 不拉
  - 多 task 取延迟最低者; 丢包率 = `value<0 计数 / 总计 × 100`; 失败/无数据返回 null 不阻塞
- `src/components/MiniBars.tsx`: 纯 CSS 迷你柱图 (不引 Recharts), 柱高 = 延迟相对峰值占比, 阈值色 (≤80 苔绿 `--data-2` / ≤200 赭黄 `--data-3` / 更高 + 丢包点红 `--pen-red`)
- `src/components/DetailsGrid.tsx`: 单节点详情主信息区, 三段 (Live 实时指标 / GPU / Spec 静态规格)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [woodchen-ink/komari-paper](https://github.com/woodchen-ink/komari-paper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
