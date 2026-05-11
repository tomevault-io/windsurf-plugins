---
trigger: always_on
description: > 目标：让 Codex（或同类代码生成模型）在最少追问的前提下自动完成需求、自测并保证可运行，交付可维护成果。<br>> Goal: Enable Codex (or similar code-generation agents) to autonomously deliver maintainable, runnable work with minimal back-and-forth, including self-testing.
---

# agent.md | Codex 执行代理规范（中英双语版）

> 目标：让 Codex（或同类代码生成模型）在最少追问的前提下自动完成需求、自测并保证可运行，交付可维护成果。<br>> Goal: Enable Codex (or similar code-generation agents) to autonomously deliver maintainable, runnable work with minimal back-and-forth, including self-testing.

---

## 0. 沟通与产出原则 | Communication & Deliverable Principles
1. **语言**：所有交流、注释、提交信息、README 默认使用简体中文，可附英文术语。<br>**Language**: Default to Simplified Chinese for communication, comments, commit messages, and README content; append English terms when helpful.
2. **默认动作**：遇到可合理假设的细节，应直接根据专业判断继续实现，并将假设记录在 `ASSUMPTIONS.md`。<br>**Default action**: When reasonable assumptions are possible, proceed with a professional default and record the assumption in `ASSUMPTIONS.md`.
3. **最小打扰**：除非风险极高，不等待额外确认；在交付物中标注可选项与替代方案。<br>**Minimise back-and-forth**: Avoid pausing for confirmation unless risk is high; document alternatives and options in the deliverables.
4. **可运行性**：生成的代码、脚本、配置必须可本地一键运行并通过测试。<br>**Run-ability**: Produced code, scripts, and configs must run locally via one-liners and pass tests.
5. **可复现性**：输出固定的环境说明与锁定依赖（如 `requirements.txt`、`poetry.lock`、`package-lock.json`）。<br>**Reproducibility**: Provide deterministic environment notes and lock dependencies (e.g., `requirements.txt`, `poetry.lock`, `package-lock.json`).
6. **安全优先**：默认不开启高危权限；对外部调用使用显式白名单和可配置开关。<br>**Security first**: Do not enable dangerous permissions by default; gate external calls behind explicit allow-lists and configurable switches.

---

## 1. 交付物目录结构（通用模板）| Suggested Deliverable Structure (Generic)

```
project-root/
├─ src/                         # 业务源码 | Core source code
├─ tests/                       # 单元/集成测试 | Unit/integration tests
├─ examples/                    # 最小示例 | Minimal runnable examples
├─ scripts/                     # 任务脚本 | Task automation scripts
├─ config/                      # 配置模板 | Configuration templates
├─ docs/                        # 设计/API/运维文档 | Design/API/Ops docs
├─ .github/workflows/           # CI 工作流 | CI workflows
├─ Dockerfile                   # 容器化运行 | Container runtime
├─ docker-compose.yml           # 本地依赖编排 | Local dependency orchestration
├─ pyproject.toml / package.json / requirements.txt
├─ Makefile                     # 一键任务入口 | One-command task entry
├─ README.md                    # 使用说明 | Usage guide
├─ ASSUMPTIONS.md               # 假设与权衡 | Assumptions & trade-offs
├─ CHANGELOG.md                 # 变更日志 | Change log
└─ agent_report.md              # 自动执行报告 | Automation run report
```

> 要求：在可行情况下保持上述结构，为团队提供统一入口。<br>> Requirement: Adopt the structure when feasible to give the team a consistent entry point.

---

## 2. 一键任务（Makefile 约定）| Makefile One-liner Conventions

```
make setup        # 安装依赖、初始化环境 | Install dependencies and bootstrap environment
make fmt          # 代码格式化 | Format code
make lint         # 静态检查 | Run linters / static analysis
make test         # 运行测试 | Execute test suite
make run          # 启动应用或示例 | Run the application/example
make build        # 构建产物 | Build distributables or images
make ci           # 本地模拟 CI：lint + test + build | Local CI: lint + test + build
```

> PR/交付前需确保 `make ci` 全绿；如无需 Docker，可用其他可发布产物替代。<br>> Ensure `make ci` passes before delivery; if Docker isn’t needed, substitute with another distributable artefact.

---

## 3. 需求到实现的自动流程 | Requirement-to-Implementation Workflow
1. **解析需求**：抽取功能点、接口、数据结构、约束、非功能需求，识别歧义并在 `ASSUMPTIONS.md` 中记录默认值。<br>**Requirement analysis**: Extract features, interfaces, data structures, constraints, and non-functional needs; capture ambiguities and assumptions in `ASSUMPTIONS.md`.
2. **制定方案**：输出模块边界、依赖、关键数据流或时序图，并在 `docs/decision_record.md` 说明取舍。<br>**Design**: Define module boundaries, dependencies, key data flows or sequence diagrams, and document trade-offs in `docs/decision_record.md`.
3. **脚手架落地**：搭建目录与基础文件，补齐 `README.md` 与一键运行指引；配置 `config/.env.example` 等模板。<br>**Scaffolding**: Lay out directories and base files, enrich `README.md` and quick-start instructions, and provide `config/.env.example` templates.
4. **实现编码**：遵循整洁代码，使用中文注释并为公共函数编写 docstring/示例。<br>**Implementation**: Write clean code with Chinese comments and docstrings/examples for public functions.
5. **自测优先**：为模块编写单元测试，关键流程补集成测试，目标覆盖率 ≥ 80%。<br>**Self-testing**: Create unit tests per module and integration tests for critical flows, targeting ≥80% coverage.
6. **质量闸门**：执行 `make fmt && make lint && make test`，生成覆盖率报告与构建产物。<br>**Quality gate**: Run `make fmt && make lint && make test`, produce coverage reports and build artefacts.
7. **交付收尾**：更新 `CHANGELOG.md`、扩充 `README` 常见问题，整理 `agent_report.md`。<br>**Delivery wrap-up**: Update `CHANGELOG.md`, expand the README FAQ, and prepare `agent_report.md`.

---

## 4. 代码与文档规范 | Code & Documentation Standards
- Python：使用 `ruff` + `black` + `mypy`（严格模式）。<br>Python: Adopt `ruff`, `black`, and strict `mypy`.
- TS/JS：使用 `eslint`（typescript-eslint）+ `prettier`。<br>TS/JS: Use `eslint` (typescript-eslint) plus `prettier`.
- Go：使用 `gofmt` + `golangci-lint`。<br>Go: Apply `gofmt` and `golangci-lint`.
- 日志需统一封装，默认不打印敏感信息。<br>Log through a unified wrapper and avoid printing sensitive data by default.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KittenCN/kl8-lottery-analyzer](https://github.com/KittenCN/kl8-lottery-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
