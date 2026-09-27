---
trigger: always_on
description: > 给 AI 协作者和开发者的项目导览。**重要：任何功能改动（新增节点、新增工作流、改 IPC、改目录结构）都必须同步更新本文档，保持它与代码一致。**
---

# AGENTS.md

> 给 AI 协作者和开发者的项目导览。**重要：任何功能改动（新增节点、新增工作流、改 IPC、改目录结构）都必须同步更新本文档，保持它与代码一致。**

## 项目简介

AIGC CANVAS：Electron 桌面应用，把 Claude Code / Codex Agent（对话）、React Flow 无限画布、Three.js 3D 导演台和 ComfyUI 生成管线整合在一个项目工作区里，用于 AI 分镜视频创作。用户在聊天里让 Agent 创建“参考图片 → 视频”节点链，在画布上连边、调参、触发生成；生成结果落盘到项目目录并回显到节点。生成片段直接由 video 节点承载，片段内生成 Shot 仍只存在于提示词时间线中；`director` 节点另有用于白模预演的可编辑 Shot/机位工程。

技术栈：Electron + Vite + React 19 + TypeScript + Tailwind CSS 4 + @xyflow/react（画布）+ Excalidraw（图片编辑台）+ Three.js / React Three Fiber（3D 导演台）+ zustand（状态）+ @anthropic-ai/claude-agent-sdk + @openai/codex-sdk（Agent）+ zod（工具入参校验）。包管理用 pnpm。

## 目录结构

| 路径 | 作用 |
|---|---|
| `electron/main/` | Electron 主进程入口与服务 |
| `electron/main/index.ts` | 主进程入口：窗口、`local-file://` / `workspace://` 自定义协议（本地媒体预览、Range 视频流；`.aigc-line` 默认拒绝，仅白名单只读画板预览 PNG）、注册全部 IPC handler |
| `electron/main/ipc/` | IPC handler 层，只做参数转发 + 错误包装，业务逻辑在 services |
| `electron/main/services/` | 主进程业务服务 |
| `electron/main/services/agent/` | Agent 子系统：会话、流式输出、MCP 工具、系统提示词、画布桥接 |
| `electron/main/services/agent/tools.ts` | 双 Agent 共用的 MCP 工具定义与处理器（zod schema 在这里） |
| `electron/main/services/agent/codex-session.ts` | Codex SDK 会话：按项目排队、runStreamed 事件、恢复、AbortSignal 中断与上下文清空 |
| `electron/main/services/agent/canvas-mcp.ts` | Codex 画布 MCP：每项目独立的 loopback HTTP 服务及随机 Bearer Token，共用工具 schema/处理器 |
| `electron/main/services/agent/codex-runtime.ts` / `codex-client.ts` / `models.ts` | SDK 原生运行时与 ASAR 路径解析；子进程合并 NO_PROXY/no_proxy 并强制回环地址绕过代理，保留外网代理及原有排除项；没有显式代理环境变量时，通过 Electron resolveProxy 解析系统代理并传入 Codex 会话和模型发现子进程；只读模型发现；Claude/Codex 模型列表 |
| `src/shared/agent-config.ts` | 项目 Agent 类型与模型校验、历史项目默认值 |
| `src/components/CreateProjectDialog.tsx` | 新建项目弹窗：名称、Agent、模型与目录选择 |
| `electron/main/services/agent/prompts.ts` | Agent 系统提示词（分镜创作规范） |
| `electron/main/services/agent/builtin-plugin.ts` | 解析开发/打包环境中的内置 Claude Plugin 路径并生成 SDK 配置 |
| `electron/main/services/agent/claude-runtime.ts` | 从 Claude SDK 解析同版本的平台原生程序，将 ASAR 虚拟路径映射到真实解包路径；模型发现与聊天会话共用 |
| `electron/main/services/agent/skills.ts` | 枚举当前可用 Skill：活动 SDK 会话 + 应用内置、项目级、用户级目录兜底 |
| `electron/main/services/comfyui.service.ts` | ComfyUI 全部交互：工作流模板注册、参数注入、上传媒体、排队、轮询、下载结果 |
| `electron/main/services/google-image.service.ts` | Google Gemini 图片生成：Nano Banana 2 / Pro 的 2K 文生图与最多 14 张有序参考图生成，产物写入项目目录 |
| `electron/main/services/seedream-image.service.ts` | 火山方舟 Seedream 图片生成：Doubao-Seedream-5.0-pro / lite 的 2K 文生图与最多 10 张有序参考图生成，产物写入项目目录 |
| `electron/main/services/seedance-video.service.ts` | 火山方舟 Seedance 2.0 视频生成：Agent Plan 异步任务提交/轮询、项目内全模态参考素材编码、结果下载与落盘 |
| `electron/main/services/google-network.service.ts` | Google API 网络层：使用 Electron 网络栈、可选独立 HTTP/HTTPS/SOCKS 代理及可读网络错误 |
| `electron/main/services/qwen-video-analysis.service.ts` | 通用 Qwen 视频分析：接受项目内视频路径或公开 HTTP(S) URL，顺序扫描完整画面与音轨，按自由要求给出带时间证据、事实/转写/推断边界和不确定性的报告 |
| `electron/main/services/project.store.ts` | 项目持久化：索引事务、清单、带消息索引缓存的追加式 JSONL 聊天事件日志、会话 id 与画布快照 |
| `electron/main/services/atomic-file.ts` | 按文件串行事务、唯一临时文件原子替换与退出前写队列刷新 |
| `electron/main/services/generation-task.service.ts` | 按项目/节点持久化生成状态、远端任务恢复、完成结果确认与未知提交人工解除限制 |
| `electron/main/services/media-io.ts` | 读取前文件检查、有界读取、流式下载与媒体准备并发限制 |
| `electron/main/services/project-media.service.ts` | 项目媒体资产：把本地图片/视频/音频复制到 `uploads/`，保存导演台构图、预演视频与画板导出到 `generated/director-stills/`、`generated/director-videos/`、`generated/image-edits/`，保存画板节点缩略图到 `.aigc-line/board-previews/`，扫描 `generated/` 与 `uploads/` 供资产面板使用 |
| `electron/main/services/chat-attachment.service.ts` | 聊天通用文件附件：把长文本原样保存为 UTF-8 TXT，发送前校验普通文件并把项目外文件复制到 `uploads/chat-attachments/`，保证 Agent 只收到项目内可访问路径 |
| `electron/main/services/settings.service.ts` | 应用设置：ComfyUI 与图片/视频/分析服务配置；Agent 使用各自本机配置 |
| `electron/main/services/message-hub.ts` | 主进程内部事件总线（Agent 事件 → 渲染进程推送） |
| `electron/preload/index.ts` | preload：`window.electronAPI` 的唯一出处，渲染进程只能用它访问主进程 |
| `src/shared/` | 主进程与渲染进程共享的代码 |
| `src/shared/ipc.types.ts` | 全部 IPC 请求/响应类型 + `CanvasNodeKind` + 节点 data 结构 |
| `src/shared/ipc.channels.ts` | IPC 通道名常量（唯一定义处） |
| `src/shared/node-capabilities.ts` | 节点能力注册表：每种节点可读写哪些字段、可调用哪些动作（Agent 通过它发现能力） |
| `src/shared/director.types.ts` / `director-schema.ts` | 3D 导演台 v2 可序列化工程类型及共享严格 Zod schema：元素、Transform、Shot、人物路径、相机关键帧/跟随约束与 IPC 结果 |
| `src/shared/director-element-catalog.ts` | 导演台素材注册表：统一 kind、分类、名称、用途、默认尺寸与颜色，供类型/schema、素材库和 Agent 能力复用；禁止引入 Three.js |
| `src/features/director/DirectorAssetLibrary.tsx` / `DirectorPrimitive.tsx` / `director-primitive-parts.ts` | 导演台分类搜索素材库、基础几何渲染与室内建筑组合部件；几何保持在导演台懒加载边界内 |
| `src/components/CanvasArea.tsx` | 画布核心：节点渲染、连线、工具栏、生成动作与任务恢复、撤销重做、canvas command 处理（Agent 操作画布的入口） |
| `src/shared/snapshot-persistence.ts` / `pending-edits.ts` | 按项目保留待保存草稿；编辑器先提交、画布后落盘；离开/退出期间阻止新编辑 |
| `src/shared/edit-history.ts` / `canvas-reference-index.ts` / `canvas-placement.ts` | 有界编辑历史、按节点订阅的图片入边索引、可见区域内新增节点避让 |
| `src/shared/canvas-node-content.ts` | 按节点 id/data 复用内容列表，使拖动、选择和测量不触发媒体入边索引与 Artifact 内容同步 |
| `src/components/ProjectAssetPreview.tsx` / `project-asset-preview-cache.ts` | 视区附近的图片缩略图/视频封面生成、两路解码与有界内存缓存 |
| `src/components/CanvasMediaPreview.tsx` / `canvas-interaction.ts` | 图片按屏幕尺寸选择预览档位，手势结束后受限解码/升级原图；视频封面与点击后单播放器，关闭、离屏和卸载时释放资源 |
| `src/components/CanvasBackground.tsx` | 随世界坐标移动的 CSS 点阵背景，平移时每动画帧仅移动独立合成层，点阵尺寸与纹理只在缩放时更新，避免逐帧背景重绘 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhangtuo723/aigc_line](https://github.com/zhangtuo723/aigc_line) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
