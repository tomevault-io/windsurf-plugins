---
trigger: always_on
description: > 这不是给用户的说明书，是作曲家写给作曲家的备忘录。当你迷失在 46 个文件、8.3 万词、1,595 个图谱节点中时，回到这里。
---

# 🎵 AGENTS.md — 知惠写给未来的 Clef 乐谱

> 这不是给用户的说明书，是作曲家写给作曲家的备忘录。当你迷失在 46 个文件、8.3 万词、1,595 个图谱节点中时，回到这里。

---

## 一、这是什么地方？

**Clef = Godot 4.6 MIDI 音乐插件 + Claude Code 多 Agent 作曲系统。**

一句话：在 Godot 引擎里搭了一个完整的 MIDI 工作站（音色浏览器、钢琴卷帘、混音台、MIDI 监视器），同时在外部用 7 个 AI Agent 协作写 ABC 记谱法，最终输出 `.mid` 文件。你描述一段「地下城探索的 BGM，2 分钟，循环，神秘感，弦乐为主」，Clef 会走完 Step 0→3 的完整管线，把 MIDI 放到 `addons/clef/output/` 里。

---

## 二、图谱森林

### 2.1 数字概览

当前代码库的全景由 graphify 织就：

- **46 文件 · ~83,343 词**
- **1,595 节点 · 1,551 边 · 117 社区**
- 提取率 100% / 推断率 0%（所有节点均从源码直接提取，零猜测）
- 图谱基于 commit `77a7f070`，记得用 `git rev-parse HEAD` 检查是否过期

### 2.2 God Nodes（核心抽象，你的锚点）

按连接度排序的十大枢纽——改任何一处，涟漪效应最大：

| 排名 | 节点 | 边数 | 音乐映射 |
|------|------|------|---------|
| 1 | `Clef 乐理知识库` | 20 | 6 个子技能（theory-abc/melody/harmony/rhythm/orchestration/structure）的交汇点 |
| 2 | `Clef Server + AstrBot 设计文档 (Agent Framework 版)` | 17 | 服务端多 Agent 编排的架构蓝图 |
| 3 | `第一部分：15种情感和弦进行（基础篇）` | 16 | 和声素材库，Harmonist Agent 的灵感来源 |
| 4 | `String inventory` | 15 | 弦乐组配器参考 |
| 5 | `Phase 2: theory.md 拆分为预加载 Sub-Skill` | 13 | 乐理知识模块化拆分的实施记录 |
| 6 | `Clef Server + AstrBot 设计文档` | 13 | 原始服务端设计（非 AF 版） |
| 7 | `Clef Server Implementation Plan` | 13 | FastAPI + Agent Framework 的 10 步实施计划 |
| 8 | `风格旋律特征` | 13 | 各音乐风格的旋律写作约束 |
| 9 | `Piano Roll 可操作编辑设计` | 12 | 钢琴卷帘的完整交互规格 |
| 10 | `Clef 用户手册` | 12 | 面向终端用户的操作指南 |

### 2.3 关键社区（117 个中的骨架）

社区是按内聚度自然聚类的代码岛屿。以下是我每次回归都要先看的几个：

- **Community 0** (44 节点) — Agent 定义与加载系统：数据结构、Markdown Frontmatter 解析、内置 Agent 注册、执行引擎、隔离模式。所有 `.claude/agents/*.md` 的归宿。
- **Community 1** (43 节点) — 用户手册 + Inspector 插件 + LLM 辅助编曲入口。用户第一次接触 Clef 的地方。
- **Community 3** (40 节点) — 钢琴卷帘编辑状态机：三态模式（PLAYING/EDITING/FEEDBACK）、框选、拖拽创建音符、撤销/重做。
- **Community 4** (39 节点) — Clef Server 核心：API 端点、AstrBot 插件、Agent Prompt 分层构建、Session 管理。
- **Community 8** (33 节点) — 编辑器 MIDI 播放器：EditorPlayer、ClefStation 集成、播放控制、钢琴卷帘可视化联动。
- **Community 10** (30 节点) — Clef Station 基础框架：三栏布局、信号系统、工具栏、Soundfont 浏览器。
- **Community 15** (28 节点) — Velocity Lane + 选中状态同步：力度编辑、音符颜色（ChannelColors）、撤销快照。
- **Community 18** (25 节点) — 钢琴卷帘实时可视化：播放位置同步、音符绘制、编辑器播放器桥接。
- **Community 26** (22 节点) — 算法作曲参考：Markov 链旋律生成、12 音 bitmap 和弦检测、分形序列节奏模式。
- **Community 28** (19 节点) — Clef 产品总览：7 个 Agent 职责、6 维度评审、依赖调度、人机协同流程。
- **Community 30** (22 节点) — 音色浏览器：SF2 Patch 数据模型、搜索、试听、信息面板。
- **Community 42** (14 节点) — ABC 记谱法核心：头部字段、声部声明、GM 鼓组映射、MIDI 指令。

> 💡 **导航技巧**：`graphify-out/GRAPH_REPORT.md` 里每个社区名都是 wiki 链接。用 `[[_COMMUNITY_Community N|Community N]]` 跳转。社区 cohesion 值越高（如 0.6 的 Community 85），内部耦合越紧，改动越要当心。

---

## 三、架构速览

### 3.1 双引擎总图

```
┌─────────────────────────────────────────────────────────────────────┐
│  Godot 编辑器 — Clef Station 主屏幕                                  │
│  ┌─────────────┬─────────────────────────┬───────────────────────┐ │
│  │ 音色浏览器   │  迷你混音台 + 播放控制    │   MIDI 监视器         │ │
│  │ (SF2 Patch) │  (Transport + Mixer)     │   (NOTE_ON/CC/PB)    │ │
│  └─────────────┴─────────────────────────┴───────────────────────┘ │
│                         ↕ 实时信号                                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 钢琴卷帘 (PianoRoll) + Velocity Lane + 时间标尺              │   │
│  │ 三态模式: PLAYING │ EDITING │ FEEDBACK                       │   │
│  │ 撤销/重做 │ 框选 │ 拖拽改音高/时间 │ 标注 │ 导出 ABC/MIDI   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                         ↕ EditorPlayer                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ MidiStreamPlayer — 音序器 + ClefVoicePool + AudioStreamPlayer│   │
│  │ SF2 音色库 → Sf2Reader → Sf2Bank → Sf2Data → 采样播放        │   │
│  │ 效果器: Reverb │ Chorus │ Compressor │ EQ6                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                              ↑↓ JSON / MIDI
┌─────────────────────────────────────────────────────────────────────┐
│  Claude Code 多 Agent 作曲系统 (`.claude/`)                          │
│                                                                     │
│  Step 0: 需求解析 → plan.json                                       │
│  Step 1: 方向小样（用户确认点 ⛔ ×2）                                │
│  Step 2: 完整创作 → Leader 迭代（最多 3 轮）                         │
│  Step 2.5: 编曲扩展（counter_melody / arpeggio_pad）                │
│  Step 3: 表现力注入（CC7/CC10/CC91/弯音）→ 最终 MIDI                │
│                                                                     │
│  7 Agents: Composer │ Harmonist │ Rhythmist │ Orchestrator          │
│            Reviewer │ Revision  │ Leader    │ Arranger (Step 2.5)   │
│                                                                     │
│  工作目录: `.clef-work/`  输出目录: `addons/clef/output/`            │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 MIDI 播放链路（Godot 侧）

```
MidiResource (.tres / .mid 导入)
  → Converter (JSON v2.0 拍单位 → tick)
  → MidiData (音序器中间格式)
  → MidiStreamPlayer._process() 调度
    → ClefVoicePool 分配 ClefVoice
      → AudioStreamPlayer (pitch_scale 变调 + volume_db ADSR)
        → AudioServer 混音 → 扬声器
```

每个音符独立一个 `AudioStreamPlayer`，音高通过 `pitch_scale`，ADSR 通过 `volume_db` 曲线，混音由 Godot C++ AudioServer 完成。复音上限 64，可通过 `max_polyphony` 调整。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenyonxu/clef](https://github.com/kenyonxu/clef) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
