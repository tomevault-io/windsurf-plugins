---
trigger: always_on
description: **Generated:** 2026-03-25T12:39:50Z
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-25T12:39:50Z
**Commit:** ee9bbf6
**Branch:** master

## OVERVIEW
Python 3.11 CLI package that turns browser workflows into generated site-specific commands.
Core flow: inspect Chrome via CDP + AXTree, ask an LLM to plan actions, emit Click adapter code into `~/.cliany-site/adapters/`, then replay actions with JSON envelopes.

## STRUCTURE
```text
./
├── src/cliany_site/        # runtime package; CLI, browser, explorer, codegen, adapter loader
├── qa/                     # shell-based integration checks; no pytest suite
├── docs/walkthroughs/      # change notes / troubleshooting writeups
├── pyproject.toml          # package metadata; `cliany-site` console entry point
├── uv.lock                 # locked Python dependency graph
└── README.md               # user workflow, env setup, operational examples
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| CLI entry / command wiring | `src/cliany_site/cli.py` | `SafeGroup`, root `--json`, adapter registration |
| Built-in commands | `src/cliany_site/commands/` | `doctor`, `login`, `explore`, `list` |
| Browser / CDP connectivity | `src/cliany_site/browser/cdp.py` | assumes Chrome on `localhost:9222` |
| AXTree capture / selector map | `src/cliany_site/browser/axtree.py` | truncates long trees; emits `@ref`-style selector map |
| Workflow planning | `src/cliany_site/explorer/engine.py` | env loading, LLM setup, exploration loop |
| Prompt contract | `src/cliany_site/explorer/prompts.py` | strict JSON shape and URL rules |
| Generated adapter emission | `src/cliany_site/codegen/generator.py` | renders Click code + metadata.json |
| Dynamic adapter loading | `src/cliany_site/loader.py` | loads generated `commands.py` from home dir |
| Session persistence | `src/cliany_site/session.py` | stores cookies under `~/.cliany-site/sessions/` |
| Action replay / fuzzy element resolution | `src/cliany_site/action_runtime.py` | executes click/type/select/navigate/submit |
| QA coverage | `qa/*.sh` | shell suites; integration-oriented |

## CODE MAP
| Symbol | Type | Location | Refs | Role |
|--------|------|----------|------|------|
| `CDPConnection` | class | `src/cliany_site/browser/cdp.py` | 10 | shared Chrome/CDP entrypoint for doctor, login, explore |
| `WorkflowExplorer` | class | `src/cliany_site/explorer/engine.py` | 5 | LLM-guided workflow discovery loop |
| `AdapterGenerator` | class | `src/cliany_site/codegen/generator.py` | 6 | turns `ExploreResult` into generated Click adapters |
| `execute_action_steps` | function | `src/cliany_site/action_runtime.py` | 3 | replays recorded actions against live pages |
| `register_adapters` | function | `src/cliany_site/loader.py` | 3 | mounts generated domain commands on root CLI |
| `save_session` | async function | `src/cliany_site/session.py` | 3 | persists cookies from browser session |

## CONVENTIONS
- src-layout package only: import from `cliany_site.*`; no local path hacks or aliases.
- CLI commands are synchronous Click functions that call internal async helpers via `asyncio.run(...)`.
- Root `--json` flag propagates through `ctx.find_root().obj`; user-facing commands should respect it.
- Responses use the shared envelope from `response.py`; success/error payloads stay machine-readable.
- User-facing help, docstrings, and status text are predominantly Chinese; keep tone consistent.
- Environment resolution is layered: XDG config `.env` → legacy `~/.cliany-site/.env` → project `.env` → real environment.
- OpenAI-compatible base URLs are normalized to include `/v1`; do not duplicate normalization logic elsewhere.

## AUTONOMOUS IMPROVEMENT GUARDRAILS

> 适用于所有自主改进循环中的 agent 行为约束。

- **坚守语义化**：禁止引入脆弱 CSS 选择器作兜底，必须基于 AXTree 语义模糊匹配（`selector_map` 语义 role/name 查找）。
- **零数据污染**：生成的 adapters/sessions/snapshots 严格隔离在 `~/.cliany-site/`；测试必须使用 `tmp_home` fixture，禁止在 repo 内 write 运行时状态。
- **codegen 规范**：修改 `generator.py` 须保证生成的 adapter 代码符合 type hints，兼容 Python 3.11+；禁止生成 `eval`/`exec`/`os.system` 等危险调用。
- **自主修复边界**：CI workflow 只搭脚手架（触发器 + 指令文档），不内联 opencode/agent 调用链；真正修复由 OpenCode 读取 `.github/AUTONOMOUS_FIX.md` 执行。
- **PR 门禁零真实 LLM 密钥**：所有 PR 触发的 job 必须设置 `CLIANY_QA_OFFLINE=1` 走确定性回归；受保护 workflow 可用 GitHub Secrets，仅限非 fork-PR 触发。
- **禁止重写现有基础设施**：只允许扩展/新增，禁止重写 `.github/workflows/ci.yml` 现有 jobs、`tests/conftest.py`、`src/cliany_site/testing/snapshot.py`；已标记 `# 自动生成 — DO NOT EDIT` 的文件绝对禁止修改。

## ANTI-PATTERNS (THIS PROJECT)
- Do not edit generated adapter code marked `# 自动生成 — DO NOT EDIT`.
- Do not assume generated adapters or session files live in this repo; runtime state lives under `~/.cliany-site/`.
- Do not fabricate navigation URLs during exploration; prompt contract explicitly forbids guessed URLs.
- Do not add pytest/CI assumptions to contributor docs; current validation path is shell scripts in `qa/`.
- Do not commit cache/build artifacts; repo already treats generated and transient files as out-of-scope.

## UNIQUE STYLES
- Built-in CLI plus runtime-loaded per-domain subcommands.
- AXTree-driven selector metadata enables fuzzy action replay after minor page changes.
- `doctor` reports environment health as structured JSON rather than plain logs.
- Generated adapters mirror root CLI conventions: root-aware `--json`, shared error codes, CDP/session preflight.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pearjelly/cliany.site](https://github.com/pearjelly/cliany.site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
