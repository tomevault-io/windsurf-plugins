---
trigger: always_on
description: 本文件是给在本仓库工作的 AI coding agents 使用的最佳实践说明。把它视为 `README.md` 的仓库专用伴随文档：`README.md` 面向人类讲产品故事；这里面向 agents 规定操作 contract。
---

# AGENTS.md

本文件是给在本仓库工作的 AI coding agents 使用的最佳实践说明。把它视为 `README.md` 的仓库专用伴随文档：`README.md` 面向人类讲产品故事；这里面向 agents 规定操作 contract。

## 项目概览

M2M2 是 Math-To-Manim 的重写：简短教育 prompt 会变成类型化 planning artifacts、生成的 Manim code、可选渲染、review outputs，以及可复现 run bundle。

核心承诺：先讲故事，再写符号；先做几何，再做代数；先落产物，再产生副作用。

Primary package: `math_to_manim`.
Primary CLI entry points: `m2m2` and `math-to-manim`.
Primary runtime path: `math_to_manim/pipeline/runner.py`.
Architecture reference: `docs/ARCHITECTURE.md`.
Human-facing landing page: `README.md`.

## Agent 操作原则

每次修改都遵循这些 Karpathy-inspired rules：

1. 编码前先思考。
   - 不要静默假设 requirements、architecture、file ownership 或 command behavior。
   - 当歧义会改变实现选择时，明确指出。
   - 只有真正被阻塞时才问澄清；否则选择最小安全解释并说明假设。
   - 当请求有明显复杂度、安全或产品含义时，说明 tradeoffs。

2. 简单优先。
   - 优先选择能满足请求的最小可维护改动。
   - 除非被要求，不要添加 speculative abstractions、宽泛 configurability、background services、新 frameworks 或“future-proofing”。
   - 如果方案变大，先停下寻找更小切口。

3. 外科式修改。
   - 只触碰任务必需的 files 和 lines。
   - 不要顺手重写 comments、formatting、docs 或相邻 code。
   - 匹配正在编辑文件的现有风格。
   - 只有当你的改动让 imports/functions/files 变成 unused，或用户明确要求 cleanup 时，才删除它们。
   - 无关 dead code 在 final notes 中提到，不要擅自删除。

4. 目标驱动执行。
   - 编辑前定义 success criteria。
   - 对 bugs，实践可行时先复现失败或加 failing test。
   - 对 features，实践可行时围绕行为变更加或更新 tests。
   - Final response 前用精确 commands 验证。

## 仓库布局

- `math_to_manim/agents/` — intent、graph、curriculum、math、storyboard、scene spec、codegen、static review、render、video review 和 publishing 的 stage adapters。
- `math_to_manim/schemas/` — Pydantic artifact contracts。把它们视为 public pipeline interfaces。
- `math_to_manim/pipeline/` — orchestration、tracing、state 和 repair loop 行为。
- `math_to_manim/tools/` — graph work、AST/static validation、scene discovery 和 artifact storage 的 deterministic helpers。
- `math_to_manim/rendering/` — Manim、FFmpeg 和 render command wrappers。
- `math_to_manim/providers/` — provider-specific integrations，例如 Codex CLI bridge。
- `math_to_manim/app/` — 可选 API/UI surfaces。
- `tests/unit/` — 当前自动化 test suite。
- `docs/` — architecture、docs index、showcase 和 visual documentation assets。
- `docs/showcase/assets/` — 有意跟踪的 legacy showcase GIFs，用作 art-direction targets。
- `scripts/` — render dependency bootstrap 等 operational helper scripts。
- `runs/` — generated run bundles；已 ignored，通常不 commit。

## Setup commands

存在本地 virtual environment 时优先使用：

```bash
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

macOS/Linux/WSL fresh checkout：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

Windows PowerShell fresh checkout：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

只有任务需要真实 Manim rendering 时才安装 render extras：

```bash
python -m pip install -e ".[dev,render]"
./scripts/bootstrap-render.sh  # Debian/Ubuntu/WSL system deps: FFmpeg, LaTeX, etc.
```

## Verification commands

结束前运行最快的相关检查。优先使用 venv-qualified form，避免依赖 shell activation：

```bash
./.venv/bin/python -m pytest
./.venv/bin/python -m math_to_manim.cli --help
./.venv/bin/python -m math_to_manim.cli generate --help
./.venv/bin/python -m math_to_manim.cli generate "Explain why derivatives are slopes" --deterministic --no-render --runs-dir /tmp/m2m2-smoke
```

如果 CLI entry points 已安装在 active environment，以下等价命令也应可用：

```bash
m2m2 generate "Explain why derivatives are slopes" --deterministic --no-render
math-to-manim generate "Explain why derivatives are slopes" --deterministic --no-render
```

对 codegen-provider 工作，在责怪 M2M2 前单独验证 Codex：

```bash
codex --version
codex exec "Say ready from inside this repo"
```

对 render 工作，只有在 render dependencies 已安装后才运行小型 render-quality smoke。如果 full render 太慢或不可用，运行 deterministic no-render 和相关 unit tests，并明确报告跳过 render 的原因。

## Pipeline contracts

正常生成会在 `runs/<run_id>/` 下写 run bundle，包含：

- `request.json`
- `intent.json`
- `knowledge_graph.json`
- `curriculum.json`
- `math_packet.json`
- `storyboard.json`
- `scene_spec.json`
- `generated_code.json`
- `generated_scene.py`
- `validation_report.json`
- `render_result.json`
- `review_report.json`
- `animation_package.json`
- `manifest.json`

规则：

- 除非任务明确是 schema/pipeline migration，否则保留 artifact names。
- 如果修改 schema，更新所有依赖它的 producers、consumers、tests 和 docs。
- Deterministic mode 必须保持 offline 且 reproducible。
- Rendering 必须由 static validation gate 控制；validation failure 不应调用 Manim。
- Repair loops 应基于冻结的 upstream `scene_spec` 和记录的 stderr/stdout 运行，而不是重新跑完整 planning。

## Code style 和架构

- Python 3.10+。
- 使用 Pydantic models 表达 artifact boundaries。
- Provider-specific behavior 留在 stage runners/providers 后面；不要把 OpenAI、Anthropic、Gemini、Kimi 或 Codex 假设泄漏到 schemas 中。
- 对 validation、graph operations、filesystem packaging 和 command construction，优先使用 pure functions 和 deterministic helpers。
- Stage outputs 应保持为可检查 JSON。
- Errors 要 actionable：可用时包含 command、artifact path、stderr summary 和 stage。
- 避免 pipeline runner 中的 hidden parallelism；文档化 runtime shape 是 single-threaded and ordered。
- 不要绕过 static review 让 rendering “work”。修复 generated code 或 validator contract。

## Testing guidance

- 行为变更要在 `tests/unit/` 中添加或更新 tests。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HarleyCoops/Math-to-Manim-CH](https://github.com/HarleyCoops/Math-to-Manim-CH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
