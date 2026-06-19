---
trigger: always_on
description: Agent 分层记忆管理技能（v3，适配 OpenClaw 原生记忆结构）。当用户或 Agent 需要执行以下任何操作时必须使用本技能：写入或更新长期记忆（MEMORY.md）、更新用户信息（USER.md）、生成或追加每日日志、执行记忆整合与瘦身、诊断记忆健康度、安装自动化 Hook。v3 在 v2 自进化三件套基础上新增三项关键能力：(1) Hook 自动激活——通过 OpenClaw hooks 机制在每次 Prompt 提交后静默评估、工具出错后立即记录，无需用户手动唤起 evolve；(2) 候选池三次晋升——新观察到的经验先写入 CANDIDATES.md 缓冲区，同一模式被观察满 3 次才提请用户审批晋升到 MEMORY.md，防止单次偶发指令被误固化；(3) 行级来源引用——Agent 依据记忆执行操作时必须标注来源文件及行号（格式：📍 MEMORY.md:L23），实现完整可溯源链路。本技能直接操作 OpenClaw 的原生记忆文件，不创建额外独立目录。触发词包括：初始化记忆、写入记忆、更新 memory、更新 USER.md、记录这件事、写日志、记忆整合、memor
---


# Memory Manager Skill（v3）

**设计理念**：记忆系统是 Agent 的"操作系统内核"——必须始终精简，按需扩展，并对用户完全透明。v3 的核心升级在于：从"被动等待用户要求记录"进化为"Hook 驱动的主动感知 + 候选池防误学 + 操作全程可溯源"。

---

## OpenClaw 记忆结构适配

本技能直接操作 OpenClaw 的原生记忆文件，**不创建独立的 `.agent-memory/` 目录**。

```
~/.workbuddy/                          ← 全局身份层（只读为主）
├── SOUL.md                            ← Agent 灵魂/价值观（memory-manager 不修改）
├── IDENTITY.md                        ← Agent 身份/风格/emoji（memory-manager 不修改）
├── USER.md                            ← 用户信息：偏好、背景 ✏️ memory-manager 可写
└── memory/                            ← 全局记忆目录
    └── （如有全局长期规则可放此处）

{project_workbuddy}/memory/            ← 项目工作层（主要操作目标）
├── MEMORY.md                          ← 长期记忆：规则、错误修正、重要约束 ✏️
├── CANDIDATES.md                      ← 候选池：3次晋升缓冲区（v3 新增）✏️
├── EVOLUTION_LOG.md                   ← 进化记录（v3 继承自 v2）✏️
├── skills/
│   ├── INDEX.md                       ← 技能索引 ✏️
│   └── auto-generated/                ← 自动生成的 Skill 草稿 ✏️
└── 2026-04-XX.md                      ← 每日日志（情节记忆）✏️
```

### 文件职责映射

| v2 概念 | v3 对应的 OpenClaw 文件 | 操作权限 |
|---------|------------------------|---------|
| CORE.md · 用户偏好区块 | `~/.workbuddy/USER.md` | 可写 |
| CORE.md · 重要规则区块 | `{project}/memory/MEMORY.md` | 可写 |
| CORE.md · 错误修正区块 | `{project}/memory/MEMORY.md` | 可写 |
| 情节记忆 episodic/ | `{project}/memory/2026-04-XX.md` | 可写 |
| EVOLUTION_LOG.md | `{project}/memory/EVOLUTION_LOG.md` | 可写 |
| CANDIDATES.md（v3 新增）| `{project}/memory/CANDIDATES.md` | 可写 |
| Skills 索引 | `{project}/memory/skills/INDEX.md` | 可写 |
| SOUL.md / IDENTITY.md | `~/.workbuddy/SOUL.md` / `IDENTITY.md` | **只读** |

**重要**：SOUL.md 和 IDENTITY.md 是 Agent 的身份定义文件，memory-manager **永远不修改**它们。

---

## 四层记忆架构（v3 适配版）

### Layer 0：身份层（只读）
**文件**：`~/.workbuddy/SOUL.md`、`~/.workbuddy/IDENTITY.md`
**定位**：Agent 的价值观和身份定义，由 Agent 配置者设定，memory-manager 只读取，不修改。

---

### Layer 1：用户信息层
**文件**：`~/.workbuddy/USER.md`
**定位**：存储用户的偏好、背景信息、沟通风格等。在 OpenClaw 的 workspace 注入机制下，此文件每次 session 自动加载。

**写入内容**：
- 用户语言偏好
- 用户沟通风格和习惯
- 用户明确要求记住的个人背景信息
- 工作领域和专业背景

**写入门槛**：用户明确表达，或候选池观察满 3 次的偏好类信息。

**v3 行号管理**：USER.md 头部维护行号速查注释，每次写入后更新。

---

### Layer 2：长期记忆层
**文件**：`{project_workbuddy}/memory/MEMORY.md`
**定位**：项目级长期规则、错误修正、重要约束。每次 session 由 OpenClaw 注入。

**写入内容**：
- Agent 犯过的错误 + 修正后的规则（不删除，标注"已修正"保留）
- 用户确认的重要规则和行为约束
- 技能索引引用（`→ skills/INDEX.md`）
- 月度日志归档引用（仅文件名）

**写入门槛**：高。判断标准：**"如果忘了这条，下次 Agent 一定会出错或违背用户意图"**，才写入。

**大小限制**：不超过 200 行，超过时触发 `consolidate`。

**v3 行号速查表**：文件头部维护行号索引，每次写入后更新：
```markdown
<!-- 行号速查（每次写入后更新）
L6-L20   重要规则
L22-L40  错误修正记录
L42-L48  技能索引引用
L50-L55  日志归档引用
L57-L62  进化历史摘要
-->
```

---

### Layer 1.5：候选池（v3 新增）
**文件**：`{project_workbuddy}/memory/CANDIDATES.md`
**定位**：介于"对话临时上下文"和"已确认记忆"之间的缓冲层。所有尚未验证足够重要的经验，先在此处积累观察次数。

**晋升规则**：
- 同一模式被观察到 **≥ 3 次** → 触发进化请求卡片，用户审批后正式写入 MEMORY.md 或 USER.md
- 用户**明确说出**"记住"/"以后都这样"/"永远不要" → 跳过候选池，直接生成进化卡片（1 次即可晋升）
- 候选项超过 **30 天未再次观察** → 自动丢弃（记录到 EVOLUTION_LOG.md）

**直接丢弃（不进候选池）**：
- 一次性指令（"现在帮我做 X"）
- 特定文件/上下文的临时指令（"在这个文件里用 XX 格式"）
- 假设性表达（"如果...应该怎么..."）

---

### Layer 3：情节记忆层
**文件**：`{project_workbuddy}/memory/2026-04-XX.md`（每日日志）
**定位**：记录"今天发生了什么"，对应 OpenClaw 的每日日志文件。

**写入时机**：
- 完成一次重要任务后
- 解决了一个复杂问题后
- session 结束时的简要总结

**衰减规则**：
- 超过 30 天的日志 → 执行 `consolidate` 时压缩为月度汇总
- 超过 90 天的月度汇总 → 可归档或删除（需用户确认）

---

## v3 三项新机制

---

### 机制一：Hook 自动激活

**解决的问题**：v2 依赖用户或 Agent 手动执行 `evolve`，容易遗漏。v3 通过 OpenClaw hooks 实现无感后台触发。

#### 两个 Hook

**① `activator.sh`（UserPromptSubmit）**
- 触发时机：每次用户提交 Prompt
- 作用：在 Agent 上下文末尾注入一段轻量提醒，让 Agent 在回复时顺带扫描是否出现学习信号
- Token 开销：约 40–60 tokens
- 关键原则：仅注入提示，不自动写入任何文件——写入必须经用户审批

**② `error-detector.sh`（PostToolUse）**
- 触发时机：每次工具调用（Bash、文件操作等）完成后
- 作用：检测退出码，出错（exit code ≠ 0）时立即将此次错误写入 CANDIDATES.md 作为候选经验
- 意义：工具出错是最有学习价值的时刻，不应等到 session 结束才记录
- 安全注意：此 hook 会读取 `CLAUDE_TOOL_OUTPUT`，**敏感环境中不要启用**

#### 安装命令

执行 `memory-manager install-hooks` 或手动：

```bash
# 复制 hook 脚本
HOOKS_SRC="{skill_path}/hooks"
HOOKS_DST="$HOME/.openclaw/hooks/memory-manager"
mkdir -p "$HOOKS_DST"
cp "$HOOKS_SRC/activator.sh" "$HOOKS_DST/"
cp "$HOOKS_SRC/error-detector.sh" "$HOOKS_DST/"
chmod +x "$HOOKS_DST/"*.sh

# 启用
openclaw hooks enable memory-manager

# 验证
openclaw hooks list | grep memory-manager
```

#### 精细控制

```bash
openclaw hooks enable memory-manager/activator      # 只启用 Prompt 提醒
openclaw hooks enable memory-manager/error-detector # 只启用错误检测
openclaw hooks disable memory-manager               # 临时全部禁用
openclaw hooks uninstall memory-manager             # 彻底卸载
```

---

### 机制二：候选池三次晋升（防误学）

**解决的问题**：v2 中只要 Agent 发现值得持久化的经验就直接呈现进化卡片，存在将偶发指令误固化为永久规则的风险。

#### CANDIDATES.md 文件格式

```markdown
# 候选记忆池（v3）
> 由 memory-manager v3 自动维护 | 最后更新：YYYY-MM-DD
> 晋升规则：同一模式观察 ≥3 次，或用户明确指示

---

## 🌱 观察中

| ID | 类型 | 内容摘要 | 首次观察 | 最近观察 | 次数 | 目标 |
|----|------|---------|---------|---------|------|------|
| C001 | 规则 | PowerShell 删除文件用 Remove-Item 而非 rm | 2026-04-15 | 2026-04-17 | 2/3 | 3次 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2017java/super-agent-memory-manager](https://github.com/2017java/super-agent-memory-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
