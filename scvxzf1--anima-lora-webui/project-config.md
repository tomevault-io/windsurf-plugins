---
trigger: always_on
description: 本文件是给 AI Agent 长期维护本仓库用的根级工作协议。它覆盖整个
---

# AGENTS.md

本文件是给 AI Agent 长期维护本仓库用的根级工作协议。它覆盖整个
`anima_lora` 仓库；子目录如果另有 `AGENTS.md` 或 `CLAUDE.md`，以离目标文件更近
的说明为补充约束。根目录曾经通过 `@CLAUDE.md` 引用维护说明，但当前根级
`CLAUDE.md` 可能不存在，因此不要依赖外部展开，优先以本文件和实时源码为准。

## 总体原则

- 默认用简体中文沟通，代码、命令、配置键、错误信息保持项目原有语言风格。
- 先定位子系统，再读最小必要上下文；优先用 `rg` / `rg --files` 查找。
- 保持改动小而准确。不要顺手重构、重排大文件、格式化无关文件或改写历史文档。
- 保护用户运行数据。不要把 `.venv/`、`models/`、`output/`、`post_image_dataset/`、
  `logs/`、`configs/imported/`、`configs/web-training-history/`、
  `configs/web-training-queue/` 当作普通源码清理或覆盖。
- 不要擅自删除模型、缓存、训练结果、历史任务、队列文件或用户导入配置。
- 不要擅自终止训练、清空队列、批量移动输出、下载大模型或启动长训练。确需执行时，
  先说明影响并取得用户明确同意。
- 遇到用户未提交改动，默认那是用户或其他 Agent 的工作。只在任务必须触碰同一文件时
  谨慎合并，不要 revert。
- 代码事实优先于文档。若本文件、旧说明和源码不一致，先读源码和测试，再更新文档。

## 反上帝代码守则

本节用于防止后续维护继续把复杂逻辑堆回少数大文件。

- 热点文件默认只做 facade、编排、兼容 shim 或小范围修复，不新增大块业务逻辑：
  - `train.py`
  - `inference.py`
  - `library/inference/generation.py`
  - `library/datasets/base.py`
  - `networks/lora_anima/network.py`
  - `networks/lora_anima/config.py`
  - `web/services/training_service.py`
  - `web/services/config/_legacy.py`
  - `web/static/js/features/anima-app/chunks/*`
- 修改热点文件超过约 50 行时，优先拆到现有子模块或新模块；若确实不能拆，最终回复要说明：
  - 为什么必须改热点文件。
  - 为什么不能放到新模块。
  - 后续如何继续瘦身。
  - 跑了哪些定向测试。
- 单个新增 Python 函数建议不超过 100 行；超过时优先拆成 helper、pipeline step 或策略对象。
- 单个新增 Python 类建议不超过 400 行；超过时优先按状态、IO、策略、验证、保存/加载拆分。
- 单个新增 JS 函数建议不超过 80 行；新增 UI 逻辑必须按 feature、store、api、renderer 拆分。
- 单个新增测试文件建议不超过 1200 行；超过时按领域拆成多个测试文件，不要继续加大现有超大测试。
- 已超过 1000 行的源码或测试文件，除搬迁、兼容 shim、删除旧逻辑外，默认不继续承载新业务。
- 重构优先采用“搬家型重构”：先保持行为不变地抽模块，再补测试和清理旧 facade；不要一轮同时改架构和改行为。
- 新增配置、CLI、adapter、WebUI 表单或队列/历史行为时，必须同步考虑文档入口、测试入口和旧兼容面，避免逻辑散落。

## 环境和命令入口

- 项目运行环境是 Python 3.13，依赖管理优先使用 `uv`。
- `tasks.py` 是命令入口真相；`Makefile` 只是薄转发。查命令实现时读
  `tasks.py`、`scripts/tasks/` 和 `scripts/experimental_tasks/`。
- 维护和验证命令优先使用 `.venv/bin/python`；只有确认无需项目虚拟环境，或 `.venv/`
  不存在时，才回退到系统 `python`。
- 跨平台用户文档可写成 `python tasks.py <command>`，本仓维护执行优先写成
  `.venv/bin/python tasks.py <command>`。
  用户文档里常见 `make <target>`，但维护时不要只看 `Makefile`。
- `python tasks.py <command> KEY=value` 支持 Make 风格尾随环境变量，例如：
  `python tasks.py print-config METHOD=lora PRESET=default`。
- 实验命令通常以 `exp-*` 开头，可能变动或删除。改实验命令时同步检查
  `scripts/experimental_tasks/`、配置、文档和测试。
- 常用启动：
  - WebUI：`.venv/bin/python tasks.py web --host 127.0.0.1 --port 20102`
  - GUI：`.venv/bin/python tasks.py gui`
  - 单元测试：`timeout 60 .venv/bin/python -m pytest tests/<test_file>.py`
  - 全量单测入口：`timeout 60 .venv/bin/python tasks.py test-unit`
  - 合并配置查看：`.venv/bin/python tasks.py print-config METHOD=<name> PRESET=<name>`
- 从旧 `CLAUDE.md` 继承且仍然有效的最小初始化：
  - `uv sync`
  - `hf auth login`
  - `python tasks.py download-models`
  - `python tasks.py preprocess`
  - 默认训练图片放在 `image_dataset/`，同名 `.txt` caption sidecar 也放这里。
- 命令面速记：
  - 稳定训练/服务：`lora`、`lora-gui`、`web`、`gui`、`daemon*`、`preprocess*`、
    `caption-index`、`preprocess-tagger`、`tagger`、`test-tagger`
  - 稳定推理/工具：`test`、`test-mod`、`test-hydra`、`test-merge`、`test-dcw`、
    `test-dcw-v4`、`merge`、`distill-mod`、`export-logs`、`vendor-sync`、
    `print-config`、`update`
  - 实验训练：`exp-turbo`、`exp-turbo-prep`、`exp-spd`、`exp-soft-tokens`、
    `exp-chimera`、`exp-ip-adapter`、`exp-easycontrol`、`exp-byg`
  - 实验推理/探针：`exp-test-soft`、`exp-test-turbo`、`exp-test-spd`、
    `exp-test-ip`、`exp-test-easycontrol`、`exp-test-byg`、
    `exp-test-directedit`、`exp-test-directedit-dry`、`exp-invert-directedit`
  - `python tasks.py --help` 是命令面的实时快照；老文档里出现但不在 `tasks.py`
    当前表里的命令，按历史/兼容入口处理，不要直接假定仍可用。

## Repowise 代码库地图

- 本机已建立 repowise 索引，Codex MCP 中当前仓库名为 `repowise_anima_lora`；独立
  WebUI 仓库名为 `repowise_anima_webui`。
- 跨模块排查、架构理解、风险分析、dead-code、调用链或符号定位时，优先用 repowise
  获取概览和候选上下文，再读取实时源码确认。
- repowise 索引不是实时真相，不替代 `git diff`、直接读文件和测试验证；新增/删除/重命名
  文件，或修改 import/export、路由、命令、服务注册、公共接口、跨模块调用链后，建议运行
  `uvx repowise update` 刷新地图。
- `.repowise/` 和 `.mcp.json` 是本机索引/本机路径配置，不要提交。

## Git 推送和回滚

- 当前固定协作口径：本地 `main` 只和 `webui/main` 对齐。用户说“拉取线上更新”、
  “同步线上 main”、“推送更新到线上”时，默认目标都是 `webui/main`
  (`git@github.com:scvxzf1/anima_lora_webui.git`)。
- `private/main` 不再作为默认同步或发布目标，只保留为个人主仓/历史镜像。除非用户明确点名
  `private`，不要向它 pull、push、reset 或拿它当“线上 main”。
- `origin/main` 是上游参考仓，默认只读。需要从上游合入时，先单独做差异审计和合并计划，
  不要把它和发布同步混在一起。
- 推送前至少检查：`git status --short --branch`、`git fetch webui --prune`、
  `git log webui/main..HEAD`，再跑和改动直接相关的测试。未跟踪文件默认不随推送发布，
  除非用户明确要求或本次任务已确认需要纳入版本控制。
- 用户说“推送更新到线上”时，默认执行 `git push webui main:main`。推送后要汇报目标远程/分支、
  最新提交 hash，以及本地是否还有未提交或未跟踪改动残留。
- 用户说“回滚”时，先分清是哪一种：
  - 本地工作区回退：丢弃未提交改动。只有用户明确要求时才做，执行前说明会丢失哪些内容。
  - 本地提交回退：撤销本地一个或多个提交。共享分支默认优先 `git revert`，不要默认改写历史。
  - 线上回退：撤回远程分支上的已发布提交。必须先确认目标远程、目标分支和目标提交。
- 需要以线上仓库为准同步本地时，先 `fetch` 目标远程并比较 `HEAD` 和远端分支，再决定
  是否 reset。不要在没核对差异前直接做破坏性操作。
- `git reset --hard`、`git checkout -- <path>`、`git push --force`、`git push --force-with-lease`
  都视为高风险操作。除非用户已经明确要求，或已经明确给出目标提交/分支并接受影响，
  否则不要执行。
- 如果确实要改写线上历史，优先使用 `--force-with-lease` 而不是裸 `--force`，并在执行前
  说明影响范围：会覆盖哪个远程分支、抹掉哪些提交、是否影响其他协作者。
- 可以使用环境变量中的凭据或本机已配置的 SSH key 推送，但不要把 PAT、cookie、私钥、
  或带密钥的远程 URL 写进仓库文件、文档、日志样例或长期说明。

## 项目地图

- `tasks.py`：所有稳定命令注册表。
- `train.py`：`AnimaTrainer` 主训练入口。
- `inference.py`：独立推理入口。
- `anima_lora/`：可安装包门面，给嵌入式调用暴露精选 API。
- `library/`：训练、推理、配置、数据、runtime、模型、captioning、vision 等核心逻辑。
- `library/anima/`：Anima DiT、权重加载、token/text strategy 和模型配置。
- `library/config/`：TOML 读取、合并、normalize、schema 校验。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scvxzf1/anima_lora_webui](https://github.com/scvxzf1/anima_lora_webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
