---
trigger: always_on
description: 基于 Electron + React 的桌面应用，用于抖音视频采集、转录、AI 洗稿、Skills 蒸馏和本地竖屏视频生成。当前视频生成通过 HyperFrames CLI 本地渲染 HTML/CSS/GSAP 成 MP4。
---

# 抖创工坊

基于 Electron + React 的桌面应用，用于抖音视频采集、转录、AI 洗稿、Skills 蒸馏和本地竖屏视频生成。当前视频生成通过 HyperFrames CLI 本地渲染 HTML/CSS/GSAP 成 MP4。

## 项目架构

```
douyin/
├── src/                      # 后端服务（Node.js + Express）
│   ├── app.ts               # Express 应用配置
│   ├── server.ts            # 独立 HTTP 服务器入口
│   ├── lib/                 # 核心业务逻辑
│   │   ├── jobs.ts          # 任务管理器、手动步骤、垃圾桶
│   │   ├── ai-cleaner.ts    # AI 洗稿
│   │   ├── storage.ts       # 文件存储
│   │   ├── media.ts         # 视频下载、音频提取
│   │   ├── asr.ts           # 语音识别（内置 whisper.cpp）
│   │   └── hyperframes-video.ts # HyperFrames 本地视频渲染
│   └── types.ts             # 后端类型定义
│
├── renderer/                 # 前端界面（React + Vite）
│   ├── src/
│   │   ├── App.tsx
│   │   ├── pages/
│   │   │   ├── JobListPage.tsx
│   │   │   ├── JobDetailPage.tsx
│   │   │   ├── TrashPage.tsx
│   │   │   └── SettingsPage.tsx
│   │   ├── components/
│   │   ├── services/api.ts
│   │   └── types/index.ts
│   └── vite.config.ts
│
├── electron/                 # Electron 主进程与配置 IPC
└── dist/                     # 后端编译输出
```

## 技术栈

### 后端
- Node.js 18+、Express 4、TypeScript
- `openai`：OpenAI-compatible AI 洗稿
- `yt-dlp`：视频下载（外部二进制）
- `ffmpeg` / `ffprobe`：音视频处理（外部二进制）
- whisper.cpp：内置本地 ASR，默认 `ggml-small` 多语言模型
- HyperFrames CLI：本地竖屏 MP4 渲染（生成视频步骤需要 Node.js 22+ 和 FFmpeg）

### 前端
- React 19、Vite、React Router DOM 7、Zustand、Tailwind CSS、Axios

### 桌面端
- Electron 34、electron-builder

## 核心流程

### 手动分步主链路

```
用户输入（URL 或分享文本）
    ↓
POST /api/jobs 创建任务并解析输入
    ↓
用户在详情页逐步确认执行：
    1. 视频转录（yt-dlp + ffmpeg + 内置 whisper.cpp）
    2. AI 洗稿
    3. 生成分镜
    4. 生成 9:16 MP4（HyperFrames）
```

每个步骤独立执行。用户点击某一步后，后端在同一次请求内自动重试最多 3 次；失败后停在当前步骤，用户可手动重试。后一步必须等前一步成功后才能执行。

### 数据存储

默认目录：`~/Documents/抖音AI视频/`

```
抖音AI视频/
├── raw/
│   ├── videos/              # 下载的视频
│   ├── audio/               # 提取音频和 manifest
│   ├── transcripts/         # 结构化转录 JSON
│   ├── page/                # 页面元数据
│   └── text/                # 分享文本解析结果
├── processed/
│   ├── scripts/             # 脚本资产
│   ├── cleaned/             # AI 清洗结果
│   ├── scenes/              # 历史场景数据
│   └── subtitles/           # 字幕文件
├── output/
│   └── videos/              # HyperFrames 项目和 MP4 输出
└── logs/
```

### 任务状态与步骤

```typescript
type JobStatus = "queued" | "processing" | "done" | "failed";

type JobStage =
  | "submitted"
  | "parsed"
  | "downloading"
  | "downloaded"
  | "extracting"
  | "audio_extracted"
  | "transcribing"
  | "transcribed"
  | "cleaning"
  | "cleaned"
  | "generating-video-prompts"
  | "scripted"
  | "generating-video"
  | "rendered"
  | "failed";

type WorkflowMode = "manual" | "auto";
type PipelineStep = "transcribe" | "clean" | "generate_video_prompts" | "generate_video";
type PipelineStepStatus = "pending" | "running" | "succeeded" | "failed";
```

## API 接口

### 任务管理
- `POST /api/jobs` - 创建任务
- `GET /api/jobs` - 获取未删除任务列表
- `GET /api/jobs/:id` - 获取任务详情
- `DELETE /api/jobs/:id` - 软删除任务到垃圾桶
- `GET /api/jobs/trash` - 获取垃圾桶任务并触发过期清理
- `POST /api/jobs/:id/restore` - 恢复垃圾桶任务
- `DELETE /api/jobs/:id/permanent` - 永久删除垃圾桶任务及关联文件

### 手动步骤
- `POST /api/jobs/:id/steps/transcribe`
- `POST /api/jobs/:id/steps/clean`
- `POST /api/jobs/:id/steps/generate-video-prompts`
- `POST /api/jobs/:id/steps/generate-video`

### 内容获取
- `GET /api/jobs/:id/script` - 历史脚本资产
- `GET /api/jobs/:id/cleaned` - AI 清洗结果
- `GET /api/jobs/:id/raw-transcript` - 结构化原始转录
- `GET /api/jobs/:id/video-prompts` - 分镜（兼容历史视频提示词字段）
- `GET /api/jobs/:id/video-output` - HyperFrames 视频输出信息
- `GET /api/jobs/:id/video/download` - 下载 MP4

## 关键数据结构

### JobRecord

```typescript
{
  id: string;
  sourceUrl: string;
  topic: string;
  status: JobStatus;
  stage: JobStage;
  workflowMode?: WorkflowMode;
  steps?: Record<PipelineStep, PipelineStepState>;
  deletedAt?: string;
  trashExpiresAt?: string;
  videoPath?: string;
  audioPath?: string;
  audioManifestPath?: string;
  transcriptPath?: string;
  transcriptModel?: string;
  videoProjectPath?: string;
  videoOutputPath?: string;
  videoGeneratedAt?: string;
  storagePath: string;
  createdAt: string;
  updatedAt: string;
}
```

### TranscriptAsset

```typescript
{
  jobId: string;
  sourceUrl: string;
  audioPath: string;
  transcript: string;
  text: string;
  segments: Array<{ start?: number; end?: number; text: string }>;
  words?: Array<{ start?: number; end?: number; word: string; probability?: number }>;
  duration?: number;
  language?: string;
  model: string;
  provider: string; // "whisper.cpp"
  createdAt: string;
}
```

### CleanedScript.output

```typescript
{
  title?: string;
  rawText?: string;
  summary?: string;
  keyPoints?: string[];
  cleanScript?: string;
  voiceoverScript?: string;
  videoOutline?: Array<{ title: string; bullets: string[]; visualPrompt?: string }>;
  videoPrompts?: string[];
  enhancedScenes?: any[];
  hyperframesVideo?: {
    provider: "hyperframes";
    projectPath: string;
    videoPath: string;
    manifestPath: string;
    duration: number;
    aspectRatio: "9:16";
    width: 1080;
    height: 1920;
  };
  qualityNotes?: string[];
  tags?: string[];
}
```

## 配置管理

配置文件位置：`~/.douyin-ai-video/config.json`

### AI 配置

```json
{
  "aiKeys": [
    {
      "id": "uuid",
      "name": "DeepSeek",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiChangZheng10086/doyin_ai_video](https://github.com/LiChangZheng10086/doyin_ai_video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
