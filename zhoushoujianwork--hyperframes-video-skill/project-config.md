---
trigger: always_on
description: Turn a project/tool/engineering story into a short, vivid product video for an everyday-developer audience and help publish it to Bilibili. Story-driven scriptwriting, voice-cloning voiceover (MMX/MiniMax, falls back to system TTS), HyperFrames + GSAP animation (DrawSVG/MotionPath/kinetic captions), cover + metadata, browser-assisted upload, collection management.
---


# Bilibili Product Video

Use this skill when the user wants to turn a project, tool, demo, or engineering story into a short video for a general/developer audience, especially when they mention product introduction, B站投稿, 合集, 封面, 配音, 音色克隆, MMX, HyperFrames, or using their own voice.

> **自更新（每次使用前先做）**：本 skill 由 git 仓库管理。开始任何工作前先拉最新：`bash "$SKILL_DIR/scripts/update.sh"`（等价于 `git -C "$SKILL_DIR" pull --ff-only --quiet`）。`$SKILL_DIR` = 本 SKILL.md 所在目录。claude 与 codex 两端通过 symlink 指向同一 git 克隆，拉一次两端同步。
>
> **开源依赖**：HyperFrames（Apache-2.0，`npx hyperframes`）做渲染。MMX 是私有音色克隆 CLI，**可选**；没有它时配音自动回退系统 TTS（见 §4）。

## Default Goal

Produce a complete first-pass video package and help the user publish it:

- Rendered video file.
- Cover image.
- Voiceover audio and subtitles.
- Bilibili title, description, tags, and collection plan.
- Browser-assisted upload, with explicit user confirmation before irreversible publishing actions unless the user already clearly authorized them.

## Inputs To Collect

Prefer using existing context before asking questions. Only ask if the missing detail changes the output materially.

- Project name, repo path, public URL, and open-source URL.
- Audience tier (默认 **普通大众程序员**): 他们知道 AI / Claude / 终端是什么、自己会写代码，但**不知道你项目的内部架构**（Adapter、driver 接口、协议、ASR/TTS 链路）。讲东西用「结果 + 类比」，不讲内部行话。比"非专业大众"高一档、比"硬核极客"低一档。
- Core scenario: the real pain point, who had the problem, and why the tool matters.
- Voice (音色克隆): 用 `scripts/voice.sh` 生成配音——配了 MMX/MiniMax 克隆声就用克隆声（声音 ID 走环境变量 `MMX_VOICE_ID`），没配就**回退系统 TTS**（macOS `say` / Linux `espeak`）。见下方「Generate Voiceover」。
- Preferred 合集/系列 name (可选): 同一系列的视频归到一个合集；名字由用户提供，无则不强求。
- Episode format: 常规故事 / 对比评测 / 翻车踩坑（见下方「形态轮换」）。

## 创作标准（核心 — 2026-06 升级，优先于下方模板）

来自前几期的复盘：早期片子**内容单调（每个场景同布局）、脚本是功能罗列、太钻（讲内部架构）、动画不形象**。以下四条是新标准，每期必守。标杆参照：B站「工科男孙老师」（故事壳 + 真实物当锚 + 逐段高亮 + 信号流框图 + 章节条 + 人格化吐槽）、「爱上半导体」（自绘概念动画把抽象变直观 + 大字幕 + "X vs Y 区别"对比形态，单条 98 万播放）。

### ① 受众下沉：删内部行话
- 目标受众 = **普通大众程序员**（见 Inputs）。**禁止**出现这些内部词：Adapter、driver 接口、router.Register、active_driver、协议名、ASR/TTS 链路、session 等。
- 一律换成**结果或类比**：不说「driver matrix 热切」，说「想换哪个 AI 就换哪个」；不说「本地 ASR/TTS」，说「连你说的话都不出这台电脑」。
- curl/终端可以露（程序员爱看真东西），但**作为"证据一闪"**，不逐字段讲解。

### ② 故事化脚本骨架（默认形态）
不要功能罗列。每期一条**故事弧**，且有"我"这个角色和真实赌注：
```
钩子(5s 痛点/反差) → 我为什么做这个(我遇到的具体麻烦) → 试 / 翻车(过程有波折) → 啊哈时刻(它怎么解决) → 一句话怎么用 → 开源/CTA
```
- 钩子库（挑一个，不平铺）：「我受够了X，于是…」「都说Y很难，其实…」「同事/老板说Z，我顺手做了…」「有手机/有现成的，为什么还要这个？」
- 多用第一人称、具体场景、有情绪起伏；**少用"它支持…它可以…"的清单句**。

### ③ 形态轮换（破单调）
不要每期都同一种片子。按节奏轮换：
| 形态 | 何时用 | 公式 |
|---|---|---|
| 常规故事 | 默认 | 上面的故事弧 |
| **对比评测** | 每 3 期插 1 期 | 「X vs Y」「为什么用X不用Y」——云AI vs 本地AI、智能助手 vs 自己搓、一堆工具 vs 一个入口。对比形态天然高播放。 |
| 翻车踩坑 | 有真实事故时 | 「我把X搞炸了」→ 复盘 → 修好。真实、好笑、共鸣。 |

### ④ 形象动画 + 布局轮换（破抽象、破单调）
- **禁止**整片都用「左文字栈 + 右面板」同一布局。每 2-3 个场景**换一种构图**：满屏大字、居中单图、左右对调、上下分层、纯动画特写。
- 至少有**一个高潮 demo beat**（信息密度最高、最出彩的一幕），别让每幕权重一样。
- **隐喻优先于文字面板**——把抽象概念变成能看见的东西。逐期积累「隐喻库」：
  | 概念 | 形象隐喻 |
  |---|---|
  | AI 后端可切换 | 可插拔**卡带 / 转接头**，咔哒换一个 |
  | 本地 vs 云 | 数据**出不出家门**（云=飞出窗外，本地=锁在屋里）|
  | 厂商锁死 | **上锁的笼子 / 焊死的盖子** |
  | 适配层 | **万能转接头 / 翻译官** |
  | 自绘概念图 | 像爱上半导体那样让图表/连线**自己画出来**，配大字幕 |
- **隐喻生成法（每期现造，别照搬旧图）**——抽象概念卡壳时按这三步走，比查表更可靠（借鉴自 [ian-xiaohei-illustrations](https://github.com/helloianneo/ian-xiaohei-illustrations) 的方法论，MIT）：
  1. **概念→物理动作**：卡住 / 漏掉 / 变重 / 分拣 / 沉淀 / 发酵 / 开门 / 折叠 / 拆包 / 回流。
  2. **结构→低科技物件**：坏机器 / 纸箱 / 抽屉 / 漏斗 / 秤 / 邮筒 / 门 / 井 / 梯子 / 水管 / 线团 / 闸门 / 转盘 / 黑盒 / 压面机。一次只用 1-2 个，别堆满。
  3. **让角色承担动作**：不是站旁边解说，而是卡在机器里、拉错线、守门、搬运、修补、称重——动作服务核心意思，别为怪而怪。
- **固定 IP 角色（系列连续性）**——指定一个贯穿全系列的吉祥物（默认沿用上面隐喻法里的执行者），每期都让它做核心动作而非装饰；表情克制、有点呆、不卖萌。用 SVG/CSS 自绘成可被 GSAP 驱动的矢量件（这样能 DrawSVG 自绘 / MotionPath 运动），对标「工科男孙老师」的人格化吐槽，给观众记忆锚点。具体形象一旦定稿写进本节固化。
- **构图模式库（配合「每 2-3 幕换构图」）**——一幕只用一种结构，别混：**Workflow**（左输入→中处理→右输出，橙箭头主流向）/ **系统局部**（只画 3-5 个核心模块，角色参与其一）/ **前后对比**（左乱右稳，中间箭头）/ **角色状态**（2-4 个小状态各配短标注）/ **概念隐喻**（一个大怪物件，少量输入一个输出）/ **方法分层**（一层层盒子，角色在旁搭建）/ **地图路线**（一条弯路径少量节点）/ **小漫画分镜**（2-4 格，每格一个动作）。
- 已验证好用的视觉件（沿用）：底部**章节进度条**（随播放高亮，留存利器）、**动画信号流**（节点间流动小点 + active 项发光）、真机实拍当证据、点阵/Nothing 风格。

## Workflow

### 1. Understand The Product

Read the project README, docs, app screenshots, and public site when available. Extract:

- One-sentence plain-language value proposition.
- The before/after story.
- The key demo moments worth showing.
- Terms that should be simplified for non-specialists.

For Agent Room-like projects, frame it as:

- A tool made to solve coworkers' Windows environment/software installation troubleshooting.
- Slave mode lets a user's machine without local AI be coordinated by the creator's AI.
- Multiple AI agents can enter a room and talk through a problem like a meeting or voice room.
- The hook is practical: "同事电脑没 AI，也能让我的 AI 帮他排查问题".

### 2. Create A Video Folder

Create a dedicated folder under `videos/`:

```text
videos/episode-NN-topic-bilibili/
├── assets/
├── preview-frames/
├── design.md
├── narration.md
├── narration.txt
├── index.html
├── publish-draft.md
├── cover-*.png
└── *.mp4
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhoushoujianwork/hyperframes-video-skill](https://github.com/zhoushoujianwork/hyperframes-video-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
