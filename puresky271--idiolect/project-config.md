---
trigger: always_on
description: > 给 AI 编码代理的项目说明。本文描述的是仓库实际内容，读者无需任何背景知识。
---

# AGENTS.md

> 给 AI 编码代理的项目说明。本文描述的是仓库实际内容，读者无需任何背景知识。

## 项目概览

**idiolect**（个人语型）：让 AI 说话像《BanG Dream! It's MyGO!!!!!》的五个角色（爱音、灯、立希、素世、乐奈），并且能**用数据证明「真的变像了」**。

方法：从原作台词里蒸馏可检验的风格特征（长度、句数、口癖、称呼、场景分类），把它们落成 system prompt，再用探针与机械门禁验证效果。**不做微调，不分发原作文本**——仓库只发布派生统计量（`data/` 下的聚合 JSON）。

核心入口：

```python
from idiolect.assemble import build_messages
messages = build_messages("乐奈", "你今天又想去哪找猫")
```

装配出四层 prompt，顺序固定（稳定前缀在前，动态块在后）：

| 层 | 内容 | 变化频率 |
|---|---|---|
| `canon` | 角色长档案 | 静态 |
| `voice` | 语气 manifest：句式、口癖、关系差异、反模板化硬约束 | 静态 |
| `style_target` | 说话尺度：字数/句数/句末/自称的可检验数字，命中场景时换成该场景的数字 | 每轮 |
| `turn_logic` | 本轮场景/主题指引，只在命中时注入 | 每轮 |

文档在 `docs/`（索引与阅读路径见 `docs/README.md`）：`00-methodology.md`（方法总纲）、`01-quickstart.md`、`02-corpus.md`（语料）、`03-features.md`（特征）、`04-evaluation.md`（指标口径）、`05-tooling.md`（工具手册，最实用）、`06-lessons.md`（踩坑清单）、`07-turn-logic-and-postprocessing.md`、`08-context-workspace.md`（四层在真实系统里的前后文）。

## 技术栈与运行环境

- 纯 Python 3.11+，有 `pyproject.toml`（`pip install .` 可装，`idiolect` 运行时零第三方依赖；CLI 入口 `python -m idiolect`）。CI 是 GitHub Actions（`.github/workflows/ci.yml`：离线 smoke 矩阵 + wheel 包数据校验）。测试通过 `conftest.py` 把仓库根挂上 `sys.path`（直接 `import idiolect.*`）。
- 依赖见 `requirements.txt`：`openai`、`numpy`、`jieba`、`requests`、`pytest`。重算语料派生统计才需要 `requirements-corpus.txt`（`sentence-transformers`、`scikit-learn`，模型权重首次运行时下载）。
- **Windows 上统一用 `py -X utf8`** 跑脚本；裸 `python` 可能解析到没装依赖的解释器。所有命令在仓库根目录执行。

## 目录与模块划分

```text
idiolect/            # 运行时包：把特征写成 prompt 约束
  assemble.py        #   四层装配器（核心入口，build_messages / build_system_prompt）
  registry.py        #   角色名 → 角色包的唯一分发点（含别名归一化）
  scene_classifier.py / scene_engine.py / general_scenes.py   # 场景分类与动态注入；scene_engine 另托管 turn_logic 共享脚手架（SessionStore 去重表 / SessionValues 值状态表 / env 假值表 / 深模块执行器）
  style_target.py / scene_length_targets.py                   # 长度/句数目标块
  workspace.py       #   上下文工作区（12 层装配 + 事实选择器），四层之外的骨架
  facts.py / tone.py / _text_rng.py   # 事实选择器 / 语气分类 / 文本定种 RNG（后处理随机步骤的确定性）
  characters/<key>/  #   五个角色包（anon/tomori/taki/soyo/rana）：
                     #     api.py（对外窄入口）、canon.py、voice.py、
                     #     turn_logic/（场景模块；其中的纯数据目录已迁 data/*.json，
                     #       经 importlib.resources 读取）、voice_check/（出站清洗）
tools/               # 工具链，按职责分子目录
  _paths.py          #   路径与环境变量的唯一实现，脚本不许自己拼路径
  offline_smoke.py   #   一条命令健康检查（不调 LLM，可进 CI）
  mock_clock.py      #   评测时钟
  secrets_loader.py  #   密钥加载
  distill/           #   从语料统计派生量（export_targets / tic_profile / scene_char_baseline …）
  gates/             #   机械门禁（voice_meta_gate、_tl_deep_check、oob_check、evidence_check、accept_check 验收门禁 …）
  probe/             #   探针（probe_runner、oob_probe 越界、multiturn_probe 多轮漂移、prompt_patch …）
  score/             #   评分（probe_report、scene_distill、_copy_audit、power_calc、ab_blind 盲评 …）
  corpus/            #   语料抓取与构建（需要 IDIOLECT_CORPUS_DIR）
data/                # 随仓库发布的派生统计（无原作文本），每个文件的生成脚本见 data/README.md
fixtures/            # 探针夹具（messages_<角色>.json；占位夹具 system 为空）
raw/gold/            # 语料默认位置（仓库不带语料）
report/              # 所有工具产物的默认输出目录
tests/               # pytest 单测
bootstrap.py         # 一条命令装成可跑状态（建 .venv → 装依赖 → 自检 → 打印一份 prompt）
.claude/skills/      # 给 agent 的流水线 skill（见下）
.github/workflows/   # CI（离线 smoke 矩阵 + wheel 包数据校验）
```

**访问角色包只走 `idiolect/registry.py`**（`get_canon_profile` / `get_voice_manifest` / `render_turn_special_block`），不要按角色名堆 if/elif，也不要直接 `import idiolect.characters.<key>.*`。名字表（含日文写法、简繁差异、常见误写）也只有 `registry._ALIASES` 一份，角色包里的 `is_<char>()` 问它，别自带名单。

`layer_sizes` 是不推进真实会话的诊断：使用 `scene_engine.isolated_session_state()`，在 ContextVar 隔离副本中观察 SessionStore/SessionValues。不得通过全局 reset 或事后回滚实现诊断，否则会丢失并发更新。此作用域限定同步使用，不是跨表原子快照；契约见 `tests/test_diagnostic_state.py`。

## 流水线 skill（`.claude/skills/`）

展示层在 `skills/mygo-five-roleplay/`，包含总 Skill 与五个可选的 `references/<角色>/` 目录。角色资料从 registry/style_target 导出，不手改生成文件：`py -X utf8 tools/export_roleplay_skill.py --out report/roleplay-release`；`--check --out skills/mygo-five-roleplay` 校验发布副本。静态 agent 扮演不等价于 Python 的动态路由、去重及后处理；核心评测流程保持独立。

五个 skill 把「换成别的角色」这件事拆成阶段，每个都带命令与验收条件；agent 会自动加载，人也可以当操作手册读：

| skill | 阶段 |
|---|---|
| `idiolect-corpus` | 语料：格式、切分、角色 key 决策、要跟着改的表清单 |
| `idiolect-distill` | 蒸馏：`data/` 六个 JSON + `scene_length_targets.py` |
| `idiolect-cast` | 角色包：canon/voice/turn_logic/voice_check + 注册 + 门禁 |
| `idiolect-evaluate` | 评测：probe、四件套、池化、功效、两臂 dump |
| `idiolect-pipeline` | 编排：交接物、每段门禁、自动／人写对照表 |

## 构建与测试命令

探针结果可用 `py -X utf8 tools/score/report_html.py --labels <批次>` 导出为 `report/evaluation.html`。浏览页只消费既有 JSONL/summary/scene_distill，不重新实现评分；模板在 `tools/score/report_template.html`。缺测与旧批次未知条件不能显示成零或通过。

```bash
python bootstrap.py                        # 一条命令：建 .venv + 装依赖 + 自检 + 打印 prompt
python bootstrap.py --no-tools             # 只要装配库（运行时零第三方依赖）
python bootstrap.py --skip-smoke           # 跳过自检

py -X utf8 -m pip install -r requirements.txt

# 健康检查（最常用，零 LLM、零写仓库文件）
py -X utf8 tools/offline_smoke.py          # 全套：装配 + 红线 + 数据 + 门禁 + 文档表格 + 单测 + 零写校验
py -X utf8 tools/offline_smoke.py --fast   # 跳过门禁与单测，一秒出结果

# 单测
py -X utf8 -m pytest tests/

# 查看某角色某句话的完整 prompt（不需要密钥和语料）
py -X utf8 tools/gates/dump_prompt.py --char 乐奈 --msg "你今天又想去哪找猫"
py -X utf8 tools/gates/dump_prompt.py --all --matrix

# 跑探针（需要 LLM_API_KEY / LLM_BASE_URL / LLM_MODEL）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puresky271/idiolect](https://github.com/puresky271/idiolect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
