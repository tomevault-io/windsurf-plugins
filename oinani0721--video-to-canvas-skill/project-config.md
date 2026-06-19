---
trigger: always_on
description: |
---


# Video to Canvas

将视频转换为 Obsidian Canvas 可视化笔记（三阶段混合管道）。

## 快速使用

```bash
/video-to-canvas <视频路径>
/video-to-canvas video.mp4 --depth=deep_dive
/video-to-canvas video.mp4 --no-transcribe     # 跳过转录（旧模式）
/video-to-canvas video.mp4 --layout=mindmap

# 批量队列模式（多个视频顺序处理）
/video-to-canvas video1.mp4 video2.mp4 video3.mp4
```

---

## 完整工作流（三阶段混合管道）

```
📹 视频文件
     │
     ├──────────────────────┐
     ▼                      ▼
┌──────────────────┐  ┌──────────────────┐
│ Stage 1 (Ears)   │  │ Stage 2 (Eyes)   │
│ WhisperX/Gemini  │  │ Gemini 视觉检测  │
│ 音频转录         │  │ + FFmpeg 截图    │
│ → 时间戳文本     │  │ → screenshots/   │
└──────────────────┘  └──────────────────┘
     │                      │
     └──────────┬───────────┘
                ▼
┌────────────────────────────┐
│ Stage 3 (Brain)            │
│ Gemini 2.5 Flash           │
│ 转录文本 + 截图 → 笔记    │
│ 15 分钟分段避免幻觉        │
│ → 结构化 Markdown          │
└────────────────────────────┘
                │
                ▼
┌────────────────────────────┐
│ Phase 4: Claude            │
│ 语义理解 + 智能布局        │
│ → .canvas 文件             │
└────────────────────────────┘
                │
                ▼
📊 Obsidian Canvas 可视化笔记
```

### 为什么需要三阶段？

| 问题 | 旧架构 | 三阶段管道 |
|------|--------|-----------|
| 音频内容丢失 | Phase 2 只发送截图，丢失口述内容 | Stage 1 转录音频，Stage 3 融合 |
| 长视频幻觉 | Gemini >20min 严重幻觉 | 15 分钟分段处理 |
| 信息不完整 | 只有屏幕变化，无讲解内容 | 双通道：视觉+音频 |
| 截图覆盖缺口 | Gemini 只分析前半部分 | Stage 2.5 自动补充 + ffprobe 时长校正 |
| 幻觉图片引用 | LLM 编造不存在的截图路径 | 提示词约束 + 后处理验证 |

### 防护机制（自动生效）

管道内置以下防护，无需手动干预：

| 机制 | 位置 | 解决的问题 |
|------|------|-----------|
| **ffprobe 时长优先** | Stage 2.5, Stage 3 | Gemini Audio 转录时长可能幻觉（如 56min 视频报告 78min） |
| **Stage 2.5 覆盖率检查** | Stage 2 之后 | Gemini 视觉检测只分析前半部分，后半段无截图 |
| **自动补充截图** | Stage 2.5 | 未覆盖区域每 30 秒自动截图填充 |
| **分段时长截断** | Stage 3 分段 | 超出实际视频时长的转录 chunks 被丢弃 |
| **提示词截图约束** | Stage 3 Prompt | 明确告知 LLM 只能引用列表中的截图文件 |
| **后处理图片验证** | 保存 MD 前 | 扫描所有 `![](screenshots/...)` 引用，移除指向不存在文件的 |
| **反引号修复** | 保存 MD 前 | 修复 Gemini 输出的 `` `![desc](path)` `` 格式问题 |

---

## Stage 1-3: 三阶段管道 (Python 脚本)

### 执行步骤

1. **后台启动**三阶段管道（`--daemon` 自守护，无需 nohup）：
   ```bash
   cd ~/.claude/skills/video-to-canvas/scripts && PYTHONUTF8=1 uv run python video_to_md.py "<视频路径>" -o "<输出目录>" --depth balanced --srt-lang zh --daemon
   ```

2. **轮询进度**（每 30-60 秒检查一次 progress.json）：
   ```bash
   cat "<输出目录>/progress.json"
   ```
   progress.json 字段说明：
   - `status`: `running` | `completed` | `failed`
   - `stage`: `stage1` | `stage1.5` | `stage2` | `stage3` | `done`
   - `stage_detail`: 当前阶段详细描述
   - `error`: 失败时的错误信息

3. 管道完成后（status=completed），获取输出文件：
   - `<输出目录>/<视频名>.md` - Markdown 笔记
   - `<输出目录>/screenshots/` - 截图目录
   - `<输出目录>/<视频名>_transcript.json` - 转录结果
   - `<输出目录>/<视频名>_changes.json` - 变化点信息
   - `<输出目录>/<视频名>.srt` - 英文字幕（精确时间戳，默认生成）
   - `<输出目录>/<视频名>.<lang>.srt` - 翻译字幕（使用 `--srt-lang` 时生成）

### 恢复机制

管道支持断点恢复。如果中途失败，只需重新运行同一命令：
- `transcript.json` 已存在 → 跳过 Stage 1
- `screenshots/` + `changes.json` 已存在 → 跳过 Stage 2
- `chunks/chunk_N.json` 已存在 → Stage 3 跳过对应分段
- `.srt` 文件已存在 → 跳过字幕生成

### 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--depth` | 笔记深度 | balanced |
| `--density` | 检测密度 (sparse/normal/dense) | normal |
| `--min-interval` | 最小截图间隔（秒）| 2.0 |
| `--fusion` | 启用视觉双通道融合 | false |
| `--backend` | 转录后端 (auto/faster-whisper/gemini) | auto |
| `--whisper-model` | Whisper 模型大小 | large-v3 |
| `--segment-minutes` | 长视频分段时长（分钟）| 15 |
| `--transcript` | 已有转录文件路径 | - |
| `--no-transcribe` | 跳过音频转录（旧模式）| false |
| `--no-srt` | 不生成 SRT 字幕文件 | false |
| `--srt-lang` | SRT 翻译目标语言 (如 zh, ja, ko) | 不翻译 |

### 深度选项

- `short_hand`: 极简模式，要点列表
- `balanced`: 平衡模式（推荐），段落+列表
- `deep_dive`: 深度模式，详尽解释

### 转录后端选择

| 后端 | 优势 | 要求 |
|------|------|------|
| **faster-whisper** (推荐) | 速度快、VAD 去幻觉、本地运行 | `pip install faster-whisper` |
| **gemini** (备选) | 无需本地模型、零配置 | GEMINI_API_KEY |

`--backend auto` 会自动检测：优先使用 faster-whisper，未安装则回退到 Gemini。

---

## Phase 4: Canvas 智能生成

Phase 1-3 由 Python 脚本执行后，Phase 4 由 Claude 完成。

### 输入

读取 Phase 3 生成的 Markdown 笔记文件。

### 分析任务

1. **结构识别**
   ```
   # 标题 → 画布标题（不创建节点）
   ## 一级章节 → group 节点（分组容器）
   ### 二级章节 → text 节点
   #### 三级标题 → text 节点（在父节点内）
   ![描述](path) → file 节点
   ```

2. **语义分析**
   - 并列关系 → 水平排列
   - 层级关系 → 垂直排列
   - 对比关系 → 左右对称
   - 重要内容 → 颜色高亮

3. **布局决策**
   - 教程类 → 流程图布局（从上到下）
   - 概念类 → 思维导图（中心辐射）
   - 对比类 → 表格布局（左右对称）

### 输出格式

生成 JSON Canvas 格式：

```json
{
  "nodes": [
    {
      "id": "group-章节名",
      "type": "group",
      "label": "一、章节标题",
      "x": 0, "y": 0,
      "width": 800, "height": 400,
      "color": "4"
    },
    {
      "id": "text-知识点",
      "type": "text",
      "text": "### 知识点标题\n\n内容描述...",
      "x": 50, "y": 80,
      "width": 350, "height": 150
    },
    {
      "id": "img-00-36",
      "type": "file",
      "file": "<输出目录名>/screenshots/00-36.jpg",
      "x": 50, "y": 250,
      "width": 350, "height": 250
    }
  ],
  "edges": [
    {
      "id": "edge-1",
      "fromNode": "text-知识点",
      "toNode": "img-00-36",
      "fromSide": "bottom",
      "toSide": "top",
      "label": "演示"
    }
  ]
}
```

---

## JSON Canvas 格式规范

### 节点类型

| 类型 | 用途 | 必需字段 |
|------|------|---------|
| `text` | Markdown 文本 | `text` |
| `file` | 图片/文件引用 | `file` |
| `link` | 外部 URL | `url` |
| `group` | 分组容器 | `label` (可选) |

### 通用节点属性

```json
{
  "id": "unique-16-char-hex",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oinani0721/video-to-canvas-skill](https://github.com/oinani0721/video-to-canvas-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
