---
trigger: always_on
description: name: remix-animation
---

---
name: remix-animation
description: 'Generate motion graphics and animation videos with Remotion. Use when the user asks for 动画, 动效, video, motion graphics, transition, 片头, GIF animation, GSAP animation, or wants to turn static images, SVG, or text into motion. Search open-source references, adapt them to Remotion, render the result, and save reusable local templates.'
argument-hint: 'Describe the animation, duration, resolution, fps, output format, and any GSAP or reference link if available.'
---

# Skill: remix-animation

> 基于 Remotion 官方 prompt-to-motion-graphics 模板改造，加入"开源社区检索+Agent改编"能力，
> 让 Agent 能从 GSAP/Three.js/Lottie 等开源社区找到参考动画并改编为 Remotion 视频输出。

## 默认目标

- resolution: 默认 1440x1440（至少 1K；用户未指定时优先高分辨率）
- fps: 默认 120（最低 60；高复杂度或受限环境可降到 60）

## 资源引入

进入实现前，优先加载以下资源：

- [Remotion system prompt](./prompts/remotion-system-prompt.md)
- [GSAP skills / GSAP to Remotion rules](./rules/gsap-to-remotion.md)

如果用户请求明确涉及 GSAP、CodePen、timeline、ScrollTrigger 或已有 GSAP demo，必须同时读取上面的 GSAP 规则文档，再决定走“代码转换”还是“直接录制”路径。

## 触发条件

当满足以下任一条件时激活此 skill：

- 用户要求制作动画、视频、Motion Graphics
- 用户描述了一种具体的动画效果（如"粒子爆炸"、"文字飞入"、"3D地球旋转"、"数据图表动画"）
- 用户要求将静态内容（图片、SVG、文字）变为动态
- 触发关键词：动画、animation、motion、视频特效、动效、GIF、片头、转场

## 架构基础

本 skill 基于以下现有项目构建，**不重复造轮子**：

| 项目 | 作用 | 我们如何使用 |
|------|------|-------------|
| [create-vibe-motion](https://github.com/vibe-motion/create-vibe-motion) | Remotion 项目脚手架（npm: `create-vibe-motion`） | **作为兜底脚手架**：`scripts/init-workspace.sh` 优先复制仓库内置模板，缺失时才调用它 |
| [Remotion System Prompt](https://remotion.dev/docs/ai/system-prompt) | 教 LLM 写 Remotion 代码的标准 prompt | **作为 Agent 知识库**：[prompts/remotion-system-prompt.md](prompts/remotion-system-prompt.md) |
| [template-prompt-to-motion-graphics](https://github.com/remotion-dev/template-prompt-to-motion-graphics) | Remotion 官方 AI→动画 SaaS 模板 | **作为参考**（仅设计思路），不用来脚手架——我们优先使用仓库内置工作区模板 |
| [@remotion/three](https://remotion.dev/docs/three) | Three.js 官方适配 | **直接使用**：Three.js 动画无需手动转换 |
| [@remotion/lottie](https://remotion.dev/docs/lottie) | Lottie JSON 播放 | **直接使用**：Lottie 动画无需手动转换 |
| [remotion-animated](https://github.com/stefanwittwer/remotion-animated) | 声明式动画库 | **作为动画表达层**：简化动画编写 |
| [remotion-animation](https://github.com/ahgsql/remotion-animation) | animate.css → Remotion 桥梁 | **直接使用**：CSS 动画无需手动转换 |
| [gsap-video-export](https://github.com/workeffortwaste/gsap-video-export) | GSAP 动画逐帧录制导出视频 | **GSAP 路径 B**：无法代码转换时，直接录制 GSAP 动画为视频 |

> **为什么不用 GitHub clone `template-prompt-to-motion-graphics`？**
> 该仓库是 Vercel/Remotion 官方演示项目，依赖 SaaS 后端（OpenAI Key + Webhook），不适合本地命令行 Agent。
> `create-vibe-motion` 是社区维护的纯本地 Remotion 脚手架，npm 上一键 `npx` 创建，更适合 Agent 工作流。

### 我们的差异化能力（本 skill 新增）

官方模板只能让 AI "从零生成"动画代码。我们在此基础上增加：

```
📌 开源社区检索 → 找到相似动画参考
📌 Agent 智能改编 → 基于参考代码修改适配
📌 本地模板沉淀 → 改编成功自动保存，越用越快
📌 GSAP 双路径  → 代码转换 or 直接录制，灵活选择
```

---

## 前置条件

### 系统要求

- Node.js >= 18
- pnpm
- Chrome/Chromium，或本地 `chrome-headless-shell`（Remotion 渲染需要）
- ffmpeg（GSAP 录制 + 缩略图生成需要，可选）

### ⚠️ 不需要 OpenAI API Key

**与官方 `template-prompt-to-motion-graphics` 不同，本 skill 不依赖 OpenAI**。
所有代码生成与改编由 Agent 自己完成（Agent 本身就是 LLM），无需调用外部 LLM API。

### 环境初始化（仅首次，一行命令）

```bash
# Linux 全自动安装（Node.js / pnpm / Chromium 或 headless-shell / gsap-video-export / Remotion 工作区）
chmod +x scripts/setup-linux.sh scripts/init-workspace.sh scripts/smoke-test.sh
./scripts/setup-linux.sh

# 或手动（如果 Node.js 18+ / pnpm / 浏览器已就绪）：
./scripts/init-workspace.sh
```

补充说明：
- 仓库已自带 `remix-workspace/` 模板，初始化优先复用它。
- 无 sudo 的 Linux / WSL 环境会自动回退到本地 `chrome-headless-shell`。
- `ffmpeg` 不是基础 Remotion 渲染的硬依赖，但 `gsap-video-export` 和模板 GIF 缩略图需要它。

### 验证安装

```bash
chmod +x scripts/smoke-test.sh && ./scripts/smoke-test.sh
```

成功时会渲染一段测试动画到 `remix-workspace/out/smoke-test.mp4`。

---

## 执行流程

```
Step 1: 解析需求
Step 2: 检索本地模板
Step 3: 多源社区检索（如果本地未命中）
Step 4: 评估筛选
Step 5: 代码获取 + 改编
Step 6: 渲染输出
Step 7: 失败处理
Step 8: 沉淀为本地模板
```

---

### Step 1: 解析用户需求

从用户自然语言中提取结构化意图：

```json
{
  "animation_type": "",
  "style": "modern-minimal",
  "colors": [],
  "text": "",
  "duration_seconds": 5,
  "resolution": "1440x1440",
  "fps": 120,
  "output_format": "mp4",
  "reference_description": ""
}
```

**默认值**：未提及的字段使用默认值，不反复确认。若用户未给风格，默认使用“现代化简约（modern-minimal）”。若用户未给技术指标，默认目标为至少 1K 分辨率且 >60fps，优先 120fps。极度模糊时询问一次动画类型即可。

---

### Step 2: 检索本地模板库

检查 `local-templates/` 目录是否有匹配的已沉淀模板。

- 匹配方式：metadata.json 中的 tags 与用户需求语义匹配
- 匹配阈值：相似度 > 70% 视为命中
- **命中** → 跳到 Step 5 改编阶段
- **未命中** → 进入 Step 3

---

### Step 3: 多源社区检索

将用户描述转为英文搜索关键词，从以下来源并行检索（每源 Top 3）：

| 优先级 | 来源 | Agent 实际可用的检索工具 | 适合类型 |
|--------|------|------------------------|----------|
| 1 | GitHub | `github_text_search` / `github_repo` (跨 scope 搜)；`{kw} animation language:javascript sort:stars` | 通用 |
| 2 | CodePen | `fetch_webpage("https://codepen.io/search/pens?q={kw}&tag=gsap")` | DOM/SVG/CSS |
| 3 | NPM | `run_in_terminal` 跑 `npm search {kw} remotion` | Remotion 原生组件 |
| 4 | LottieFiles | `fetch_webpage("https://lottiefiles.com/search?q={kw}")` | 矢量动画/图标动效 |
| 5 | Shadertoy | `fetch_webpage("https://www.shadertoy.com/results?query={kw}")` | GPU 着色器特效 |

> **降级**：如果当前环境没有上述工具(例如纯 LLM 离线),Agent 应承认"无法检索",并直接进入 Step 5 分支 B(从零生成)。

#### Step 3 输出要求（硬门槛）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [embalmer-Y/remix-animation](https://github.com/embalmer-Y/remix-animation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
