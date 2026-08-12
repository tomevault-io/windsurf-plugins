---
trigger: always_on
description: >
---


# f2s-task（变更追踪规则）

## 生效条件

各技能按自身子项判断：

- `f2s-kb-feat`：读 `changeTracking.feat`
- `f2s-kb-fix`：读 `changeTracking.fix`
- `f2s-implement-tech-design`：读 `changeTracking.implement`

若对应子项为 `false` 或字段不存在，**该技能内的变更追踪步骤不执行**，直接跳过。

> `f2s-req-plan` 命令不受此条件约束，始终执行（见 `skills/f2s-req-plan/SKILL.md`）。

## 多人协作与任务根 `TASK_ROOT`（必须先解析）

进入本规则任何「读/写 `.task`」步骤前，**必须** `Read("flow2spec.config.json")`，并解析 **`TASK_ROOT`**（本会话内固定，禁止中途改 id）：

| 条件 | `TASK_ROOT` | `developerId` 来源 |
| --- | --- | --- |
| `collaboration.enabled === false` | `.task` | legacy（强制单人根） |
| `collaboration.developerId` 非空（trim 后） | `.task/<sanitize(id)>` | **config** |
| 否则能读到 `git config user.email` | `.task/<sanitize(email@前)>` | **git-email** |
| 否则能读到 `git config user.name` | `.task/<sanitize(name)>` | **git-name** |
| 仍无 | `.task` | **legacy**（单人旧布局；须在回复中提示：建议配置 `collaboration.developerId`） |

**sanitize**：小写；若含 `@` 只取本地部分；非 `[a-z0-9]` 收成 `-`；去首尾 `-`；长度 1–64，否则视为无 id。

**路径一律用 `TASK_ROOT` 前缀**（下面凡写 `TASK_ROOT/...` 均指解析结果）：

- 索引：`TASK_ROOT/todo.json`
- 进行中：`TASK_ROOT/active/<task-name>/`
- 已完成：`TASK_ROOT/completed/<YYYYMMDD>-<task-name>/`

**防串戏（硬约束）**：

1. **只**读写当前会话的 `TASK_ROOT`；**禁止**为续作/匹配去遍历 `.task/*/todo.json` 或其他 developer 目录。
2. keywords 匹配范围 **仅** 当前 `TASK_ROOT/todo.json` 内条目。
3. 创建任务时 `todo.json` 的 `folder` 必须写成当前 `TASK_ROOT/active/<task-name>/`（posix 相对路径）。
4. 可选在任务开始时向用户回显一行：`[task] developerId=<id|legacy> TASK_ROOT=<path>`。
5. **`.Knowledge/` 仍为全员共享**；本规则不按人拆分知识库。

> 实现参考（CLI/工具）：包内 `lib/developerId.js` 的 `resolveDeveloperContext` / `taskRootFor`（与上表同口径）。

## f2s-req-plan 调用时的绑定

执行 **`f2s-req-plan`**（或续作命中 `linkedSkill: "f2s-req-plan"`）时：

- **不受** `changeTracking.feat` / `fix` / `implement` 限制，但 **必须** 按本规则「任务开始 / 执行中 / 中断与会话结束 / 任务完成 / 新会话续作」维护 **`TASK_ROOT`** 下任务树；
- 技能 **步骤 0** 须 `Read` 本规则全文（**Cursor/Claude**：`rules/f2s-task.*`；**Codex**：`.codex/topics/f2s-task.md`）；
- 落盘、打钩、归档、`user-todos.md` / `acceptance.md` 格式 **以本规则为准**；技能正文不得省略 `todo.json` / `user-todos.md` / `acceptance.md`，不得改写归档目录命名（`<YYYYMMDD>-<task-name>`）。

## 目录结构

```
TASK_ROOT/                             ← `.task` 或 `.task/<developerId>`
├── todo.json                          ← 活跃任务索引，仅主 agent 写
├── active/
│   └── <task-name>/
│       ├── task.md                    ← checklist（执行步骤）
│       ├── context.md                 ← 涉及文件路径、相关资料链接
│       ├── user-todos.md              ← 须用户执行的代办（改库、配环境等），见下文
│       └── acceptance.md              ← 验收清单：task.md 全部 [x] 后、归档前生成，见下文
└── completed/
    └── <YYYYMMDD>-<task-name>/
        ├── task.md
        ├── context.md
        ├── user-todos.md              ← 随任务一并归档，便于验收后逐项消项
        └── acceptance.md              ← 随任务一并归档，便于用户最终核对
```

**归档目录命名**：`completed/` 下文件夹名为 **`<YYYYMMDD>-<task-name>`**（**本地日历日期 8 位在前**，`<task-name>` 与 `active/` 下一致、为 snake_case；便于按时间排序）。**新归档一律使用本格式**；仓库中已有的旧式 `<task-name>-<YYYYMMDD>` 目录可保留，择机人工重命名即可。

**从单人布局迁移**：若磁盘仍有根级 `.task/active/` 与 `.task/todo.json`，而当前已解析出非 legacy 的 `TASK_ROOT=.task/<id>`，可在用户确认后将旧 `active/*` 与条目迁入新根（一次性）；未确认前 **不要** 自动挪动他人可能共用的根目录。

## todo.json 结构

```json
[
  {
    "name": "任务名称",
    "folder": "TASK_ROOT/active/<task-name>/",
    "keywords": ["关键词1", "关键词2"],
    "linkedSkill": "f2s-kb-fix",
    "createdAt": "YYYY-MM-DD",
    "assignee": "<developerId 或 legacy>"
  }
]
```

`folder` 落盘时须写成真实相对路径（例如 `.task/alice/active/fix_foo/` 或 legacy 的 `.task/active/fix_foo/`）。`assignee` 建议写入当前 `developerId`（legacy 可写 `"legacy"` 或省略）。

**写权约束**：`todo.json` 仅由主 agent 写，禁止子 agent 修改。

## 任务开始（代码变更前）

0. 按上文解析并固定 **`TASK_ROOT`**（及 developerId / legacy）。
1. 检查 `TASK_ROOT/todo.json` 是否存在活跃任务。
2. 将用户输入与**该文件内**各条目 `keywords` 匹配（**禁止**读取其他 `TASK_ROOT`）：
   - 命中一个 → 加载对应 `task.md`、`context.md`，**若存在** `user-todos.md` 则一并加载，展示剩余清单与未消用户代办
   - 命中多个 → 列出候选，让用户选择
   - 无命中 → 确认任务名称后创建新任务
3. 创建新任务（无命中时）：
   a. 确认任务名称（snake_case，简短描述变更内容）
   b. 在 `TASK_ROOT/active/<task-name>/` 创建文件夹
   c. 将本次工作步骤写入 `task.md`
   d. 将涉及文件路径和相关资料链接写入 `context.md`
   e. **创建 `user-todos.md`**（固定文件名，与 `task.md` 同目录）：见下文「`user-todos.md` 格式与写盘义务」；尚无代办时可写入占位说明
   f. 在 `TASK_ROOT/todo.json` 新增条目（仅主 agent 写；`folder` 指向本任务目录）

## 执行中

- 每完成一个步骤，**立即**用 `Edit` / `Write` 将 `task.md` 中对应 checkbox 由 `[ ]` 改为 `[x]`（与代码改动同等对待，**禁止**仅靠会话内口头宣称「已完成」代替磁盘更新）
- 禁止批量勾选或跳步
- **用户代办须落盘**：凡须任务责任人（用户）在本机、数据库、配置平台或流程上完成的项（例如执行 DDL/DML、填密钥、点审批、发版、补数据），**同一会话内**追加写入 `user-todos.md`（`Edit` 追加小节或列表项），**禁止**仅在对话里交代而不写入该文件；可与对话摘要并存，以磁盘文件为交接真值

## 中断与会话结束（硬约束）

- **长记忆以 `task.md` 的 checkbox 为真值**：下一会话通过「首个仍为 `[ ]` 的步骤」定位进度；未写盘则续作失真。
- 本会话内每真实完成 `task.md` 所列一步：**当步**打钩，不得积压到归档前一次性勾选。
- 若用户结束对话、工具流中断、或预计无法继续：在结束前至少打钩**已真实完成**的步骤，并在「## 备注」写明阻塞原因或「下一会话从步骤 N 继续」；**禁止**在未更新 `task.md` 的情况下直接结束（否则等同丢失进度信号）。
- 中断前若本会话已识别出**用户代办**：**必须**写入或追加到 `user-todos.md`，避免下一会话丢失「交给用户的事」。
- 若本会话为子任务创建过 **`git worktree`** 或等价隔离目录：结束前按 **`f2s-flow2spec-unified-entry`**「Git worktree 与子任务工作目录卫生」完成移除或写明残留路径与删除命令（必要时写入 `user-todos.md`）。

## 任务完成

**归档门禁（须先于移动目录自检）**：

- 将目录移入 `completed/` **当且仅当** `task.md` 的「## 步骤」下，与本次交付相关的条目**全部为 `[x]`**（或用户明确取消的项已在「## 备注」说明，且对应列表项已改为 `[x]` / 已删除该项并注明取消）。
- `task.md` 全部 `[x]` 后、移动目录前，**必须**已生成或更新 `acceptance.md`（见下文「acceptance.md 格式与写盘义务」）；缺失 `acceptance.md` 或仍为创建任务时的占位说明 → 视为门禁未过，禁止归档。
- 若仍存在 `[ ]`：**禁止**移动 `active` → `completed/`、**禁止**从 `todo.json` 删除该条目；应先回到「执行中」补完或改清单后再归档。

完成上述门禁后：

1. 将 `TASK_ROOT/active/<task-name>/` 整体移至 `TASK_ROOT/completed/<YYYYMMDD>-<task-name>/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [double-coding-lab/Flow2Spec](https://github.com/double-coding-lab/Flow2Spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
