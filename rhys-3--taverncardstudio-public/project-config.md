---
trigger: always_on
description: 本工作区用来在本地编写、检查、拆分和打包 SillyTavern 角色卡，供人和 AI 共同维护。用户直接在工作区对话即可，不要求填写简报或复制开工指令。按任务加载下面的技能；详细文档作为按需参考，不一次读完。
---

# TavernCardStudio 工作区约定

本工作区用来在本地编写、检查、拆分和打包 SillyTavern 角色卡，供人和 AI 共同维护。用户直接在工作区对话即可，不要求填写简报或复制开工指令。按任务加载下面的技能；详细文档作为按需参考，不一次读完。

## 三条路径

| 路径 | 起点 | 之后做什么 |
| --- | --- | --- |
| 简单卡 | `pnpm card new <名称>` | 改 `src/index.yaml` 与 `src/text/*.md`；可选加世界书。人物设定可以留在卡字段，也可以放世界书 |
| 复杂卡 | `pnpm card new <名称> --modules "worldbook,mvu,vue"` | 世界书、变量、正则、脚本、界面互相引用，改动前先读 [docs/COMPLEX-CARDS.md](docs/COMPLEX-CARDS.md) 定位内容 |
| 导入卡 | `pnpm card split <PNG或JSON文件> --name <名称>` | 先 `pnpm card diff <名称>` 确认拆分无损，再在 `src/` 上改；不从原卡重新拆分覆盖编辑 |

## 跨 agent 入口

- 以当前对话和实际卡片为起点。能自行判断的细节直接处理，只询问影响目标或重要结果的缺失信息；已有创作笔记仅作参考，不要求用户补表。
- 用户请求安装或首次运行遇到环境问题时，按 [安装环境](docs/INSTALL.md) 检查并补齐 Node.js、固定版本 pnpm 和项目依赖，再验证能启动；普通设定讨论不必先跑环境检查。
- 工作顺序：本文 → 下表技能 → 当前任务需要的 docs 页。不要一次加载全部技能和 `references/`。
- 开工前用 `pnpm card map <名称>` 和卡根 `内容导航.md` 定位内容落在哪个文件，再动手改。
- 导入卡的世界书要整理完才算交付：`card map` 会给出世界书清单与待分类条目，分批读取必要正文后把目录写进条目 YAML 的 `分类`，再跑 `pnpm card organize <名称>`；范围与边界见 [docs/WORLDBOOK-FILES.md](docs/WORLDBOOK-FILES.md)。
- 改完跑 `pnpm card check <名称>` 与 `pnpm card build <名称>`；改过引用、模块或构建项再加 `pnpm card diff <名称>`。报告时把静态检查、构建和真实宿主验证分开写。

## 按任务加载技能

技能放在仓库 `.agents/skills/`，随仓库维护，不必安装到用户全局目录。支持该目录的 agent 可按描述选择；其它 AI 按下表直接读取 `SKILL.md`，同样能使用。上游参考目录中的技能属于研究资料，不是本仓库执行规范。

| 任务 | 入口 |
| --- | --- |
| 新卡、整体改造、复杂卡研究 | [.agents/skills/tavern-card-authoring/SKILL.md](.agents/skills/tavern-card-authoring/SKILL.md) |
| 提示词文体：世界书条目、卡文本字段、规则与输出协议、变量初值与更新规则 | [.agents/skills/tavern-prompt-style/SKILL.md](.agents/skills/tavern-prompt-style/SKILL.md) |
| 世界书蓝绿灯、关键词、分类、顺序、深度、提示词 | [.agents/skills/tavern-worldbook/SKILL.md](.agents/skills/tavern-worldbook/SKILL.md) |
| MVU 条目、初值、更新规则与输出协议 | [.agents/skills/tavern-mvu/SKILL.md](.agents/skills/tavern-mvu/SKILL.md) |
| Zod 4 结构、默认值、转换与注册 | [.agents/skills/tavern-mvu-zod/SKILL.md](.agents/skills/tavern-mvu-zod/SKILL.md) |
| 正则显示、提示词清理、流式块与深度 | [.agents/skills/tavern-regex/SKILL.md](.agents/skills/tavern-regex/SKILL.md) |
| 酒馆助手脚本、EJS | [.agents/skills/tavern-scripting/SKILL.md](.agents/skills/tavern-scripting/SKILL.md) |
| HTML/Vue 前端 | [.agents/skills/tavern-frontend/SKILL.md](.agents/skills/tavern-frontend/SKILL.md) |

规则分三层：**运行时要求**以锁定实现为准；**本仓库规范**是新卡的一致写法；**参考卡设计**只在适合新卡目标时采用。已有卡迁移保留自身协议，不因套用模板而默默改名或更换数据模型。

## 用户偏好与工具强约束

写卡前分清这三类要求，不要把偏好当成硬规则，也不要因为它是偏好就忽略。

- **工具强约束**：违反会让 `check`／`build` 失败或产物出错。例如依赖必须是无凭据 HTTPS 且地址里含 `version` 所写的固定版本、`.card/` 与 `original/` 不能手改、条目 YAML 的键必须在允许列表内、引用路径不能越出 `src/`、`project.name` 必须与目录名一致。
- **本仓库规范**：新卡的一致写法，例如先定体验目标再选模块、新卡推荐用世界书承载设定、模块增删要同步条目与 `dependencies`。它是默认值；用户有明确目标时可以偏离，偏离后在报告里说明。
- **用户偏好与参考卡设计**：题材、文风、数值体系、条目划分、界面布局、开场白数量。按用户目标执行，从不照搬示例卡或参考卡；也不要把自己的偏好写成仓库规范。

三类冲突时，以运行时实现和用户明确要求为准。拿不准就按默认值做，并在报告里写清这是默认值。

## 编辑边界

- 完整导入原卡放 `imports/`，拆分编辑区放 `cards/<名称>/src/`，打包成品放 `dist/<名称>/`。界面导入与 CLI `split` 共用归档逻辑；同名同内容复用，同名不同内容加序号，不覆盖。AI 与创作者编辑同一份 `src/`，修改后直接 build，不从原卡重新拆分覆盖编辑。
- `cards/` 默认不提交 Git，只有仓库自带的官方示例按 `.gitignore` 的名单被跟踪，名单与 `maintenance/public-release.json` 的 `examples` 一致；用户自建卡留在本地。`dist/`、`imports/`、`work/` 与参考资料缓存同样不进版本库。要把自己的卡纳入版本管理，见 [docs/UPDATING.md](docs/UPDATING.md)。
- `project.original.importPath` 只记录工作区内原卡归档位置；它不是构建输入，删除 `imports/` 归档不影响工程内 `original/` 的独立校验副本。
- 写卡时人只改两类文件：`cards/<名称>/src/` 下的正文（Markdown、JavaScript、样式、界面源码）与可读配置（`src/index.yaml`、条目 YAML）。卡 JSON、PNG、指针与打包产物都由脚本生成，不手改。
- `cards/<名称>/src/index.yaml` 是卡片总入口：`角色`、`文本`、`备选开场白`、`群聊开场白`、`世界书`、`正则`、`脚本` 都在这里引用；文本字段的值是 `src/...` 文件路径。
- 世界书、正则、脚本条目各自是一个 YAML。`关联` 由工具生成，用来在重排后对上原卡里的条目；保留即可、不手改，新条目可以省略。文件没改名时，漏写 `关联` 会按原路径找回原条目，但保留 `关联` 仍是首选。条目正文用 `文件: src/...` 引用，正则用 `替换文件`。
- 世界书条目还可以写可选键 `分类: 地理/北境`：它只决定 `organize` 把设置与可移动正文放进哪个目录，不写进卡，也不改变触发、位置与注入。
- `cards/<名称>/project.json` 只记录模块、依赖、头像、`runtimeVerified`、`original` 与工具维护的 `source`／`state`；增删条目、改顺序、改引用都由索引与条目 YAML 决定。
- `cards/<名称>/内容导航.md` 是工具生成的内容导航，用来看内容落在哪个文件，不是编辑入口。
- `.card/` 由工具维护：`state.json` 保存机器兼容数据并带 sha256 校验，未知与 legacy 数据原样保留，迁移快照在 `.card/migration-v1/`；它要和工程一起备份／提交，删掉就丢失 `关联` 映射与已取消引用记录，平时不读取、不修改。
- `modules` 只是新建模板的选择记录，不是运行时开关；增删已有卡的功能要同步维护实际条目、条目 YAML 与依赖。
- `cards/<名称>/original/` 保存导入原件与基线，只读；加载时会逐个校验 sha256。
- 构建产物在仓库根目录 `dist/<名称>/`，由 `pnpm card build` 生成，不手改。
- `references/` 是锁定的参考资料区，只读；其中代码不执行，内容不自动打进卡。公开包不带参考资料实体，用 `pnpm refs restore` 按锁文件取回本地缓存。
- 新增或重命名卡片目录优先用 `pnpm card new` 与 `pnpm card split`，两者默认生成 v2；旧版目录用 `pnpm card migrate` 原地迁移。引用增删或重排后下次打包自动生效，删除引用后源文件可以保留，但不参与打包。

世界书文件按分类目录与条目名组织，不用全局编号排序。旧路径继续支持；整理已有工程见 [docs/WORLDBOOK-FILES.md](docs/WORLDBOOK-FILES.md)，动态宏与 EJS 按需读 [docs/WORLDBOOK-TEMPLATING.md](docs/WORLDBOOK-TEMPLATING.md)。

## 命令

创作者可双击根目录的 `打开角色卡工具.cmd`（Windows）或 `打开角色卡工具.sh` 使用本地界面，也可以运行 `pnpm studio`。界面使用同一拆分／检查／构建实现，增加工程改名、副本与封面操作；服务只监听 127.0.0.1。界面源码在 `tools/studio-ui/`，服务在 `tools/studio.mjs`，工程管理逻辑在 `tools/lib/studio-operations.mjs`。操作说明见 [docs/STUDIO.md](docs/STUDIO.md)。

| 命令 | 作用 |
| --- | --- |
| `pnpm setup:workspace` | 首次准备：检查环境，缺依赖或锁文件漂移时按 `--frozen-lockfile` 安装一次；不满足条件时打印操作指导并以退出码 1 结束 |
| `pnpm doctor:workspace` | 只读检查：Node 与 pnpm 版本、依赖、锁文件与本地工具；不安装、不写文件 |
| `pnpm card new <名称> [--modules worldbook,regex,script,mvu,html,vue]` | 新建卡片骨架；不带 `--modules` 时为纯文字卡 |
| `pnpm card split <PNG或JSON文件> [--name 名称]` | 导入已有卡并拆分成可编辑文件 |
| `pnpm card check <名称\|all>` | 静态检查 |
| `pnpm card build <名称\|all>` | 生成可导入产物到 `dist/<名称>/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhys-3/TavernCardStudio-Public](https://github.com/rhys-3/TavernCardStudio-Public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
