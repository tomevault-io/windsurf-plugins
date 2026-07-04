---
trigger: always_on
description: > **Project**: 个人主页 (Personal Homepage)
---

# Agent Instructions — Bento Homepage

> **Project**: 个人主页 (Personal Homepage)
> **Type**: 纯静态单页应用 (Static SPA)
> **Architecture**: Config-Driven Bento Grid with Shared Liquid Glass Renderer
> **Live**: [iacg.moe](https://iacg.moe)

---

## Selected Tech Stack

| Layer | Technology | Version | Selection Rationale |
|---|---|---|---|
| **Framework** | Next.js (App Router) | 16.x | 支持 `output: "export"` 静态导出，App Router 提供 RSC + Metadata API |
| **UI Runtime** | React | 19.x | Next.js 16 默认绑定 React 19，支持 Server Components |
| **Styling** | Tailwind CSS | 4.x | 原子化 CSS 极适合静态站点，Tree-shaking 确保最小产物 |
| **Animation** | Framer Motion | 12.x | spring 物理动画 + `whileTap`/`whileHover` 声明式交互 |
| **Icons** | lucide-react | latest | 轻量级 SVG 图标库 + 自定义 SVG (VRChat/Steam) |
| **Language** | TypeScript | 5.x | 严格模式启用，确保配置文件类型安全 |
| **Package Manager** | pnpm | 10.x | 磁盘高效 + 严格依赖隔离，`packageManager` 字段锁版本 |
| **Deployment** | GitHub Pages | — | 通过 GitHub Actions 自动 `next build` 静态导出 |
| **CI/CD** | GitHub Actions | — | pnpm/action-setup@v4 自动检测版本 |

### 不选型项

| Technology | Reason for Exclusion |
|---|---|
| Database | 纯静态站点，所有数据由 `src/config/site.ts` 驱动 |
| Backend API | 无服务端逻辑；网易云/GitHub/VRChat 数据通过公开 API 获取 |
| Authentication | 公开展示页面，无需登录 |
| State Management | 单页无交互状态流转，React 内置 state 足够 |
| CSS-in-JS | Tailwind CSS + globals.css 已覆盖所有样式需求 |
| Shadcn/UI | 所有 UI 组件均为自研，无外部组件库依赖 |

---

## Agent Role Definitions

### 1. `Frontend_Architect` (前端架构师)

**Responsibilities**:
- 维护 Next.js 项目结构与目录规范
- 配置 Tailwind CSS 主题 (colors, fonts, breakpoints)
- 定义 TypeScript 类型系统 (`SiteConfig` interface)
- 维护 `src/config/site.ts` 配置驱动架构
- 确保 `next.config.ts` 正确配置 `output: "export"` 静态导出
- 维护共享 Liquid Glass 配置 (`src/lib/liquid-glass.ts`) 与 shader 导入配置

**Constraints**:
- 所有颜色值必须通过 CSS 变量管理，禁止硬编码 HEX
- 响应式断点严格使用 Tailwind 预设 (`sm`, `md`, `lg`)
- 组件必须为 Server Components by default，仅在需要交互时标记 `"use client"`

---

### 2. `UI_Developer` (UI 开发者)

**Responsibilities**:
- 维护 `GlassCard` 核心组件（卡片注册、variant、fallback 与内容层栈）
- 维护 `LiquidGlassCanvas` 共享 WebGL2 画布（bgPass → vBlurPass → hBlurPass → mainPass）
- 实现 Bento Grid 4 列布局容器
- 实现各功能卡片（Profile、Social、Hardware、Projects、Friends、NowPlaying、PhotoStack、GitHubHeatmap、VRChatStatus、Blog、Software、Map、Weather）
- 实现入场动画（stagger + spring）
- 性能优化（共享画布、rAF 驱动进度条、避免多层 blur 叠加、减少布局探测）
- 确保触控目标 ≥ 44×44pt (Apple HIG)

**Constraints**:
- 所有动画必须使用 spring 物理曲线，禁止 linear/ease-in-out
- 动画必须可被用户交互打断 (interruptible)
- 所有组件 API 仅从 `siteConfig` 读取数据，禁止组件内硬编码文本
- 外层 `GlassCard` 负责液态玻璃壳层与注册，不负责媒体裁剪；媒体裁剪必须下沉到子容器并使用 `borderRadius: inherit`
- 玻璃光学参数必须通过 `GlassVariant` / CSS 变量集中管理，禁止在业务卡片中复制 shader 参数
- 内层控件使用组件局部样式并复用现有 glass / tint token；禁止重新引入 Prism 风格玻璃参数或新的共享内层视觉体系

---

### 3. `DevOps` (部署与 CI/CD)

**Responsibilities**:
- 维护 GitHub Actions workflow 自动构建部署
- 维护自定义域名配置（`public/CNAME`）
- 确保 `packageManager` 字段与 CI pnpm 版本一致

**Constraints**:
- 构建产物必须是纯静态文件 (HTML/CSS/JS/Images)
- 部署通过 `actions/deploy-pages` 至 GitHub Pages
- pnpm 版本由 `package.json` 中 `packageManager` 字段驱动，CI 不再硬编码版本

---

### 4. `QA` (质量保障)

**Responsibilities**:
- 验证所有功能卡片在 Desktop/Tablet/Mobile 三端的响应式表现
- 验证 GlassCard 3D tilt 动画在各浏览器的流畅度
- 验证网易云音频播放功能（加载、播放、暂停、切歌、循环）
- 验证 VRChat 状态卡片的实时轮询与状态显示
- 验证 GitHub 热力图的数据加载与响应式滚动
- 验证 SEO meta 标签完整性
- 验证所有外部链接的 `rel="noopener noreferrer"` 属性

**Constraints**:
- 浏览器兼容性: Chrome 90+, Safari 15+, Firefox 90+
- 移动端: iOS Safari 15+, Chrome Android 90+

---

## Project Structure

```
Bento-Homepage/
├── .agent/
│   ├── rules/                    # AI 行为约束
│   └── workflows/                # 标准化工作流（CI/CD、开发循环）
├── .github/workflows/
│   └── deploy.yml                # GitHub Actions → Pages 流水线
├── public/
│   ├── cat.png                   # 默认头像
│   ├── CNAME                     # 自定义域名 (iacg.moe)
│   ├── avatar/                   # 多头像目录（3D 轮播）
│   ├── bg/                       # 背景图目录（多张自动轮播）
│   ├── photos/                   # 照片堆叠目录
│   └── optimized/                # WebP 降采样运行时资源（bg/photos）
├── src/
│   ├── app/
│   │   ├── globals.css           # 设计令牌（明/暗）、玻璃样式、动画关键帧
│   │   ├── layout.tsx            # 根布局、SEO 元数据、背景图扫描、主题注入
│   │   └── page.tsx              # 首页 — Bento Grid 组装 + 数据 fetch
│   ├── components/
│   │   ├── glass-card.tsx        # 核心卡片壳层（variant + 注册 + fallback）
│   │   ├── bento-grid.tsx        # 4 列响应式网格容器
│   │   ├── background-layer.tsx  # 背景轮播 + 渐变遮罩 + 浮动光球 + 噪点纹理 + active bg 发布
│   │   ├── liquid-glass-provider.tsx # Client wrapper，注入共享 LiquidGlassCanvas
│   │   ├── liquid-glass-canvas.tsx   # 共享 WebGL2 画布，渲染所有 GlassCard 壳层
│   │   ├── profile-card.tsx      # 头像轮播 + 多语言问候 + 打字机 + i18n 简介
│   │   ├── avatar-carousel.tsx   # 多头像旋转 3D 轮播组件
│   │   ├── typewriter.tsx        # 打字机效果组件
│   │   ├── now-playing-card.tsx  # 网易云音乐播放器（真实音频 + rAF 进度条）
│   │   ├── photo-stack-card.tsx  # 照片堆叠卡片（点击展开/收起）
│   │   ├── github-heatmap-card.tsx # GitHub 贡献热力图（无需 Token）
│   │   ├── vrchat-status-card.tsx  # VRChat 实时在线状态（VRCX-Cloud API）
│   │   ├── blog-card.tsx         # 博客最新文章（Halo 2.x API）
│   │   ├── social-card.tsx       # 社交图标链接
│   │   ├── skills-card.tsx       # 兴趣标签
│   │   ├── hardware-card.tsx     # 硬件清单（与 Interests 一致）
│   │   ├── projects-card.tsx     # 项目展示（含 GitHub Stars/Forks API）
│   │   ├── friends-card.tsx      # 友情链接（轻量 hover 反馈）
│   │   ├── map-card.tsx          # Mapbox 互动地图（城市标记 + 脉冲弹窗 + IP 距离显示）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ero-Cat/Bento-Homepage](https://github.com/Ero-Cat/Bento-Homepage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
