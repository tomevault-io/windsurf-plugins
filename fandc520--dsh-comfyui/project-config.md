---
trigger: always_on
description: 本文件为 Claude Code 在本仓库工作时的向导：项目结构、模块职责、开发命令与必须遵守的契约。
---

# CLAUDE.md

本文件为 Claude Code 在本仓库工作时的向导：项目结构、模块职责、开发命令与必须遵守的契约。

## 项目概览

`dsh-comfyui` 是 **DeepSeek Harness (DSH / cordis) 的 ComfyUI 插件**：让 Agent 通过工具直接驱动 ComfyUI 生成、处理图像与视频，并在浏览器端提供工作流库 / 资产 / 队列面板、对话内媒体卡片和设置页。

- 包名 `dsh-comfyui`，ESM（`"type": "module"`），Node ≥ 22.19，MIT。
- 入口：host 侧 `lib/index.js`（由 `src/*.ts` 经 tsc 编译）；浏览器侧 `client/client.js`（由 `src/client/*` 经 tsdown 打包）。
- peer 依赖：`@deepseek-ai/cordis`（必需）、`@deepseek-ai/dsh-settings`（可选，设置页持久化）。运行时依赖只有 `@deepseek-ai/schemastery`。
- `cordis.patch.yml` 把插件以 `id: comfyui` 插入 profile 层栈；`package.json` 的 `dsh` 字段声明 bundle patch 与 client 平台/注入。

## 常用命令

```sh
npm run typecheck    # host + client 双 tsconfig 类型检查（不产物）
npm run build        # build:host (tsc → lib/) && build:client (tsdown → client/client.js)
npm run build:host   # 仅 host
npm run build:client # 仅客户端 bundle
npm pack --dry-run   # 发布前必查：README 引用的资源都要在 files 白名单里
node scripts/test-store-params.mjs   # 离线自测：参数保存/重载（需先 build）
```

**改动生效规则**：`src/*.ts`（host：tools/routes/store/params/skill…）改完要 **重启 DSH**；`src/client/*` 改完 **刷新页面** 即可。skill 文本编译进 `lib/skill.js`，属 host 侧。

构建产物 `lib/`、`client/` 不入库（见 `.gitignore`），clone 后需 `npm install && npm run build`。

## 架构

插件分两半，通过同源 HTTP 路由通信，浏览器 **从不** 直连 ComfyUI（无 CORS、无混合内容、密钥不下发）：

```
Agent ──tools──┐
               ├─→ ComfyUIRuntime（src/index.ts 组装的能力对象）
浏览器 ─routes─┘        │
                        ├─ ComfyUIClient (comfyui.ts) ──HTTP/WS──→ ComfyUI 服务器
                        ├─ ComfyUIStore  (store.ts)   ──JSON 文件─→ $DSH_HOME/data/dsh-comfyui/
                        ├─ QueueTracker  (queue.ts)
                        └─ ProgressTracker (progress.ts, WebSocket)
浏览器 ←媒体─ /comfyui/media 同源代理 (proxy.ts)
```

`ComfyUIRuntime`（定义在 `src/tools.ts`，实例化在 `src/index.ts`）是唯一的能力面：工具层和路由层都只依赖它，不各自持有 client/store。新增功能优先加 runtime 方法，而不是在 routes/tools 里直接 new client。

### 两个"工作流主题"（贯穿全项目的核心概念）

- **图工作流（衍生主题）**：ComfyUI 端保存的画布 UI 图（nodes/links/widgets），是"源"，**不能直接运行**；一张画布可能含多个互相独立的流程。
- **API 工作流（运行主题）**：API 格式 prompt（node id → `{ class_type, inputs }`），是"运行单元"，由图 **提取（extract）** 而来或用户直接粘贴导入。

提取路径：`analyzeGraph`（连通分量分析）→ 用户在面板选 整体 / 按分量 / 主流程 → `convertGraphToApi`（图→API）→ `analyzeWorkflowParameters`（识别可调参数）→ `store.saveWorkflow`。

## host 侧模块（`src/`）

| 文件 | 职责 |
| --- | --- |
| `index.ts` | 插件入口：解析配置、组装 `ComfyUIRuntime`、注册设置节 / 工具 / skill / 路由 / 媒体代理，全部挂在 fiber 上随插件卸载。`export const inject = ['tools']`。 |
| `config.ts` | schemastery 配置 schema（同时供 cordis.yml 入口配置和 `comfyui:` 设置节使用）+ 同形状的 TS 类型。`outputDir` 只走 cordis.yml，留空时删除资产会自行推断 ComfyUI 输出目录。 |
| `comfyui.ts` | ComfyUI HTTP 客户端：queuePrompt / history / queue / jobs / userdata / object_info / view / upload / interrupt 等，加上 `collectMedia`、`mediaProxyUrl`、`waitForCompletion`。上传有两个入口：`uploadFile` 转发浏览器原样的 multipart，`uploadMedia` 用 FormData 包好字节再传（`/upload/image` 只吃 multipart，裸 body 会 400）。模块级 `CLIENT_ID` 让排队与 WS 进度同源。 |
| `store.ts` | 持久化：工作流库、资产索引、加载区加载位（`LoadSlot[]`，`null` = 空位，兼容旧的单图格式）、媒体尺寸、上传哈希、任务跟踪，均为 dataDir 下的 JSON 文件。 |
| `queue.ts` | `QueueTracker`：记住本插件提交过的 prompt，`sweep()` 在读取（queue/assets 路由）时把完成的运行归档进资产索引；无后台定时器。 |
| `progress.ts` | `ProgressTracker`：连 ComfyUI `/ws` 收 `progress` 事件，best-effort（远端鉴权代理下可能无进度），断线重连直到 dispose。 |
| `analyze.ts` | 画布分析：groups 无执行语义，可执行单元 = 激活节点的连通分量，忽略 bypass(mode 4) 与悬空 UI 节点。 |
| `convert.ts` | 图 → API 转换：链接变 `[String(nodeId), slot]`，widgets_values 按图节点自身 `inputs` 顺序对齐，Reroute/bypass 直通，无法表达的节点报错。 |
| `params.ts` | 可调参数：自动识别（提示词/分辨率/步数/种子/时长/宽高比/加载节点）+ 用户高级参数；`numberSpecOf` 从 object_info 读数字输入的声明类型（INT/FLOAT + min/max/step），存进参数的 `numberKind`；`applyWorkflowParameters` 在运行时写回工作流（int 四舍五入、bool 归一化 `"true"`/`0` 这类写法、连线输入与加载参数不被空默认值覆盖、未传值的加载参数按加载位顺序取用）。 |
| `templates.ts` | 内置 API 模板：`txt2img`、`img2img`（核心节点）、`video`（Wan 2.1，需 ComfyUI-WanVideoWrapper）。 |
| `tools.ts` | 模型侧工具定义与注册 + `ComfyUIRuntime` 接口 + 后台任务/结果回显。 |
| `routes.ts` | 浏览器侧同源 HTTP 路由（面板与设置页的全部数据来源），写操作强制同源。 |
| `proxy.ts` | `/comfyui/media` 媒体代理：主路径按 `file`+`subfolder`+`type` 直取（不依赖 history），旧的 prompt/node/index 链接先查 history，查不到再回落到资产索引里的文件引用。 |
| `http.ts` | 路由小工具：`sendJson` / `readJsonBody` / `readRawBody` / `sameOrigin` / `errorMessage`。 |
| `host-hint.ts` | 记住浏览器实际访问用的 origin（Host/Referer），让生成的媒体 URL 对远端浏览器可达；回环 origin 不覆盖已知外部 origin；`detectLanOrigin` 作兜底。 |
| `skill.ts` | 配套 skill `dsh-comfyui-workflows`（runtime，rank 250）：两个主题的区分、画布分析规则、图→API 技术规则、参数与加载区说明、省 token 的运行流程。 |

### Agent 工具（`registerComfyUITools`）

| 工具 | 作用 |
| --- | --- |
| `comfyui_run` | 提交 `workflow`（API 格式）或 `template`（txt2img / img2img / video），`inputs` 按节点 id 覆盖输入；`mode: sync`（默认，返回媒体）/ `async`（返回 job id，用 `job_output` 收结果）。 |
| `comfyui_object_info` | 列出服务器支持的节点定义，可用 `filter` 按类名子串收窄。 |
| `comfyui_workflow` | `action: list` 列出库里可运行工作流（参数清单含 `numberKind` 整数/小数标注）+ ComfyUI 端图工作流（含 `extracted` / `derived`）+ `loadArea`（加载位数量与已放入的素材，Agent 据此知道用户加载了什么）；`action: run` 按 id 运行并传 `parameters` 覆盖；`action: get` 仅供诊断（输出完整 JSON，很费 token）。 |

### 浏览器路由（全部挂在 `webServer` 子 fiber 上）

`/comfyui/` 前缀，写操作要求同源：

- 配置与探活：`ping`(GET)、`config`(GET 读脱敏 / POST 写)、`test`(POST 连接探测)
- 工作流库：`workflows`(GET/POST)、`workflows/recognize`(POST)、`workflows/input-options`(POST)、`workflows/delete`(POST)、`workflows/run`(POST)
- ComfyUI 端图工作流：`comfy-workflows`(GET)、`comfy-workflows/analyze`(GET)、`comfy-workflows/extract`(POST)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fandc520/dsh-comfyui](https://github.com/fandc520/dsh-comfyui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
