---
trigger: always_on
description: 把一个技术概念/流程/系统转化为"剧场式"动画 HTML — 多场景 + 自动播放 + 章节导航 + 旁白与真实代码。适用于网络协议、底层原理、算法、架构原理等"看不见但需要看见"的主题。单文件 HTML，无构建步骤。
---


# Animation Explainer · 知识动画化技能

## 这个技能是什么

输入一个主题（"打开浏览器到屏幕上像素的全过程"、"一个 SQL 查询是怎么被执行的"、"TLS 1.3 握手"、"V8 是怎么编译 JS 的"），输出一份**单文件 HTML 动画演示**，能在任意浏览器打开播放。

参考实现：`examples/browser-request/index.html`。**任何新主题都从复制这个文件开始**，而不是从零写。

## 它不是什么

- ❌ 不是幻灯片：每节有 SVG 动画，元素会移动、闪烁、依次出现
- ❌ 不是单页长滚动信息图：内容被切成离散场景，自动按节奏播放
- ❌ 不是 PPT 截图导出工具：纯 HTML+SVG+CSS+原生 JS，没有打包步骤
- ❌ 不是通用 explainer 框架：只解决一类问题 — **多场景剧场式技术解说**

---

## 核心结构（必须遵守）

### 文件
- **单一 `index.html`**，所有 CSS/JS/SVG 内联。零外部依赖（除了 Google Fonts，可选）。
- 复制 `examples/browser-request/index.html` 改，**不要从空文件开始**。

### 页面骨架（grid）
```
┌─────────────────────────────────────────────┐
│ 顶栏: brand + chips + 进度条 + N/总数        │  ← 章节导航
├─────────────────────────────────────────────┤
│  舞台 (SVG 全屏)        │  旁白面板          │
│  ─ 当前场景的动画区     │  - 节号 + 大标题   │  ← 主体（1.55 : 1）
│  ─ 角落计时             │  - 正文（带高亮）  │
│                         │  - 代码块（真实）  │
├─────────────────────────────────────────────┤
│ 控件: ◀ 上一节  ⏸/▶  下一节 ▶  ↻  + 章节名 │  ← 控件
└─────────────────────────────────────────────┘
```

### DOM 约定
- 每个场景一个 `<svg class="scene" id="scene-N">`，**只有 `.active` 那个显示**
- 每个旁白一个 `<div class="pane" data-pane="N">`，配对切换
- JS 用 `CHAPTERS` 数组驱动一切，每项必须有 `{ id, title, duration, narration }`
  - `narration` = 中文口播稿（**必填**，详见"口播旁白系统"那节）
  - `duration` = TTS 开时是进度条参考；TTS 关时是切换间隔
- 切换 = 切 `.active` + 调用 `runScene(N)` + 触发口播

### 必备控件
- 上一节 / 下一节 / 播放暂停 / 重播本节 / 🔊 旁白开关
- 键盘: ← → / Space / R / **M（静音切换）**
- 章节 chip 可点击直接跳转
- 顶部进度条 + N/总数标签
- 角落 SCENE 编号 + `mm:ss / mm:ss` 计时
- **自动推进由口播结束驱动**，不是固定 setTimeout（详见"口播旁白系统"那节）

---

## 工作流（每次接到任务执行这 6 步，可选第 7 步导出 MP4）

### 步骤 1 · 拆主题为场景

**约束：3–8 个场景**。少于 3 节没必要做动画；多于 8 节注意力分散。

每个场景 = "一个核心概念 + 一个能动的视觉隐喻"。先在脑子里过一遍，写出来：

```
场景 1 · [标题]
  核心概念: [一句话能讲完的那个点]
  视觉隐喻: [打字机 / 飞行的包 / 流水线 / 树 / 时序图 / ...]
  真实数据: [命令输出 / 报文 / 日志 / 耗时数字]
  时长: [6000~9000ms]
```

**好的隐喻是排他的**。"DNS 是电话簿" ✅ 一眼就懂；"DNS 是分布式服务" ❌ 太泛。

### 步骤 2 · 选择视觉模式（按场景类型）

复用 demo 里已有的 5 种基础模式，不要每节都发明新的：

| 类型               | 用途                          | demo 里在哪          |
| ------------------ | ----------------------------- | -------------------- |
| **实物 mockup**    | 起点/终点、用户看得见的东西   | scene 1（浏览器壳）  |
| **节点 + 飞包**    | 多方协调、网络拓扑、消息流转  | scene 2（DNS 递归）  |
| **时序图**         | 两端协议、握手、状态转移      | scene 3/4/5/7（TCP/TLS/HTTP） |
| **流水线 + 日志**  | 内部处理、阶段流转、副作用    | scene 6（服务器内部）|
| **结构树 / 合并**  | 数据结构、编译、解析          | scene 8（DOM/CSSOM） |

需要新隐喻时，先问"能不能用以上 5 种里的 1 种讲清楚"。

### 步骤 3 · 写真实数据，不要编

旁白里的代码块是教学价值最高的部分。**所有命令、报文、日志、时间都要真实**：

- 命令: 真的运行 `dig`、`curl -v`、`tcpdump` 输出
- 报文: 用 HTTP/1.1 真实格式（动词 路径 协议\r\n + Header\r\n + ...）
- 时间: 用合理的 RTT/TTFB 范围（不要写 0ms 或 999ms）
- 日志: 抄真实框架的输出格式（Rails / Nginx / Postgres 各有特征）

虚构的报文/日志读者一眼就知道是 AI 编的，整个 demo 的可信度崩塌。

### 步骤 4 · 写动画函数（每场景一个）

每个 `scene N()` 函数干两件事：
1. **重置元素**到初始状态（`resetEl('id1','id2')`）
2. **按时间序列**调用 `showEl(id, delayMs)` 或自己写 RAF 动画

复杂运动（飞行）用 RAF + cosine easing：

```js
const start = performance.now(); const dur = 2400;
function step(ts){
  const p = Math.min(1, (ts - start) / dur);
  const ease = 0.5 - Math.cos(p * Math.PI) / 2;  // ease-in-out
  el.setAttribute('transform', `translate(${x0 + dx * ease}, ${y0})`);
  if (p < 1) requestAnimationFrame(step);
}
requestAnimationFrame(step);
```

简单显隐用 `showEl(id, delay)`（CSS transition 已挂好）。

**动画总时长**应 ≤ 口播时长（10–20s）。动画演完后画面应"稳"住等口播读完，给观众回看代码的时间 —— 实际推进时机由口播结束驱动，详见下一步和"口播旁白系统"那节。

### 步骤 5 · 写口播稿 + 生成 MP3

给 `CHAPTERS` 每项补 `narration` 字段（口播稿写法见"口播旁白系统"那节）。然后跑一次：

```bash
node scripts/generate-audio.mjs examples/<your-demo>
```

脚本会自动从 HTML 抠出 narration 字段，生成 `audio/scene-N.mp3`（默认云希男声）。重新打开 demo，HTML 会自动检测并使用 mp3 旁白；没生成 mp3 会回退到浏览器原生 Web Speech API。

第一次跑前要装依赖：`npm install`。

### 步骤 6 · 自检（反 AI Slop 清单）

提交前逐项核对：

- [ ] 没有"作为一个 AI 助手" / "希望对你有帮助"这类废话
- [ ] 没有 emoji 泛滥（一节最多 1 个，且作为视觉锚点而不是装饰）
- [ ] 代码块的命令是真的能跑的
- [ ] 时间数字是合理范围（不是占位的 100ms / 1000ms）
- [ ] 没有用 `<div>` 堆出"动画"（必须是真的会动）
- [ ] 每个场景结束时画面"稳"，给读者读完代码的时间
- [ ] 暗色背景不刺眼（不要纯黑 #000，用 #060912 这种）
- [ ] 字体用 Inter + JetBrains Mono，不要 system-ui 凑合
- [ ] 浏览器打开后能键盘 ← → / Space / M 操作
- [ ] 主题专业术语都准确（"三次握手"不写成"三次连接"）
- [ ] `.codeblk` 的 CSS 有 `white-space: pre; tab-size: 2;`（否则代码挤一行）
- [ ] `CHAPTERS` 每项都有 `narration` 字段，没漏节
- [ ] 口播稿是真口语（不会读出"等" "之" "乃"这种书面字）
- [ ] `audio/scene-N.mp3` 已生成
- [ ] 切换章节是按口播结束驱动，不是 `setTimeout(duration)` 硬切
- [ ] 控件区有 🔊 按钮 + M 键绑定

### 步骤 7 · 人工审核 + 导出 MP4（可选）

前 6 步走完、在浏览器里看过完整 demo、觉得满意后，再导出 MP4。这一步是**人工 in-the-loop**：不审核就导出会浪费几分钟跑 Playwright，最终发现某节口播错字或动画卡顿，还得重跑。

```bash
node scripts/export-video.mjs examples/<your-demo>
# 或：npm run video:<demo-name>
```

输出 `<your-demo>/video.mp4`（1280×800 30fps，5MB / 2 分钟左右）。可以直接发微信、抖音、B 站、知乎。

第一次跑前要装浏览器：`npx playwright install chromium`（约 200MB，只装一次）。

实现细节见下方"视频导出"那节。

---

## 设计系统（颜色 / 字体 / 间距）

### 颜色（5 个语义槽，按需选）

| Token       | 值       | 语义                         |
| ----------- | -------- | ---------------------------- |
| `--cyan`    | #22d3ee  | 客户端 / 主线 / "去"的方向   |
| `--violet`  | #a78bfa  | 服务端 / 系统 / "回"的方向   |
| `--lime`    | #a3e635  | 代码字符串 / 成功            |
| `--amber`   | #fbbf24  | 状态码数字 / 中间步骤        |
| `--rose`    | #fb7185  | Paint / 警告 / 危险          |
| `--emerald` | #34d399  | 最终成功 / 200 OK            |

一个场景里**最多用 3 种**主色，其他降到 `--text-dim` / `--muted`。

### 字体（已在 demo `<head>` 引入）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentBuff/animation-explainer](https://github.com/AgentBuff/animation-explainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
