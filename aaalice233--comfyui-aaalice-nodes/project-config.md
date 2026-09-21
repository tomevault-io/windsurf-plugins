---
trigger: always_on
description: 供协作者与 AI 助手使用；与当次明确指令冲突时，以当次指令为准。**本文件只记录长期有效的开发硬规则，不记录具体 Bug、调查过程、操作教程或测试日志，并保持在 500 行以内。**
---

# AGENTS.md

供协作者与 AI 助手使用；与当次明确指令冲突时，以当次指令为准。**本文件只记录长期有效的开发硬规则，不记录具体 Bug、调查过程、操作教程或测试日志，并保持在 500 行以内。**

## 1. 项目边界与依据

本项目选择性重写 [ComfyUI-Danbooru-Gallery](https://github.com/Aaalice233/ComfyUI-Danbooru-Gallery)，只实现已确认的节点和前端能力。

- ComfyUI 官方文档入口为 [https://docs.comfy.org/](https://docs.comfy.org/)；API、生命周期、Schema、list、缓存或前端行为不确定时，先查官方文档和当前安装版本源码，再决定实现。
- 排查前端显示、渲染、widget、画布或交互问题时，必须同时核对当前 ComfyUI 源码，以及本地存在的对应版本 ComfyUI 前端源码（当前参考 `E:/git/ComfyUI_frontend`）；以源码确认生命周期、DOM、LiteGraph、Classic / Nodes 2.0 和样式行为，不得只凭截图、打包 bundle 或经验猜测。
- 现象与预期冲突时，先用同版本的官方内置节点交叉验证。若官方节点也复现，按上游或环境问题处理，不给本包堆私有兼容补丁。
- 方案开始依赖多层时序补丁、轮询或重复状态时，暂停实现并重新核对职责和根因。
- 当前已进入正式发布前稳定期；现有节点身份、工作流序列化、公开前端 API、用户交互和已验证行为均按稳定契约对待。修复优先采用边界清晰的最小改动，禁止因为局部问题顺带改写无关模块、替换成熟实现或进行大范围“顺手重构”。
- 重构必须由明确根因、架构债务或无法安全局部修复的问题驱动，并在动手前确认调用方、状态真源、生命周期、Subgraph、Classic / Nodes 2.0、旧工作流和第三方集成影响。若判断重构能避免实质技术债或显著降低后续风险，先向用户说明触发原因、拟改范围、兼容风险、不重构的代价和验证计划，并明确询问是否进行；获得同意前只做不会锁定重构方向的诊断与必要止损。扩大模块边界、改变持久协议或删除现有能力时同样必须先征得同意；breaking change 同步版本、迁移策略、双语 README 和公开限制。
- 未发布的内部中间态可以删除死代码或收敛实现，但不得借此绕过兼容性评估、迁移责任和回归验证，也不得保留无调用方的兼容壳、废弃别名或历史文档。
- 标识符使用英文；用户可见文案提供 English + 简体中文，并跟随 ComfyUI 界面语言。
- Classic、Nodes 2.0 与 Subgraph 是所有节点开发的基线支持面，不是后续可选适配；新增或修改节点必须同时成立。暂不支持 App Mode。
- 新增依赖前必须征得同意；禁止静默吞错、伪造成功或用降级掩盖根因。
- 只修改任务直接涉及的内容；工作区已有改动默认属于用户。
- 第一方手写 JS / TS / Vue / Python / CSS / SCSS（含测试、脚本与部署代码）以不超过 600 个物理行为模块化目标，800 行为禁止超过的硬上限；空行和注释同样计入。接近目标时必须按稳定职责拆分，业务入口只保留注册、路由或装配，状态模型、运行时协调、DOM、Dialog、生命周期和领域样式分别归属独立模块。固定上游产物只能按 `js/vendor/**` 等精确路径豁免，手写代码不得借 vendor / generated 名义、压缩排版、删除必要说明或转移到另一巨型文件规避检查。
- 提交消息使用 `type(scope): 中文描述`，标题不超过 72 个字符。

## 2. 文档职责

| 位置 | 职责 | 不应包含 |
|---|---|---|
| `README.md` / `README.en.md` | 用户安装、已发布功能、用法和公开限制 | 开发进度、下一项、完整排期、测试记录、协作规则 |
| `AGENTS.md` | 开发硬规则、架构边界、验收门槛 | 具体 Bug、长命令、教程、调查过程 |
| `CONTEXT.md` | 项目领域词汇和统一称呼 | 文件路径、字段名、实现方案 |
| `docs/adr/` | 难逆且存在真实取舍的架构决策 | 操作步骤、视觉细节 |
| `docs/design/` | 设计语言、组件和交互规范 | 后端协议决策 |
| `docs/development/` | 架构、内部路线图、测试与发布 runbook | 普通用户安装教程 |

- 文档入口见 [`docs/README.md`](docs/README.md)。
- `README.md` 为简体中文首页 README（同时作为 Registry readme）；`README.en.md` 为 English。两份结构必须对齐、页顶互链。
- 节点重置或增删时：README 只更新已发布节点、用户用法和公开限制；[`roadmap.md`](docs/development/roadmap.md) 独立维护进度、下一项、稳定编号和排期。
- ADR 状态只用 `Accepted`、`Superseded by ADR NNNN` 或 `Rejected`。已发布决策被替代时保留历史并链接后继；未发布中间态删除后不保留 ADR。
- 一次性调查、聊天结论、本机故障笔记和测试截图不进入仓库。

### 2.1 上下文入口

`AGENTS.md` 是开发上下文总入口。需要参与判断的项目文档必须在这里使用 `@相对路径` 引用；普通 Markdown 链接只用于阅读导航，不视为上下文注入。

所有开发任务先加载：

- @PRODUCT.md
- @DESIGN.md
- @CONTEXT.md
- @docs/development/architecture.md

按任务类型继续加载：

| 任务 | 注入文档 |
|---|---|
| 文档整理、职责判断或查找入口 | @docs/README.md |
| 节点重置、增删节点或调整优先级 | @docs/development/roadmap.md |
| 测试、调试、GUI 验收或发布前检查 | @docs/development/testing.md |
| 前端性能、富 DOM、画布卡顿或热路径 | @docs/development/performance.md、@docs/development/testing.md |
| 发布、版本和 Registry | @docs/development/release.md |
| 前端视觉、组件、主题或可访问性 | @docs/design/ui-system.md |
| QuickGroupManager 交互与布局 | @docs/design/quick-group-manager.md |
| ResolutionPreset、画幅坐标板或个人分辨率预设 | @docs/design/resolution-preset.md |
| FetchFromKrita 或 Krita Bridge | @docs/adr/README.md、@docs/adr/0011-krita-bridge-execution-snapshots.md |
| BooruGalleryNode、多站点画廊或虚拟瀑布流 | @docs/design/booru-gallery.md、@docs/adr/0010-booru-gallery-capability-snapshots-masonry.md |
| Discord 分享、最新运行相册或成员验证中继 | @docs/design/discord-share.md |
| PromptSelector、词库或 DIY 侧边栏 | @docs/design/prompt-selector-workspace.md、@docs/adr/0007-independent-prompt-library-live-references.md、@docs/adr/0008-stable-dashboard-control-bindings.md、@docs/adr/0012-dashboard-source-scoped-groups.md、@docs/adr/0013-dashboard-multi-target-binding-sets.md、@docs/adr/0014-dashboard-value-import-recovery.md |

新增专题文档时，若其内容会影响实现或验收，必须同时补到本节。README 面向用户，不作为默认开发上下文注入。

## 3. 仓库与后端

```text
ComfyUI-Aaalice-Nodes/
├── __init__.py
├── nodes/{control,prompt,tools,_lib}/
├── js/{lib,assets}/
├── locales/{en,zh,zh-TW}/
├── tests/
└── docs/{adr,design,development}/
```

- 根 `__init__.py` 只公开 `WEB_DIRECTORY` 和 `comfy_entrypoint()`，不放业务节点。
- V3 节点默认一节点一文件；`nodes/<domain>/__init__.py` 导出 `NODE_CLASSES`，只注册已实现的域。
- 新增域时同步 `nodes/__init__.py` 与 `pyproject.toml` packages。
- category 使用 `Aaalice/<domain>`；当前域为 `Aaalice/control`、`Aaalice/prompt` 与 `Aaalice/tools`。
- `nodes/_lib/` 只放不依赖运行中 ComfyUI 的纯逻辑，并可直接单测。
- 运行时错误保留原始原因与参数上下文；不得把导入错误伪装成未实现。
- `validate_inputs()` 只校验执行前可获得的字面量、类型或前端注入 payload；连接输入在该阶段没有上游运行值，禁止对其做非空或内容语义校验。此类检查必须放在 `execute()` 拿到真实上游值之后。
- HTTP 公共认证 header 不得无条件包含 `Content-Type: application/json`；无 body 的 GET 只发送认证等实际 header，JSON POST 使用客户端 `json=` 参数自动生成编码和 Content-Type，避免服务端把空 GET body 当作无效 JSON。
- 模块关系与状态真源见 [`architecture.md`](docs/development/architecture.md)。

## 4. 前端、渲染与状态

### 4.1 生命周期与状态

- `WEB_DIRECTORY = "./js"`；业务扩展使用 `app.registerExtension`，共享模块不得自行重复注册。
- `js/extension.js` 是前端唯一包入口；每个业务扩展模块必须由该入口显式静态导入，不得假设 `WEB_DIRECTORY` 会自动执行目录中的其它 `.js` 文件。新增节点时必须用入口契约测试锁定该导入。
- 前端相对 import 必须按浏览器中的 `/extensions/ComfyUI-Aaalice-Nodes/` 挂载路径计算，不能按仓库文件系统层级猜测；新增或移动嵌套模块后必须用契约测试确认所有相对 import 只解析到本包公开路径或 ComfyUI `/scripts/`，避免单个 404 阻断整个 `extension.js` 模块图。
- 模块拆分或新增 barrel re-export 时，本包相对路径的 named import 必须能在目标文件或其 re-export 链中解析到稳定的 named export；提交前运行 `node --test tests/frontend_import_paths.test.js`，该契约不替代 ComfyUI 公共脚本的运行时验证。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aaalice233/ComfyUI-Aaalice-Nodes](https://github.com/Aaalice233/ComfyUI-Aaalice-Nodes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
