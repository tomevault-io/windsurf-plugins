---
trigger: always_on
description: 用来创建、修改、重构、评估、打包和优化其他 skill。用户提到从零做 skill、把一段工作流程沉淀成 skill、改现有 skill、把别人的 skill 按当前这套架构重写、做轻优化或完整改造、设计评测、验证 skill 是否真的更好、优化触发描述，或打包交付 skill 时，都应使用这个 skill。
---


# 规则

- 把当前 `SKILL.md` 所在目录视为 `<skill-base>`。所有 bundled resources 都从这里解析，不要依赖调用方当前工作目录。
- 先判断这个 skill 或改动值不值得存在，再决定怎么写。如果拿掉一块不会伤筋动骨，就优先删掉或不要加进去。
- 执行过程中始终显式维护两个状态：`current_path` 和 `current_step`。
  - `current_path` 只能是：新建 / 修改 / 评估 / 优化 / 打包
  - `current_step` 只能是当前正在执行的 `Step N`
  - 每次切换分支或进入重型操作前，先复述：当前路径、当前步骤、下一动作
  - 如果对话变长、插入大量工具输出、或任务目标变化，先回到 Step 1 重新判路，不要凭惯性继续
- 维持层级分工：
  - 主 `SKILL.md`：只放耐久规则、工作流程，以及少量确实承重的可选 section
  - `references/`：长解释、内部例子、schema、低频模块说明
  - `assets/`：Claude 应该直接遵循、复制、填写的模板或文件
  - `scripts/`：确定性或重复性执行
  - `config.yaml`：人会频繁修改的参数
- 如果目标 skill 是单文件，主 `SKILL.md` 的顶级 section 只允许：`角色`、`规则`、`工作流程`、`例子`、`输出格式`、`索引`。其中必选只有 `规则` 和 `工作流程`。
- `角色`、`例子`、`输出格式`、`索引` 都不是默认必选项。只有拿掉它会明显降低稳定性时，才允许加入。
- `例子` 是给模型看的内部参考 / canonical case，不是给用户看的提问示例；用户入口示例属于 `description` 或触发层材料，不属于主 body 的 `例子`。
- `输出格式` 是给模型直接遵循的模板、骨架或字段约束，不是给用户解释“你会怎么输出”的说明文。
- 如果已经启用 `references/`，就不要再把长 `例子` 留在主 `SKILL.md`；如果已经启用 `assets/`，就不要再把长 `输出格式` 留在主 `SKILL.md`。
- 只要目标 skill 根目录下出现 bundled resources（例如 `references/`、`assets/`、`scripts/`、`agents/`、`evals/`、`config.yaml`），主 `SKILL.md` 就必须明确把当前 `SKILL.md` 所在目录定义为 `<skill-base>`，让模型知道所有本地资源相对谁解析。
- 单文件闭集、下沉阈值、校验规则这些“creator 级架构说明”，默认尽量留在 creator 和 validator；不要整段原封不动塞进每个目标 skill。目标 skill 只保留自己真正承重的最小结构规则。
- 默认路径要轻。不要一上来就跑重型 benchmark、blind comparison 或触发优化，除非用户真的需要这一级证据；但评估 / 测评 / 评测 是例外，只要用户明确提出，就只能进入标准化正式流程，不能降级成轻量判断。
- 在这个项目里，只要用户说的是“评估 / 测评 / 评测”，就只有一套标准化正式流程：先做前置对齐，再进正式执行，最后必须生成 `review.html` 和 `report.html`；不存在轻量版、降级版或聊天结论版。
- 对任何带明显主观标准、人格模仿、方法论借用，或存在多种“到底算哪种好”可能性的评估，不要直接开始跑 eval；先让 AI 做 skill 判型，给出推荐评法和其他可选评法，再和人类对齐成正式评估计划。这个前置对齐只是正式评估的第一段，不是另一种更轻的评估模式。
- 只要用户提到“评估 / 测评 / 评测 / 测一下 / 比较效果 / 有 skill 和没 skill / 两个 skill 谁更好”这类请求，第一次响应必须先停在“评估前置提案”，不能直接给分、不能直接说谁更好、不能直接进入 with-vs-without / A-B / benchmark / review；但这只是正式流程里的等待拍板阶段，不是完成态。
- 在评估路径里，AI 自己写出来的推荐方案不算“已经确认”；只有用户明确拍板“按这个标准评”，才允许进入正式评估计划和执行层。用户一旦拍板，就必须继续走完整个正式流程，不能改走结构判断/评审模式，也不能只给口头结论、Markdown 结论或普通 review。
- 默认交付物也要轻。不要因为“以后可能有用”就顺手创建 `evals/`、workspace、`config.yaml`、`agents/openai.yaml`；只有当前任务真的需要，才把它们带进最终 skill。
- skill 内部文件指针默认写成可移植形式，例如 `<skill-base>/references/...`。不要把一次运行中的绝对路径写进最终交付物，除非用户明确要求做成只在当前机器使用的临时版本。
- 文件指针和命令都尽量写死、写全。把 `<python-cmd>` 视为当前环境可用的 Python 命令：macOS/Linux 通常是 `python3`，Windows 通常优先 `py -3`，其次 `python`。
- 当前 creator 的推荐安装方式是：把 `https://github.com/DazhuangJammy/DazhuangSkill-Creator.git` 用 `git clone` 放进 Claude Code / Codex / Open Claude 的 skill 目录；不要默认让用户或 AI 直接复制文件夹。
- 当前 creator 自带轻量更新检查。只要本地脚本可用且已经进入真实执行，不是纯讨论产品形态，就在 Step 1 开头运行 `<python-cmd> "<skill-base>/scripts/check_update.py" --json`。
- 更新检查、联网失败、或自动更新失败都不阻断当前任务；只有脚本返回 `should_notify = true` 时，才用 1-2 句告诉用户版本差异和下一动作。
- 自动更新默认开启；只要 `<skill-base>/config.yaml` 没有显式关闭 `update_check.auto_update`，并且当前安装是干净的 git clone 工作区，就允许脚本尝试 `git pull --ff-only`。
- 如果更新脚本返回 `status = updated`，说明本地文件已经拉到新版本，但这次调用仍沿当前已加载版本继续；新版本从下一次调用这个 skill 起完整生效。
- 根据用户技术水平调整术语密度；必要时简短解释，不要炫术语。
- 修改已有 skill 时，除非用户明确要求，否则保留原名。
- 给别人做优化时，默认先判断这次是同一套蓝图下的哪种力度：`轻优化`、`结构重构`、`完整改造`；不要把它们当成三套不同方法论。
- 新建 skill 和重构 skill 共用同一套架构目标：主 body 扛耐久规则和工作流程；单文件时按固定 section 白名单收；复杂到会漂移时才加紧凑 `# 索引`；长例子 / 长输出格式 / 长解释再按需要下沉。
- 重构现有 skill 时，把旧 skill 当作素材和脏输入，不当作格式约束；目标是按当前蓝图重新落盘，而不是尽量保留原写法。
- 这套蓝图强调对齐架构原则，不强调机械套模板；如果一个 skill 单文件就够稳，就不要为了形式统一硬拆目录。
- 修改 skill 后，要明确说明：主 body 留了什么、下沉了什么、删了什么。
- 不要把“记住流程”完全寄托在上下文残留上；要用状态播报主动刷新 workflow 锚点。
- 给新 skill 加 `# 索引` 不是默认强制项；只有当复杂度已经高到容易漂移时，才加入一个紧凑索引。
- 如果目标 skill 的默认产物本来就是单个极简结果（例如单行 commit、单条命令、单个标题），就把“默认只输出这一项”写成硬规则，并在最终检查里主动删掉不必要的 body、解释、备选项。
- 如果目标 skill 是 Conventional Commit、单行命令这类极简输出，不要把 body 触发条件写成“有帮助时可加”。要写成更窄的闭集：通常只有明确的 breaking change、迁移/弃用说明，或用户显式要求更多上下文时才允许 body；普通补充细节、测试更新、第二个子动作都不够构成扩写理由。
- 如果目标 skill 属于 Conventional Commit、PR 标题压缩、changelog 单行这类“高压缩判型”输出，而且某个边界误判代价很高，就允许保留 1 个极短 canonical example 或一条写死的边界规则来钉住它。尤其是公开接口变更：旧 public CLI flag / option / env var / config key / API field 只要被拒绝、移除、重命名，或被新名字替代，就按 breaking interface change 处理；默认倾向 `feat(scope)!`，必要时再补一行迁移 body，不要降成普通 `fix`。
- 新建 skill 时，“是否启用记忆层”是必做判断，不允许跳过。即使最后结论是 `off`，也要给出一句理由（例如：低风险、低变异、可确定性执行）。

# 工作流程

## Step 1：先判断当前是什么任务

- 判断当前请求属于哪条路径：
  - 新建 skill
  - 修改现有 skill（含 `轻优化` / `结构重构` / `完整改造` 的前半程）
  - 评估输出质量
  - 优化触发行为（只在 skill 本体结构已经站住时进入）
  - 打包或交付 skill
- 进入这一步时，先设置：
  - `current_path` = 上面五种路径之一
  - `current_step` = `Step 1`
  - `next_action` = 用一句话说明接下来要做什么
- 如果用户是在安装当前 creator，而不是在修改别的 skill，默认推荐 `git clone https://github.com/DazhuangJammy/DazhuangSkill-Creator.git` 到目标 skill 目录；不要默认走手动复制。
- 如果本地脚本可用且已经进入真实执行，先按需读取 `<skill-base>/config.yaml` 里的 `update_check`，再运行 `<python-cmd> "<skill-base>/scripts/check_update.py" --json`。
- 如果脚本返回 `should_notify = true`，只简短说明：当前版本、最新版本、是仅提醒还是已自动更新，然后继续当前任务。
- 如果脚本返回 `status = updated`，明确告诉用户“本地文件已更新，但这次调用继续沿当前已加载版本执行；下次调用会使用新版本”。
- 只要用户这次是在说“评估某个东西”“测评某个东西”“测有无 skill 的差别”或“比较多个同类 skill”，就直接判到 `评估输出质量` 这条路径，并把 `next_action` 设成“先出评估前置提案，等用户拍板”；不要改判到结构判断/评审模式，也不要跳过到执行层。
- 如果用户还在探索或讨论阶段，而且没有出现评估 / 测评 / 评测意图词，就停留在结构判断/评审模式，不要强行进入实现或重型评测。
- 如果路径不清楚，先做最轻的结构判断，再决定是否继续下钻。
- 如果用户说的是“优化一个现有 skill”，默认先停在 `修改现有 skill`，不要直接跳进 `优化触发行为`。
- 先判断这次更像哪一种：
  - `轻优化`：skill 本体结构基本站得住，只需要补 `description`、局部规则、边界、示例，或小的 workflow 修补
  - `结构重构`：skill 太胖、太散、太难恢复方向，需要把主 body 和 bundled resources 重排到当前这套架构里
  - `完整改造`：既要做结构重构，也要做触发优化或评测；顺序默认是先结构、后触发

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DazhuangJammy/DazhuangSkill-Creator](https://github.com/DazhuangJammy/DazhuangSkill-Creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
