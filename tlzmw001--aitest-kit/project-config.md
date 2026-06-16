---
trigger: always_on
description: 1. 说明这个仓库是什么、主要目录做什么、推荐按什么工作流协作
---

# AIAutoTest 协作指南

本文件适用于当前目录及其所有子目录。

它的目标有两层：

1. 说明这个仓库是什么、主要目录做什么、推荐按什么工作流协作
2. 约束 AI 在本仓库中的编码、调试、验证和安全行为

## 项目定位

`AIAutoTest` 是一个 AI 驱动的自动化测试项目，围绕一条稳定的测试飞轮展开：

`开发文档 -> 测试知识库 -> Markdown 用例 -> codegen -> pytest 执行 -> 修正与沉淀`

仓库中同时包含：

- `aitest-kit` 工具代码
- workspace 初始化模板
- AI skills 与协作规则
- 内置示例/验证目标：`coupon_system/`、`ab_experiment_sdk/`
- 本仓自用的测试资产工作区 `test_workspace/`

在这个仓库里协作时，优先遵守这条飞轮，而不是绕过文档、知识库和 Markdown 用例，直接产出零散 pytest。

## 仓库结构

- `coupon_system/`
  待测系统：智能优惠券推荐策略服务，包含 FastAPI、gRPC、Redis 与相关数据层逻辑。

- `ab_experiment_sdk/`
  AB 实验服务与 SDK。默认视为待测/依赖服务，不为测试便利随意修改。

- `docs/`
  开发文档输入目录，通常是文档审查、知识构建、测试设计的起点。

- `test_workspace/`
  AI 生成测试资产的工作目录。新链路优先使用 `targets/`、`suites/`、`generated/`、`reports/`；历史兼容目录可能仍存在，例如 `cases/`、`tests/`、`backups/`。

- `test_workspace/knowledge/`
  测试知识库目录，包含 L0/L1/L2 以及 `TEST_SPEC` 等内容。

- `test_workspace/targets/`
  按目标系统组织 target/module registry、模块 fixture、helper 和 module profile。

- `test_workspace/suites/`
  按目标系统组织独立 suite；suite 通过 `suite.yaml` 绑定 target/module。

- `test_workspace/generated/`
  按目标系统保存 codegen 生成的 pytest 文件，视为编译产物。

- `test_workspace/reports/`
  测试执行报告目录，包含 `result.json`、`report.md` 和 JUnit XML。该目录是运行产物，不入库。

- `test_workspace/results/`
  待测系统 bug、测试发现和执行结果记录。

- `test_workspace/plans/`
  测试方案、规划、spec 与阶段性设计文档目录。

- `aitest_kit/`
  Python 测试工具库，包含 parser、emitter、CLI、HTTP/gRPC 客户端与断言能力。

- `aitest_kit/templates/project_workspace/`
  新项目 workspace 的包内唯一模板源。包含干净 `aitest_config/`、`test_workspace/`、`AGENTS.md`、`CLAUDE.md` 和 agent-neutral `skills/`。模板内只维护 `skills/` 一份源目录，不维护 `.codex/.claude/.agents` 三份副本；不要再新增顶层 `templates/project_workspace/` 镜像。

- `tests/`
  `aitest-kit` 自身的单元测试与集成测试。

- `examples/`
  面向用户的示例项目或示例 workspace。

- `plugins/`
  Codex plugin 方向的实验和产品化资产。

- `aitest_config/`
  项目级配置目录。

- `aitest_config/aitest.yaml`
  统一配置入口，包含 workspace 路径和 codegen 默认规则。

- `.claude/skills/`
  Claude Code Skill 定义。本仓开发时实际使用的 agent skill 目录之一，不是 `aitest init` 模板的三份副本。

- `.codex/skills/`
  Codex 原生本地 skill 定义。Codex 协作时优先使用这里的同名 skill。本仓开发时实际使用的 agent skill 目录之一，不是 `aitest init` 模板的三份副本。

- `.agents/skills/`
  agents 工作流的 skill 定义。本仓开发时实际使用的 agent skill 目录之一，不是 `aitest init` 模板的三份副本。迁移或同步本仓 skill 时，保持 `.claude/skills/`、`.codex/skills/`、`.agents/skills/` 三处语义一致。

## 常用命令

除非用户明确要求别的方式，否则优先使用项目已有命令。

```bash
pip install -e ".[dev,server]"
python -m coupon_system.main
python3 -m aitest_kit.cli init --target /path/to/aitest_workspace
python3 -m aitest_kit.cli doctor --workspace /path/to/aitest_workspace
python3 -m aitest_kit.cli codegen --suite-file test_workspace/suites/<target>/<suite>/suite.yaml --validate-profile
python3 -m aitest_kit.cli codegen --suite-file test_workspace/suites/<target>/<suite>/suite.yaml --check
python3 -m aitest_kit.cli codegen --suite-file test_workspace/suites/<target>/<suite>/suite.yaml --health-report --write-report
python3 -m aitest_kit.cli registry register-suite --target <target> --module <module> --suite-file test_workspace/suites/<target>/<suite>/suite.yaml
python3 -m aitest_kit.cli task create --name <task_name> --suite-file test_workspace/suites/<target>/<suite>/suite.yaml
python3 -m aitest_kit.cli run --suite-file test_workspace/suites/<target>/<suite>/suite.yaml
python3 -m aitest_kit.cli run --suite-file test_workspace/suites/<target>/<suite>/suite.yaml --case-id TC-XXX-001
python3 -m aitest_kit.cli report --suite-file test_workspace/suites/<target>/<suite>/suite.yaml
python3 -m aitest_kit.cli codegen --target <target> --module <module> --check
python3 -m aitest_kit.cli run --target <target> --module <module>
python3 -m aitest_kit.cli report --target <target> --module <module>
python3 -m aitest_kit.cli run --target <target>
python3 -m aitest_kit.cli report --target <target>
python3 -m aitest_kit.cli run --all
python3 -m aitest_kit.cli report --all
python3 -m aitest_kit.cli upgrade --check --workspace /path/to/aitest_workspace
python3 -m aitest_kit.cli upgrade --apply --workspace /path/to/aitest_workspace
python3 -m compileall aitest_kit/codegen
python3 -m pytest test_workspace/generated --collect-only -q
```

完整 codegen/run/report 选项见 `aitest <subcommand> --help`。

## 技术栈

- Python 3.9+
- FastAPI
- gRPC
- Redis
- `httpx`
- `grpcio`
- `pytest`

## 测试飞轮工作流

九个核心 skill 构成一条闭环流水线。仓库本地可能还有 `project-learning` 等辅助 skill，但它们不属于默认测试飞轮。

```text
设计阶段：
docs/
  -> doc-review
  -> doc-gen（按需）
  -> knowledge-build
  -> test-design
  -> 人工评审
     -> 通过：进入 test-scaffold / test-codegen
     -> 有问题：test-fix 修正用例并沉淀经验

脚手架阶段：
test-scaffold
  -> 构建或增量修正 target/module fixture、helper、module profile
  -> 基于具体 suite 用例补 suite profile
  -> 验证：validate-profile / dump-ir / codegen --check / collect

代码生成阶段：
test-codegen
  -> validate-profile / dump-ir / check
  -> 生成 generated pytest
  -> collect 验证

执行报告阶段：
aitest run
  -> freshness check
  -> pytest 执行
  -> result.json + report.md
  -> 失败分流
     -> 用例问题：test-fix -> 重新 codegen
     -> fixture/codegen 问题：test-scaffold 或更新 fixture/profile/emitter -> 重新 codegen
     -> 待测系统问题：记录到 test_workspace/results/

维护与沉淀：
test-maintain
  -> 诊断当前 workspace 状态
  -> 判断是文档、知识库、用例、scaffold、codegen、run/report 哪一层断裂
  -> 路由到对应 skill 或 CLI
测试稳定通过且出现重复模式
  -> emitter-build 分析是否值得沉淀
  -> 人工 review 后再更新 profile / assertion_rules / fixture helper / emitter 规则
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tlzmw001/aitest-kit](https://github.com/tlzmw001/aitest-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
