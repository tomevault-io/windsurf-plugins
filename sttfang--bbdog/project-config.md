---
trigger: always_on
description: - **洗稿**（launder）：复刻已有短剧/剧本，AI 改写 + 重新生成资产
---

# bbdog — AI 短剧制作工具链（Agent-First CLI）

## 项目定位

AI 短剧制作工具链，支持三种模式：
- **洗稿**（launder）：复刻已有短剧/剧本，AI 改写 + 重新生成资产
- **原创**（original）：从一句话 logline 生成完整剧本
- **改编**（adapt）：小说/长文改编为短剧剧本

设计理念：**Agent-First**，CLI 输出结构化 JSON + nextStep 引导，Claude Code 作为 UI 层驱动完整流程。

## CLI 命令结构（扁平化）

```bash
# 初始化 & 配置
bbdog init                          # 初始化项目目录
bbdog config set-key <key>          # 设置 API key
bbdog config use <type> <model>     # 切换模型
bbdog config models                 # 查看可用模型
bbdog config show                   # 查看当前配置
bbdog status                        # 项目状态摘要（含 nextStep 建议）

# 剧本生成
bbdog create <logline>              # 一句话原创剧本
bbdog adapt <file>                  # 小说/长文改编为短剧剧本
bbdog parse <file>                  # 解析剧本为结构化数据（角色/场景/美术指导/视觉提示词）
bbdog shots                         # 基于剧本生成分镜列表

# 资产生成（gen 命令组）
bbdog gen characters [--id X]       # 生成角色参考图
bbdog gen scenes [--id X]           # 生成场景参考图
bbdog gen keyframes [--shot X]      # 生成关键帧（起始帧/结束帧）
bbdog gen videos [--shot X]         # 生成视频片段
bbdog gen tts [--shot X]            # TTS 语音合成
bbdog gen subtitles                 # 生成字幕文件（SRT/VTT）

# 设计（独立重跑）
bbdog design art                    # （重新）生成美术指导
bbdog design characters [--id X]    # （重新）生成角色视觉设计
bbdog design scenes [--id X]        # （重新）生成场景视觉设计

# 全局资产库
bbdog asset push --type <t> --id X  # 推送角色/场景到全局库
bbdog asset pull --type <t> --gid X # 从全局库导入
bbdog asset list [--type <t>]       # 列出全局资产

# 后期 & 导出
bbdog render                        # AI 智能剪辑 + ffmpeg 渲染成片
bbdog pipeline <file>               # 一键执行完整流程（DAG 编排）
bbdog episode split                 # 按 cliffhanger 自动分集
bbdog export project                # 导出项目包
```

所有命令支持 `--json` 全局选项，输出 `{ ok, data, nextStep }` 结构供 Agent 消费。

## 制作流程

```
init → create/adapt/parse → shots → gen characters → gen scenes → gen keyframes → gen videos → gen tts → gen subtitles → render → export
```

每步 `nextStep` 引导 Agent 自动执行下一步。`bbdog status --json` 可随时获取当前进度和建议。

## 技术栈

- Monorepo：`packages/bbdog` CLI
- CLI 框架：TypeScript + Commander + Chalk
- 媒体处理：fluent-ffmpeg, sharp, archiver
- AI 提供商：antsk（OpenAI 兼容）、Google、Kling、本地 Claude CLI
- 包管理：npm workspaces
- 构建：`tsc`
- 存储：JSON 文件（项目状态 `.bbdog/project.json`）

## 模型配置

运行时配置存储在 `~/.bbdog/config.json`。

| 用途 | 默认模型 | 说明 |
|------|----------|------|
| 文本（chat） | `claude-cli`（本地） | 通过 Claude Code CLI 调用，免费无需 API key |
| 图片（image） | `gemini-3-pro-image-preview` | 通过 antsk API |
| 视频（video） | `sora-2` | 通过 antsk API |

API key 优先级：模型级 → 提供商级 → 全局 → `process.env.API_KEY`

## 关键文件路径

```
packages/bbdog/
├── bin/bbdog.ts                         # CLI 入口
├── src/cli.ts                           # Commander 命令注册（Agent-First 扁平化）
├── src/commands/
│   ├── flat-aliases.ts                  # 扁平化顶层命令（parse/shots/create/adapt/render）
│   ├── gen.ts                           # gen 命令组（characters/scenes/keyframes/videos/tts/subtitles）
│   ├── init.ts                          # 项目初始化
│   ├── config.ts                        # 配置管理
│   ├── status.ts                        # 项目状态
│   ├── pipeline.ts                      # DAG 一键流程
│   ├── episode.ts                       # 分集
│   ├── export.ts                        # 导出
│   ├── design.ts                        # design 命令组（art/characters/scenes）
│   ├── asset.ts                         # 全局资产库（push/pull/list）
│   ├── direct.ts                        # 关键帧/视频底层实现
│   ├── audio.ts                         # TTS 底层实现
│   └── subtitle.ts                      # 字幕底层实现
├── src/core/
│   ├── api-client.ts                    # HTTP 客户端、重试、Claude CLI 后端
│   ├── workspace.ts                     # 工作区上下文（统一加载项目状态）
│   ├── output.ts                        # JSON/文本双模式输出 + nextStep 协议
│   ├── errors.ts                        # 错误码定义
│   ├── dag.ts                           # DAG 编排引擎
│   ├── prompt-loader.ts                # Prompt 模板加载器（lib/prompts/*.txt）
│   ├── pipeline-steps.ts               # Pipeline 步骤定义
│   ├── model-registry.ts               # 模型/提供商管理
│   ├── builtin-models.ts               # 内置模型列表
│   ├── concurrency.ts                  # 并发控制 + rateLimitDelay
│   ├── render-log.ts                   # 渲染日志
│   ├── media-utils.ts                  # 媒体工具
│   ├── screenplay/                     # 剧本业务域
│   │   ├── script-service.ts           # 剧本解析（薄包装）、分镜生成
│   │   ├── parse-phases.ts             # 4 个独立解析阶段函数
│   │   └── creative-service.ts         # 一句话原创 + 小说改编
│   ├── assets/                         # 视觉资产业务域
│   │   ├── visual-service.ts           # 分层 Prompt 生成 + 图片生成
│   │   ├── prompt-constants.ts         # 系统 prompt + 题材模板
│   │   └── art-presets.ts              # 7 题材美术预设（都市情感/重生逆袭/霸总甜宠/悬疑/古装权谋/穿越/仙侠）
│   ├── video/                          # 视频业务域
│   │   ├── video-service.ts            # Veo/Sora 视频生成
│   │   ├── video-prompt.ts             # 视频 prompt 构建
│   │   ├── prompt-manager.ts           # 角色/场景/美术 prompt 管理
│   │   ├── keyframe-utils.ts           # 关键帧工具
│   │   └── camera-guides.ts            # 镜头运动构图指导
│   ├── editing/                        # 剪辑业务域
│   │   └── edit-service.ts             # AI 剪辑分析 + ffmpeg 渲染
│   └── providers/                      # 多提供商适配
│       ├── base.ts                     # 提供商基类
│       ├── antsk.ts                    # antsk（OpenAI 兼容 API）
│       ├── openai.ts                   # OpenAI 直连
│       ├── google.ts                   # Google（Gemini/Veo）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SttFang/bbdog](https://github.com/SttFang/bbdog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
