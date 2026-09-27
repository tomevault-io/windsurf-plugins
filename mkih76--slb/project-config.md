---
trigger: always_on
description: > 给 Claude Code / Codex 等 AI 编码代理使用。
---

# AGENTS.md — SLB 申论AI批改平台构建指引

> 给 Claude Code / Codex 等 AI 编码代理使用。
> 本文件定义完整项目结构、代码规范和构建顺序。

---

## 一、项目概述

- **项目名**：SLB（申论帮，ShenLunBen）
- **仓库**：https://github.com/mkih76/SLB
- **核心功能**：粉笔批量抓卷 → 数据库存储 → 用户提交答案 → AI多维度批改 → 好词好句鉴赏
- **目标用户**：备考公务员/事业单位考生
- **技术栈**：Python(Flask) + SQLite + LLM API + Bootstrap/Tailwind + Telegram Bot
- **部署环境**：VPS + Docker + Caddy（与 gaokao-number 共用基础设施）

---

## 二、UI设计规范

> **设计语言**：Stripe 风格（详见 [DESIGN.md](DESIGN.md)）
> 深海军蓝 + 靛蓝强调 + 金色点缀 = 金融级信任感 × 公务员考试权威气质

### 2.1 色彩体系
| 用途 | 色值 | 说明 |
|------|------|------|
| 主色（深海军蓝） | `#1c1e54` | 品牌色，正气官方感 |
| 主色（靛蓝） | `#533afd` | 功能强调色，按钮/链接 |
| 辅色/点缀 | `#9b6829` | 金色，权威感 |
| 金色浅底 | `#f5e9d4` | 得分卡片背景 |
| 画布 | `#ffffff` | 纯白背景 |
| 表面 | `#f6f9fc` | 浅灰蓝卡片底色 |
| 正文文字 | `#0d253d` | 深墨蓝 |
| 次要文字 | `#273951` | 蓝灰 |
| 弱化文字 | `#64748d` | 灰色说明 |
| 边框 | `#e3e8ee` | 发丝线 |
| 成功色 | `#24a148` | 绿色（得分高） |
| 警告色 | `#f5a623` | 橙色 |
| 错误色 | `#ee0000` | 红色（低分） |

### 2.2 字体规范
- 标题：SF Pro Display / Inter，300 轻字重，紧凑字间距（-0.96px）
- 正文：SF Pro Text / Inter，400 常规
- 数字/等宽：JetBrains Mono / Geist Mono

### 2.3 组件规范
- 卡片：圆角12px，`#f6f9fc` 底色，1px `#e3e8ee` 边框，无阴影
- 按钮：圆角24px（药丸形），主色 `#533afd` 白字，金色渐变可选
- 输入框：1px `#e3e8ee` 边框，聚焦时 `#533afd` 边框
- 表格：无斑马纹，行悬停 `#f6f9fc` 背景

### 2.4 动效与过渡规范

> **原则**：动效服务于反馈，不是装饰。Stripe 风格追求"安静的专业感"——用户感知到流畅，但说不出为什么。

#### 2.4.1 时长与缓动

| 场景 | 时长 | 缓动函数 | 说明 |
|------|------|----------|------|
| 颜色/背景变化 | 150ms | `ease-out` | hover、focus、active 态切换 |
| 尺寸/位移变化 | 200ms | `cubic-bezier(0.4, 0, 0.2, 1)` | 展开、折叠、滑入 |
| 模态框/抽屉 | 250ms | `cubic-bezier(0.16, 1, 0.3, 1)` | 进入用 ease-out，退出用 ease-in |
| 页面级过渡 | 300ms | `ease-in-out` | 路由切换、Tab 内容切换 |
| 骨架屏闪烁 | 1.5s | `ease-in-out` 循环 | `opacity: 1 → 0.4 → 1` |

**禁止**：`linear` 缓动（机械感）、超过 500ms 的任何动画（拖沓）。

#### 2.4.2 按钮交互态

```css
/* 四态完整定义 */
.btn-primary {
  background: #533afd;
  color: #ffffff;
  transition: all 150ms ease-out;
}
.btn-primary:hover {
  background: #665efd;           /* primary-soft，微亮 */
  transform: translateY(-1px);  /* 微浮起 */
}
.btn-primary:active {
  background: #4434d4;          /* primary-deep，按压变深 */
  transform: translateY(0);     /* 回弹 */
}
.btn-primary:focus-visible {
  outline: 2px solid #533afd;
  outline-offset: 2px;
}
.btn-primary:disabled {
  background: #e3e8ee;
  color: #888888;
  cursor: not-allowed;
  transform: none;
}
```

#### 2.4.3 表单交互

| 元素 | 交互行为 |
|------|----------|
| 输入框聚焦 | 边框 `#e3e8ee` → `#533afd`，150ms 过渡，外圈 2px `rgba(83,58,253,0.15)` 光晕 |
| 输入框失焦 | 反向过渡，150ms |
| 校验错误 | 边框变 `#ee0000`，下方滑入红色提示文字（200ms），输入框轻微 shake（300ms，`translateX` ±4px × 3次） |
| 校验通过 | 边框变 `#24a148`，右侧 fadeIn 勾号图标（150ms） |
| 密码可见性切换 | 图标 crossfade 150ms，输入内容不闪烁 |

#### 2.4.4 卡片与列表

```css
/* 卡片 hover 微升 */
.card {
  transition: box-shadow 150ms ease-out, transform 150ms ease-out;
}
.card:hover {
  box-shadow: 0 4px 12px rgba(13, 37, 61, 0.08);
  transform: translateY(-2px);
}

/* 列表项进入动画 — 交错淡入 */
@keyframes fadeSlideIn {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}
.list-item {
  animation: fadeSlideIn 200ms ease-out forwards;
}
.list-item:nth-child(1) { animation-delay: 0ms; }
.list-item:nth-child(2) { animation-delay: 30ms; }
.list-item:nth-child(3) { animation-delay: 60ms; }
/* 最多延迟到第 8 项 (210ms)，之后的不延迟 */
```

#### 2.4.5 加载状态

| 场景 | 方案 | 细节 |
|------|------|------|
| 页面首次加载 | **骨架屏** | 模拟内容布局的灰色块，`#f6f9fc` 底 + `#e3e8ee` 骨架条，闪烁周期 1.5s |
| 按钮提交 | **按钮内 spinner** | 按钮文字替换为旋转图标 + "提交中..."，按钮 disabled，宽度不变 |
| AI 批改中 | **进度条 + 状态文字** | 顶部细进度条（2px，`#533afd`），下方显示"正在分析踩点命中…""正在评估逻辑结构…"轮播 |
| 列表加载更多 | **底部 spinner** | 小尺寸旋转图标 + "加载更多…"，不遮挡已有内容 |
| 局部刷新 | **内容区 skeleton** | 仅刷新区域显示骨架，不影响页面其他部分 |

**骨架屏 CSS**：
```css
.skeleton {
  background: linear-gradient(90deg, #e3e8ee 25%, #f6f9fc 50%, #e3e8ee 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
  border-radius: 4px;
}
@keyframes shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

#### 2.4.6 反馈提示（Toast / Notification）

| 类型 | 颜色 | 图标 | 自动消失 |
|------|------|------|----------|
| 成功 | `#24a148` 左边框 + `#f0fdf4` 背景 | ✓ | 3s |
| 错误 | `#ee0000` 左边框 + `#fef2f2` 背景 | ✕ | 5s（或手动关闭） |
| 警告 | `#f5a623` 左边框 + `#fffbeb` 背景 | ⚠ | 4s |
| 信息 | `#533afd` 左边框 + `#f5f3ff` 背景 | ℹ | 3s |

**动画**：从右上角滑入（`translateX(100%)` → `translateX(0)`，250ms ease-out），退出时 fadeOut + slideUp 200ms。

#### 2.4.7 模态框 / 抽屉

| 元素 | 行为 |
|------|------|
| 遮罩层 | `rgba(0,0,0,0.4)`，fadeIn 200ms，点击可关闭 |
| 模态框 | 从 `scale(0.95) + opacity:0` → `scale(1) + opacity:1`，250ms，居中 |
| 抽屉（侧边） | 从 `translateX(100%)` → `translateX(0)`，250ms，右侧滑入 |
| 关闭 | 反向动画，时长减 50ms（200ms），ESC 键触发 |
| 焦点陷阱 | 打开时焦点锁定在模态内，Tab 循环，关闭后焦点回到触发元素 |

#### 2.4.8 分页与滚动

| 场景 | 方案 |
|------|------|
| 分页切换 | 内容区 fadeOut 100ms → 替换 → fadeIn 200ms，不跳动 |
| 无限滚动 | 触底前 200px 预加载，新内容交错淡入 |
| 回到顶部 | `scrollTo({ behavior: 'smooth' })`，300ms |
| 锚点跳转 | smooth scroll + 目标位置 offset 80px（避开固定导航栏） |

#### 2.4.9 AI 批改结果展示（SLB 专属）

这是 SLB 最核心的交互场景，需要特殊设计：

```
提交答案
  ↓
按钮 → spinner + "AI 正在批改…"
  ↓
顶部进度条开始推进（分 4 段）：
  [踩点分析 ████████░░] → [逻辑评估 ░░░░░░░░░░] → [语言检查 ░░░░░░░░░░] → [综合评分 ░░░░░░░░░░]
  ↓
结果卡片从下方滑入（300ms ease-out）：
  ┌─────────────────────────────────────┐
  │  综合得分        78.5 / 100         │ ← 金色 #f5e9d4 底，数字 countUp 动画
  │  ─────────────────────────────────  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mkih76/SLB](https://github.com/mkih76/SLB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
