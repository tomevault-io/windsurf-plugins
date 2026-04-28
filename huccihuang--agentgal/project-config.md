---
trigger: always_on
description: 多 Agent 角色扮演 / 叙事游戏项目。当前实现以 **FastAPI + pydantic-ai + Pydantic 结构化输出 + 文件记忆 + sqlite-vec** 为核心，使用 `uv` 作为项目管理器。
---

多 Agent 角色扮演 / 叙事游戏项目。当前实现以 **FastAPI + pydantic-ai + Pydantic 结构化输出 + 文件记忆 + sqlite-vec** 为核心，使用 `uv` 作为项目管理器。

## 核心设计

- **独立记忆**：角色维护自己的 `memory.md / status.md / user.md`，`narrator` 维护 `status.md` 与 raw 历史
- **信息差**：消息按 `visible_to` 控制可见范围，未参与场景的角色不会看到该轮内容
- **旁白先行**：`narrator` 先做路由与场景推进，再顺序调用目标角色
- **结构化输出**：所有结构化 Agent 使用 `PromptedOutput`，不输出 XML；系统直接读取 typed 字段写回文件
- **双层记忆**：Markdown 文件可读可编辑，向量库负责检索

## 技术栈

- Python 3.11+
- FastAPI + SSE（服务端推送）
- pydantic-ai（`pydantic-ai`）—— `Agent` / `PromptedOutput` / `OpenAIChatModel` / provider-specific `Provider`
- sqlite-vec + aiosqlite
- asyncio

## 当前项目结构

```text
agentgal-memos/
├── server.py                   # FastAPI 入口（UI 适配层）
├── config.toml                 # 非密钥运行参数
├── data/
│   ├── characters/             # 运行时角色数据
│   ├── templates/              # 故事模板（school / modern）
│   └── vectors.sqlite          # 向量库
├── engine/                     # 对话运行时编排
│   ├── character.py            # Character / Narrator 运行封装与 typed 输出写回
│   ├── character_factory.py    # 新角色孵化
│   ├── conversation_flow.py    # 单轮对话编排与 UI 适配函数
│   └── prompt_builder.py       # 对话 prompt / 历史窗口 / schedule 快照构造
├── agents/                     # SDK 基础设施（技术支撑层）
│   ├── factory.py              # Agent 创建、注册表与 SDK model 配置
│   ├── runner.py               # SDK Runner 调用、Logfire trace 与 typed parse
│   └── schema.py               # Pydantic 结构化输出类型
├── world/                      # 世界模型（时间 / 位置 / 离场）
│   ├── schedule.py             # 角色 schedule 查询、游戏时间解析、时段匹配
│   ├── sync.py                 # 回合后位置 / last_seen 同步
│   └── offstage.py             # 离场追补（offstage_synthesizer）
├── consolidation/              # 后台记忆整理（独立流程）
│   ├── flow.py                 # 整理编排：memory 归并 / growth / user 精炼
│   └── inputs.py               # 整理 prompt 组装（memory_owner / raw_dialogue）
├── llm/
│   ├── providers.py            # Provider 配置与 URL 解析（返回 provider/api_url/api_key/model/temperature）
│   ├── embedding.py            # Embeddings 客户端（embed_async / embed_sync）
│   └── rerank.py               # Rerank API 客户端
├── log_config/                 # Logfire 与业务 logger 配置
├── memory/                     # 记忆规则与流程
│   ├── indexer.py              # 向量索引重建入口（从 memory.md 解析后写入 storage）
│   ├── parser.py               # memory.md 格式解析、事件切分、日期工具、记忆块操作
│   └── retrieval.py            # 完整检索 pipeline（融合、rerank、recency、召回状态更新）
├── shared/                     # 纯配置与无副作用工具函数
│   ├── config.py               # 路径、运行参数、character_path、get_agent_names
│   └── text_utils.py           # 文本清理、get_display_name
├── storage/                    # 持久化基础设施（文件 / JSONL / sqlite-vec / 存档）
│   ├── agent_files.py          # 角色目录文件操作（read/write soul/memory/status/user/growth/sidecar）
│   ├── history.py              # narrator raw JSONL 对话历史读取
│   ├── message_router.py       # 对话写入 / 可见性过滤
│   ├── save_manager.py         # 存档 / 读档 / 重置 / 开场加载
│   └── vector_store.py         # sqlite-vec 向量存储（write/delete + 原始候选检索）
├── prompts/                    # narrator / character / consolidation prompts
├── scripts/                    # 维护脚本
├── static/                     # Alpine.js + HTML/JS 前端
├── tests/                      # pytest 测试
├── README.md
├── AGENTS.md
├── CLAUDE.md
└── .env
```

### 分层依赖方向

```
shared/          ← 无内部依赖
storage/         ← shared/
llm/             ← shared/
agents/          ← shared/                            # SDK 基础层
memory/          ← shared/ + storage/ + llm/
world/           ← shared/ + storage/ + agents/
consolidation/   ← shared/ + storage/ + agents/ + memory/ + llm/
engine/          ← shared/ + storage/ + agents/ + memory/ + world/ + consolidation/
server.py        ← 全部
```

## 运行时文件职责

### 角色文件

- `soul.md`：手写角色定义，只读；分 `<identity>` / `<goal>` / `<dynamic>` / `<behavior>` / `<voice>` 五段，其中 `<goal>` 写角色在故事期内要拿到的具体长期目标（外部可验证里程碑 + 可选的关系愿景），整个故事期大体不变
- `memory.md`：角色长期记忆，记录事件与情绪变化（仅角色有）
- `status.md`：当前状态；角色包含「打算」，旁白包含「待触发事件」和「角色位置」（所有主要角色的当前位置快照，由 `state_updater` 每轮维护，角色端不再自维护「当前位置」）
- `user.md`：角色对玩家的认知（仅角色有，`narrator` 无）
- `tmp_user.md`：`user.md` 的工作草稿；首次写入时复制正式档案，整理后删除
- `growth.md`：人格沉淀，由整理器维护并在角色 prompt 中注入（仅角色有）
- `relations.md`：角色对其他角色（不含 `player`）的当下视角；`## {target}` 一节一段；每轮 `output.relations[target]` 整段覆盖（仅角色有）。对玩家的长期视角走 `user.md` 与 `status.md` 的「和玩家的关系」

### 历史文件

- 当前对话历史**只写入** `data/characters/narrator/raw/YYYY-MM-DD.jsonl`
- 每条消息带 `visible_to`
- 角色读取上下文时，通过可见性过滤出自己能看到的消息

### 其他运行时文件

- `data/characters/last_choices.json`：最新一组玩家选项，续档时恢复展示，重置时清除
- `data/characters/narrator/tasks.md`：可选剧情种子文件；当前主流程主要由 `state_updater` 从角色 `打算` 同步 `待触发事件`
- `data/characters/*/.history_window_state.json`：各 Agent 的对话历史高低水位窗口 sidecar
- `data/characters/*/.consolidation_state.json`：角色记忆整理进度 sidecar
- `data/characters/*/.memory_recall_state.json`：角色长期记忆 recall 快照（仅存档时从 DB 生成，运行期不维护）
- `data/characters/*/.last_seen.json`：角色上次出场的游戏内时间，由 `world_sync` 在 targets 出场时写入

## 消息路由

由 `narrator` 负责决定谁参与当前回合。

```text
用户输入 → narrator → targets: [“角色名”, ...]（NarratorOutput.targets）
```

### narrator 的职责

- 分析玩家输入，输出非空 `targets` 数组
- 判断玩家是否仍有和角色互动的意愿：有则延续当前场景；分别、跳过时间或不再互动时，导向待触发事件或制造同等作用的即时张力
- 每轮都必须让至少一个主要角色当轮可感知玩家并回应
- 描述时间、地点、在场信息、环境、纯 NPC 行为和当前钩子
- 不新增未来事件；未来事件由 `state_updater` 从角色 `打算` 维护

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huccihuang/AgentGal](https://github.com/huccihuang/AgentGal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
