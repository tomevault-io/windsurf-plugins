---
trigger: always_on
description: > 灵感来源:Claude Code / Claw Code,做成飞书原生体验。
---

# Berry

> 接入飞书的编码 Agent 后端。
> 灵感来源:Claude Code / Claw Code,做成飞书原生体验。

---

## ⭐ Superpowers Skills 使用范围(对所有会话生效)

只允许使用以下两个 superpowers skill,其他一律不要 invoke:

- `superpowers:brainstorming` —— 设计前的方案探索与确认
- `superpowers:test-driven-development` —— 实现阶段写单测时使用

不要使用其他 superpowers skill(如 writing-plans / subagent-driven-development / systematic-debugging / verification-before-completion / using-git-worktrees / executing-plans / receiving-code-review / requesting-code-review 等),它们的精神可以体现在普通工作里,不需要走 skill 通道。

**写 plan**:直接在对话里写,或写成 markdown 文档,不 invoke `writing-plans`。
**实施**:在主会话里自己写代码、自己跑测试,需要时手动 spawn 一两个 subagent(用 `Agent` 工具),不 invoke `subagent-driven-development`。
**例外**:用户在新对话里显式要求用某个 skill,以用户当时的指令为准。

---

## ⭐ Git 安全:绝不提交 .env(对所有会话生效)

**千万不要 `git add` 任何带有真实凭证 / API key 的 `.env` 文件**,无论:
- 文件名变种(`.env` `.env.local` `.env.production` `.env.<provider>` 等)
- 路径(根目录、子目录、`config/.env`、`scripts/.env`)
- 看起来「文件很短没几行」

只有 `.env.example`(纯模板、占位值)允许进 git。

**操作规则**:
- 永远不用 `git add -A` / `git add .` 这种通配 add,改用具名 add(`git add path/to/specific.py`)
- 提交前检查 `git status` 输出,看到任何 `.env` 出现在 staged 列表立刻 `git restore --staged <file>`
- 如果一不小心已 commit,立刻告诉用户;不要自己 push,也不要试图用 `git filter-branch` / `git reset --hard` 等破坏性操作"补救"

`.gitignore` 已经把 `.env` 拉黑(`!.env.example` 例外),这条规则是双保险:`.gitignore` 是技术防线,这条是行为防线。

---

## ⭐ Git 操作:只改代码,不做提交(对所有会话生效)

在 berry 项目里,**只改代码,不做任何写入 git 的操作**。提交、推送、合并由用户本人执行。

**禁止的命令**:
- ❌ `git add` / `git commit` / `git push` / `git rebase` / `git reset` / `git tag` / `git merge` / `git checkout -b` 等任何修改仓库或工作树状态的 git 命令
- ❌ 不要主动建议「我帮你提交吧」「我帮你推一下」

**允许的命令**(只读,用来理解当前状态):
- ✅ `git status` / `git diff` / `git log` / `git show` / `git branch -v` 等只读查看类命令

**默认动作**:改完代码就停下,告诉用户「改完了,改了哪些文件、做了什么」,然后让用户自己决定怎么 commit、怎么分组、commit message 怎么写。即便是「顺手」的小改动也不行 —— 用户要保留对仓库历史的完全控制权。

例外:用户在当前对话里**明确**说「你帮我 commit 一下」,才可以执行 commit;即使如此,也要先 `git status` + `git diff` 给用户看,确认无误再提交,且不允许 push。

---

## ⭐ 部署前回归测试(对所有会话生效)

任何代码改动在 push 上线前**必须**先通过回归测试,生产可用性比开发速度优先。

### 回归测试范围(三层都要过)

1. **后端 unit**:`uv run pytest tests/unit -q` —— 必须全过(包括 baseline 已经红的)
2. **后端 integration**:`uv run pytest tests/integration -q` —— 需要本地 PG (`localhost:5432`,user `bbb`,password `berry`),conftest 自动 create/drop 临时库
3. **前端 TS 构建**:`cd web && npm run build` —— `tsc -b && vite build`,把类型错和构建错都拦下

### 强制点

- **GitHub Actions** 是**主拦截层**:每次 push 到 main / 任何 PR 都跑上面三层。CI 红 → main 视为不能上线
- **`scripts/deploy.sh`** 是**温柔提醒层**:在 build 之前跑同样的三层。任一失败 → 默认 abort,提示「跑测试失败,先修」
- **GitHub Actions 设为 required check**:从 GitHub branch protection 启用,push 不绕过(待用户在 GitHub 后台开)

### 热修逃生门

- **`./scripts/deploy.sh --skip-tests`** —— **只在生产已经挂了、必须立刻上**的时候用
- 用了 `--skip-tests` 部署后,**当天**必须把测试补上 / 修绿,下次 push 前不允许再用
- CI 没有逃生门(GitHub Actions 是 required check)。CI 不过 → push 就废,本地 deploy 也别想绕

### 逃生门用法的边界

| 情况 | 用 --skip-tests | 不用 |
|---|---|---|
| 生产 500、用户报障、要立刻补丁 | ✅ | |
| 我自己改了几行,本地懒得跑测试 | | ❌ — 跑测试 |
| 测试基线本来就有红的,跟本次无关 | | ❌ — 先把红的修绿,再 push 本次改动(否则永远修不完) |
| 紧急回滚 | ✅(用 `deploy.sh --rollback`,不走测试) | |

### 落实清单

- `scripts/deploy.sh` 新增 step 0:跑三层测试,任一失败 abort,`--skip-tests` 绕过
- `.github/workflows/test.yml`:push/PR 触发,起 PG 服务跑三层
- 默认要求 baseline 全绿 —— 当前 6 个 feishu approval card 单测红,**第一次部署前必须先修这 6 个或显式 --skip-tests**

---

## ⭐ 实现参考:claw-code(对所有会话生效)

berry 的目标是基本对齐 claw-code 的 agent 设计。**任何 agent 层的实现(工具、prompt、runtime、subagent、cache、history compaction、`<system-reminder>` 注入等)都应优先参考 `reference/claw-code_1/rust/crates/`** —— 不再单独发明 berry 自己的概念体系。

**默认对齐项**(只是常见入口,不是穷尽列表):

- **工具命名 / 形态**:用 claw-code 通用工具(`read_file` / `write_file` / `edit_file` / `bash` / `grep_search` 等),**不为业务场景单独开 `read_note` / `write_md` 这种命名**。业务边界靠 prompt 里的 `# Berry instructions` 表达,不靠工具名。
- **工具描述风格**:一句话 + 必要的「什么时候用」引导,不写多段教学。复杂行为约束放 system prompt 的 `# Doing tasks` / `# Learning together`,不放 tool description。
- **路径安全**:claw-code `path_scope.py` 那一套(`Path.resolve()` + `is_relative_to`),工具调用边界做一次,后续信任。
- **system prompt builder**:已对齐(`berry/skills/learning/prompts/system.md`)。
- **subagent / `<system-reminder>` / cache_break 跟踪 / auto_compaction**:见 `reference/claw-code_1/rust/crates/{tools,api,runtime}/src/`。

**例外**(不对齐 claw-code 的部分):

- **业务编排**(GoalTutor 等 assistants/<name>/)是 berry 自己的,因为 claw-code 是单一编程 agent,berry 是多场景平台
- **DB schema** 是 berry 自己的(claw-code 没数据库)
- **Channel 适配**(飞书等)是 berry 自己的

**红线**:发现自己写「先单独搞一个 berry 风格的 X」时,先去 reference/claw-code_1 看一下他们怎么做的,**沿用是默认值,自己造是例外**(造的话要写 ADR 解释为什么不沿用)。

---

## ⭐ 实现参考:openclaw(飞书相关功能,对所有会话生效)

**飞书 / 卡片 / 多渠道相关功能优先参考 openclaw**,不要先去翻 claw-code(claw-code 是单进程编程 agent,没 channel 层)。

具体范围:`berry/channels/feishu/` 下所有逻辑,以及任何「lark-oapi 调用 / 卡片 schema / card_action 回调 / 审批卡片 / streaming card / reply dispatcher / dedupe / mention gating」 之类话题。

**对齐路径**:`reference/openclaw_1/extensions/feishu/src/`。常见入口:

- `card-ux-approval.ts` —— 审批卡片 schema(orange header / Confirm + Cancel button / interaction envelope)
- `card-action.ts` —— `card.action.trigger` 事件 dispatcher
- `card-interaction.ts` / `card-ux-shared.ts` —— button value 用 envelope 编码,**不要把原始 args 塞 button.value**
- `approval-auth.ts` —— 操作人合法性校验(允许列表 normalize + sender 校验)
- `streaming-card.ts` / `reply-dispatcher.ts` —— streaming card / 回复编排
- `bot.ts` / `bot-content.ts` / `chat.ts` —— 入站消息处理
- `accounts.ts` / `app-registration.ts` —— 多账号 / app 配置


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DdGnoybab/berry](https://github.com/DdGnoybab/berry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
