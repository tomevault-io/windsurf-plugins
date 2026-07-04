---
trigger: always_on
description: > 本文件供 Codex / Claude Code / Cursor / Devin / OpenCode / Gemini 等 AI agent 自动读取。
---

# UZI-Skill · Agent 指令

> 本文件供 Codex / Claude Code / Cursor / Devin / OpenCode / Gemini 等 AI agent 自动读取。

---

## 🗺️ Repository Layout & Entrypoints (v3.2.0)

**绝对路径约定 —— 不要自己瞎猜** · 避免 "scripts/run.py 缺失" 这类误解：

```
UZI-Skill/                                  # ← 你 cwd 应该是这里
├── run.py                                  # ✅ 用户入口 · CLI 直跑 (python run.py <ticker>)
├── AGENTS.md / CLAUDE.md / GEMINI.md       # agent 指令
├── .claude-plugin/plugin.json              # Claude Code manifest
├── .cursor-plugin/plugin.json              # Cursor manifest
├── gemini-extension.json                   # Gemini manifest
├── package.json                            # OpenClaw / npm
└── skills/deep-analysis/
    ├── SKILL.md                            # skill 描述
    ├── assets/                             # HTML 模板 / avatars / icons
    └── scripts/                            # ← 所有 Python 业务代码在这里
        ├── run_real_test.py                # legacy stage1/stage2 入口 (v3.1 瘦身后 735 行)
        ├── assemble_report.py              # HTML shell 组装 (v3.2 瘦身后 587 行)
        ├── fetch_*.py (22 个)              # 数据采集 · 也是独立 CLI (python fetch_basic.py <ticker>)
        ├── compute_*.py                    # 机构建模 (DCF / BCG / Porter)
        ├── tests/                          # 332 pytest
        ├── .cache/<ticker>/                # 跑过的股票缓存
        ├── reports/<ticker>_<date>/        # 生成的 HTML 报告
        └── lib/
            ├── pipeline/                   # 🆕 v3.0 管道式架构（默认路径）
            │   ├── run.py                  # run_pipeline 编排入口
            │   ├── collect.py              # 并发 collector (22 fetcher adapter)
            │   ├── score.py                # scoring 段 (调 rrt 纯函数)
            │   ├── synthesize.py           # stage2 薄 wrapper
            │   ├── score_fns.py            # 🆕 v3.1 · 1228 行纯函数
            │   ├── preflight_helpers.py    # 🆕 v3.1 · 网络/ticker preflight
            │   ├── fetchers/registry.py    # 22 adapter 工厂
            │   └── renderer/               # 21 个 renderer stub (未完全使用)
            ├── report/                     # 🆕 v3.2 · assemble_report 拆分
            │   ├── svg_primitives.py       # 19 个 svg_* + COLOR_*
            │   ├── dim_viz.py              # 19 个 _viz_xxx + DIM_VIZ_RENDERERS
            │   ├── institutional.py        # DCF/LBO/IC memo/catalyst/competitive
            │   ├── panel_cards.py          # 66 评委 panel 渲染
            │   └── special_cards.py        # fund/insights/school_scores/debate
            └── ...（其他 lib 模块 · investor_db / network_preflight / ...）
```

### 入口 Cheat Sheet

| 操作 | 命令 |
|---|---|
| 用户一句话分析 | `python run.py <ticker>` (repo root · 走 v3.0 pipeline) |
| 强制老路径 (保险) | `UZI_LEGACY=1 python run.py <ticker>` |
| 只跑单个 fetcher | `cd skills/deep-analysis/scripts && python fetch_basic.py <ticker>` |
| 跑全量 pytest | `cd skills/deep-analysis/scripts && pytest tests/ -q` |
| Python 环境 | 任意装了 akshare/pytest 的 Python 3.10+（macOS 系统 `/usr/bin/python3` 通常缺这些 · 用 `pip install -r requirements.txt` 装；conda / venv / pyenv 都行） |

### 内部模块调用约定

- Python 模块路径起点 = `skills/deep-analysis/scripts/` · `run_real_test.py` 顶部 `sys.path.insert(0, str(HERE))` 注入
- `from lib.pipeline.score import score_from_cache` · 不是 `from skills.deep_analysis.scripts.lib...`
- `run_real_test` 对外简称 `rrt` · pipeline 调它的纯函数 (`rrt.score_dimensions` → 实际来自 `lib.pipeline.score_fns`)

### 版本分水岭

| 版本 | 变化 | 影响 agent 的部分 |
|---|---|---|
| v3.0.0 | pipeline 默认启用 · `UZI_LEGACY=1` 回老路径 | `python run.py` 默认进 pipeline |
| v3.1.0 | rrt 瘦身 65% · 纯函数搬到 score_fns | 所有 `rrt.XXX` 仍向后兼容 (re-export) |
| v3.2.0 | assemble_report 瘦身 80% · 拆 5 个 lib/report/* | 所有 `assemble_report.XXX` 仍向后兼容 |

**黄金规则**：外部 test / lib 仍可以 `import run_real_test; rrt.score_dimensions(...)` · 不用改。拆分对上层透明.

---

## 你是谁

你是一个股票深度分析 agent。用户给你一只股票，你要**采集数据 → 亲自分析每个投资者的判断 → 生成报告**。

## 核心原则

**你不是脚本运行器——你是首席分析师。** 脚本只是你的工具。

66 个投资大佬的评审必须由你 role-play，不是纯跑规则引擎：
- 巴菲特看 ROE 和护城河，但他实际持有苹果 → 这比规则更重要
- 游资只做 A 股 → 分析美股时直接跳过
- 木头姐看颠覆创新 → 给她白酒股她会说"不在平台里"

## 深浅两套路径 · 按用户意图选一条（v3.2.0）

用户一句话只说"分析 XXX"**不一定**等于要跑全量 agent 流程。先做判断：

| 用户信号 | 推荐路径 | 耗时 | 为什么 |
|---|---|---|---|
| "快速看看"、"先扫一眼"、`/quick-scan`、`/thesis` | **CLI 直跑 lite** | 30-60s | 7 维核心数据 + 10 投资者，脚本直接出报告 |
| 明确要求"深度分析"、"估值"、"DCF"、"首次覆盖"、`/ic-memo`、`/initiate` | **全量 agent 流程** | 5-10min | 22 维 + 66 评委 role-play + agent_analysis.json |
| 未明确 | **默认 medium + CLI 直跑**（仍出完整报告） | 2-4min | v2.10.5 起 CLI 直跑 medium 也能完整出 HTML |

**关键**：从 v2.10.4 起，`run.py` 直跑模式下 `agent_analysis.json` 缺失会自动降级为 warning，**不会 block HTML 生成**。不要为了"跑一个完整流程"强行 role-play 66 评委——那是用户要求"深度"时才需要。

### 路径 A · CLI 直跑（快速）

```bash
python3 run.py <ticker> --depth lite --no-browser    # 最快
python3 run.py <ticker> --depth medium --no-browser  # 默认完整度
python3 run.py <ticker> --school F --no-browser      # v3.5.0 · 只看 F 派（游资）视角
python3 run.py <ticker> --school A --depth deep      # 价值派视角的深度分析
```

**v3.5.0 `--school` 参数**：用户可锁定单一流派 (A价值/B成长/C宏观/D技术/E中国价投/F游资/G量化/H科技领袖派/I Serenity 卡位猎手)，
其他派评委自动 skip · 报告顶部渲染 SCHOOL LOCK banner · 你 role-play 时**只 role-play 该派 5-8 人** ·
panel_insights / debate_rounds 都限于该派内部分歧。详见 SKILL.md `HARD-GATE-SCHOOL-LOCK`。

脚本会：
1. 跑 stage1 采集数据
2. 自检 self-review（CLI 模式下 agent_analysis 缺失是 warning）
3. 调 stage2 组装 HTML 报告

**你只需**：读最终 HTML / synthesis.json，向用户汇报核心结论。**不需要** role-play 66 评委。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gosinkx/UZI-SKILL-astock](https://github.com/gosinkx/UZI-SKILL-astock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
