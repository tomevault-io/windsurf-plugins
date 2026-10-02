---
trigger: always_on
description: 给代理（Claude Code、Codex 等）看的索引和工作手册，公开仓库也带着——别人的代理照它干活。给人看的贡献规则在 [CONTRIBUTING](CONTRIBUTING.md)。结构：**一、索引**（读什么、按什么顺序）→ **二、硬规则** → **三、工作流** → **四、参考**。与别处冲突时以「效率硬规则」为准，其次本文件。
---

# HSL Remake Agent Guide

给代理（Claude Code、Codex 等）看的索引和工作手册，公开仓库也带着——别人的代理照它干活。给人看的贡献规则在 [CONTRIBUTING](CONTRIBUTING.md)。结构：**一、索引**（读什么、按什么顺序）→ **二、硬规则** → **三、工作流** → **四、参考**。与别处冲突时以「效率硬规则」为准，其次本文件。

写成代码样式、以 `docs/internal/` 或 `docs/audits/` 开头的路径是维护者的内部文档，只在私有仓库里有，公开导出不带；公开仓库里的代理跳过它们即可。

---

## 一、索引

### Mission

本项目在 Godot 4.x 中重制《幻世录》第一章。终点两个，按顺序：①第一章 127 场战斗像原版一样从标题玩到章末；②这套引擎能写续集——没有原版数据可导入时，关卡／角色／技能／剧情只靠写数据就能跑。原版资源、脚本、EXE 静态分析和原版运行实测用于恢复规则；目标是自己的可维护游戏工程，不是自动操纵原作，也不是把截图或 readback 当产品。

交付范围、当前缺口与下一步以 [docs/PROJECT.md](docs/PROJECT.md) 为准。默认一律照原版（界面选择全部照原版；选项系统默认原版预设）；讲得清的改良只做成「重製選項」里的选项（[OPTIONS](docs/OPTIONS.md)），门禁与裁判只跑原版档。原版等价声明须逐项证据支持。

### 冷启动

所有任务按顺序读：

1. `AGENTS.md`（本文件；「效率硬规则」必读）
2. [`docs/PROJECT.md`](docs/PROJECT.md)（唯一当前状态：现状、进度尺、1.0 的条件）
3. 承接 lane 时：负责人给的任务书；格式见 `docs/internal/lane_brief.md`

未知 Git 改动先确认归谁，不动别人的。历史用 Git 查（`git log -S`）；逐轮流水在 `docs/internal/ROUNDS.md`，更早的考古用仓库外的清理前 Git bundle；不要把历史文件恢复成任务入口。

### 按任务补读

| 本次任务 | 增量阅读 |
| --- | --- |
| 改 `game/`、scene 或 tests | [架构入口](docs/ARCHITECTURE.md)，再只读命中的 [战斗系统](docs/architecture/BATTLE_SYSTEMS.md)／[表现合同](docs/architecture/PRESENTATION.md) 章节 |
| 选定向测试、排查验证 | [测试路由](tests/README.md)、[工具入口](tools/README.md) |
| 资源、静态分析或原作对照 | [知识索引](docs/KNOWLEDGE_INDEX.md) 定位对应 packet；[METHOD](docs/METHOD.md#证据分级) 核对证据等级 |
| 改机制状态或等价声明 | [机制矩阵](docs/MECHANICS_EVIDENCE_MATRIX.md)、[差异清单](docs/evidence_packets/static_reverse/parity_gap_inventory.md) 及对应证据 |
| 改游戏：换素材、加关卡、改剧情流转、换配乐、加脚本 opcode 表现 | [MODDING](docs/MODDING.md)＋[加关卡与角色逐步表](docs/MODDING_LEVELS.md)（文件位置、生成命令、改代码入口、验证步骤）＋[战役总览](docs/evidence_packets/resource_inventory/campaign_overview.md) |
| 写外传、续集或新剧情 | [原作剧情简报](docs/ORIGINAL_STORY.md)（世界、人物、三个结局、原作留白）；逐句台词读 `docs/internal/ORIGINAL_SCRIPT.md`，不必再从导入件抽取 |
| 设计续集的人物、数值与关卡 | [原作人物名录](docs/ORIGINAL_CAST.md)、[规则数值手册](docs/NUMBERS.md)、[原作关卡](docs/ORIGINAL_LEVELS.md)、[战棋设计方法](docs/SRPG_DESIGN.md)（末节「落到我们的规则」是对到本作的结论），直接读，不重做调查 |
| 提到某一场战斗 | [战斗称呼对照](docs/BATTLE_NAMES.md)（见「命名口径」） |
| 派出或承接一条 lane | 下文「三、工作流」＋`docs/internal/lane_brief.md` |
| 开新机制找原函数、筛长文档、自检证据用语 | TypeSafe Jev 用法（完整文档是私有仓库的第三方镜像；要联网和 key；配方见[工具说明](tools/README.md#typesafe-判断分担与全-exe-函数目录)）：冷启动路由 `jevgrep rank`、证据用语 lint `jevgrep lint --rules tools/typesafe/evidence_lint_rules.json --diff HEAD`、全 EXE 函数候选目录 `PYTHONPATH=tools python3 -m hsltools.checks.function_catalog query`；模型判断只是路由候选，不是证据 |
| 全部文档怎么分工 | [文档地图](docs/README.md) |

视觉／交互对照再读 `docs/evidence_packets/runtime_observations/original_gameplay_reference/README.md`，按主题看原帧，代码入口查 ARCHITECTURE 的任务路由。外部模型解释是待审查线索；用 manifest 的源帧号定位，勿把导出编号、单次录像或压缩像素升级为全局规则。参考图存在不等于游戏已修复。

### Current truth

正式入口：

```text
project.godot
→ game/title/TitleScreen.tscn（原版標題畫面：開始新故事／戰場記錄／離開遊戲）
→ game/battle/scene/BattleSceneRuntime.tscn
→ BattleSceneRuntime.gd
→ BattlePlayLoop.gd
```

当前场景配置：`content/battles/campaign.json`（`start_level "51"` → `battle_051.json`，由 `level_battle:51` 生成；`BattleSceneRuntime.tscn` 仍可直接启动，默认加载同一文件，测试与开发路线不经标题）。`first_battle.json` 是名册模板与测试夹具，不是现行场景。

机器可读的权威数据：`content/imported/hsl/` 是可复用的原版资源、脚本 IR 与 manifest；`content/generated/hsl/` 是可重现的生成事实；`content/authored/` 是手写数据（续集关卡与角色、选项注册表等）；原作视觉基准是 `docs/evidence_packets/runtime_observations/first_battle_visual_evidence_index.json`；完整资源成员索引 `docs/evidence_packets/resource_inventory/resource_manifest.json` 供机器搜索，不要整文件读进上下文。

`BattlePlayLoop` 是唯一可变战斗状态所有者。Scene、`_unit_grid_coords` 和 `ActorRuntime` 只是输入/表现镜像。禁止新增第二套 battle dictionary、bootstrap snapshot 或 UI-owned combat truth。

玩家菜单以 `BattlePlayLoop.IMPLEMENTED_COMMANDS` 为准；新增命令时同时接通玩家交互与验证。未实现命令保持隐藏，直接调用返回 `not_implemented`。

---

## 二、硬规则

### 效率硬规则

干活记步骤时间；非必要不测试、不跑门禁、不做占时间的活；按最高性价比干活。与下文冲突时以本节为准。

- **记步骤时间**：开工记 `date`，每步（探路／实现／调试／验证／提交）记起止；lane 报告 ⑤ 写每步分钟、总墙钟、工具调用次数和最花时间的一步为什么；负责人把它连同派出／交回时刻、门禁模式与秒数记进时间账 `docs/internal/LANE_TIMELOG.md` 一行，每轮收口据此砍固定开销（见「时间账」）。
- **非必要不测试**：只按下文「测试政策」的两种情形写测试；验收只做任务书写明的 oracle——不自加负例、一次性探针、场景冒烟、手跑单测、逐字节复现证明、顺手的文档段落；不截图，除非是视觉改动（最多 3 张）。
- **非必要不门禁**：lane 收尾只跑一次 `tools/lane_verify.sh affected <基线>`（前台跑、timeout 给够，不 sleep 轮询），**不跑快门 `tools/verify.sh`**；不跑没命中的套件，不重生成没变的生成物；完整门禁只由负责人在合并树跑，几条 lane 一起交回只跑一次，由 `tools/lane_merge.sh gate` AUTO 按改动范围选档（见「门禁」）。
- **任务书写准再派**：负责人写明入口文件／函数、可照抄的先例和时间预算（小改 20／单条规则 45／含裁判实验 60 分钟，到点先交报告），不写与本文件冲突的条目（09-26 MUSIC-IMPORT 任务书写了提交 `.import`，与仓库规则冲突，lane 为此多探路；62 分钟墙钟里命令只占 10 分钟）。
- **少轮次**：能合并的读、查、跑合成一条命令——时间主要花在一问一答的轮次上，不在命令本身。
- **砍自证性工作**：每轮收口做一次效率审计（门禁失败原因与耗时、每条 lane 固定开销、测试与文档增长）。依据 09-25 审计：16 次门禁 202 分钟中 54% 花在失败门禁、4 次为结果文件逐字节过期、0 次拦到真规则错。

### 测试政策

测试非必要不写：只有不写就会出问题时才写。

- 测试只在两种情形写：①钉的是原版量得的事实（static-derived／模拟器实测），且没有现成套件覆盖；②不写就会让门禁抓不到会伤玩家的回归。其余一律不写。
- 重制自己随机流的产物（开场等级数组、某个种子下的流状态、动作计数）**不钉数值**，只断言不变量；不为"消融能变红"而加测试；不为机械小改加复述实现的测试。
- 一条 lane 默认不新增测试文件，优先在现有套件里加一两个代表性用例；删掉多余的测试。任务书验收只写玩家可见结果与原版对照结果行。
- 不为让测试过而改断言；测试绿只证明当前合同，不证明原版等价。

### 命名口径

- **战斗**：汇报、任务书、合并说明、PROJECT.md 里提到任何一场战斗，一律写「玩家第 N 场 · 场景名（LEVEL0xx）」＋在做什么，文件号只放括号；对照表 [docs/BATTLE_NAMES.md](docs/BATTLE_NAMES.md)。lane 报告只写文件号的，负责人转述时换成玩家口径。
- **对负责人汇报用玩家口径**：负责人要看的不是编号；需要拍板的事集中、带例子、带推荐，不阻塞。
- **不写死会随工作推进变化的数**（条目数、探针数、模块数、文件行数）；进度数字只写在 PROJECT「进度尺」并带日期与出处。

<a id="evidence-language"></a>

### 证据用语


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catoncat/hsl-remake](https://github.com/catoncat/hsl-remake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
