---
trigger: always_on
description: 使用 Claude Code + .claude/ 指令实现需求测试和测试用例生成的完整流程，帮助系统测试工程师提高工作效率。
---

# TWU - 测试用例生成工作流

## 项目背景

使用 Claude Code + .claude/ 指令实现需求测试和测试用例生成的完整流程，帮助系统测试工程师提高工作效率。

通过 command + skills 编排，自动化处理需求解析、需求评审、测试规划和用例生成等环节。

## 核心原则

### 文档产出规范

所有产出的文档必须遵循：

1. 清晰：结构清晰，一目了然
2. 简洁：不啰嗦，不重复
3. 结构化：格式统一，易于阅读和审核
4. 完整：覆盖所有必要信息

禁止产出冗长、复杂、难以阅读的文档。

---

## 目录结构

```
twu/                           # 项目根目录（Claude Code 运行目录）
├── .claude/                   # 配置目录（所有需求共享）
│   ├── commands/              # 斜杠命令定义
│   │   ├── req-parse.md       # /req-parse - 需求解析
│   │   ├── req-test.md        # /req-test - 需求评审
│   │   ├── req-merge.md       # /req-merge - 需求合并
│   │   ├── testcase-plan.md   # /testcase-plan - 测试规划
│   │   └── testcase-gen.md    # /testcase-gen - 用例生成
│   ├── skills/                # 技能实现
│   │   ├── req-parser/        # 需求解析 Skill
│   │   ├── req-tester/        # 需求评审 Skill
│   │   ├── req-merger/        # 需求合并 Skill
│   │   ├── testcase-planner/  # 测试规划 Skill
│   │   └── testcase-generator/# 用例生成 Skill
│   ├── hooks/                 # 会话钩子
│   └── settings.json          # 配置
├── CLAUDE.md                  # 业务背景和流程说明（本文档）
├── scripts/                   # 辅助脚本
│   └── init.py                # 项目初始化
│
├── 需求1/                     # 需求子目录（通过 init.py 创建）
│   ├── original-requirements/ # 原始需求文档（用户上传）
│   │   ├── PRD.pdf
│   │   ├── 原型图.docx
│   │   └── 接口文档.txt
│   ├── cleaned-requirements/  # 解析后需求
│   │   ├── index.md           # 合并后的完整需求文档
│   │   ├── chunks/            # 解析片段（按原文件名）
│   │   │   ├── PRD.md
│   │   │   ├── 原型图.md
│   │   │   └── 接口文档.md
│   │   ├── assets/            # 图片资源
│   │   │   ├── figure-1.png
│   │   │   └── figure-2.png
│   │   └── issues.md          # 需求问题清单（结构化格式）
│   ├── clarified-requirements/# 完备需求
│   │   ├── index.md           # 回填问题答案后的完整需求
│   │   └── assets/            # 图片资源（从 cleaned 复制）
│   └── test-case/             # 测试用例
│       ├── plan.md            # 测试规划（## ITEM + ### POINT + > 备注）
│       ├── {ITEM}/            # 按测试项分目录
│       │   ├── {POINT1}.md    # 测试用例文件（Markdown Text Protocol v0.2）
│       │   ├── {POINT2}.md
│       │   └── ...
│       ├── all_cases.md       # 所有用例的汇总文件
│       └── export.xlsx        # 导出的 Excel 格式（可选）
│
└── 需求2/                     # 其他需求项目
    └── ...
```

---

## 核心流程

### 准备阶段：初始化需求目录

```bash
uv run scripts/init.py --name "需求名称"
```

**产物**：创建需求子目录及 4 个子文件夹（original-requirements, cleaned-requirements, clarified-requirements, test-case）

**下一步**：将原始需求文档（PDF、Word、图片等）上传到 `需求名称/original-requirements/` 目录

---

### 阶段 1：需求解析 (Parse)

**命令**：`/req-parse`

**触发 Skill**：`req-parser`

**输入**：`需求名称/original-requirements/`（PDF、Word、纯文本、图片等）

**处理流程**：
1. 扫描并分类原始文件（PDF/Word/图片/文本）
2. 使用 Docling 解析 PDF/Word 文档，提取文本、表格和图片
3. 使用多模态能力分析图片内容（原型图、界面截图等），生成详细描述
4. 修复 Docling 解析产生的格式错误（特殊字符、列表序号、标题标记等）
5. 合并所有解析片段到 `cleaned-requirements/index.md`

**产物**：
- `cleaned-requirements/index.md` - 完整的需求文档
- `cleaned-requirements/chunks/` - 各文件的解析片段
- `cleaned-requirements/assets/` - 提取的图片资源

**人工审核重点**：
- [ ] 图片描述是否准确（特别是原型图和界面截图）
- [ ] 表格是否完整（无错行、错列）
- [ ] 格式是否清晰（标题、列表、段落）
- [ ] 有无乱码或缺失内容

**下一步**：人工修正 `index.md`，确认无误后进入评审阶段

---

### 阶段 2：需求评审 (Test)

**命令**：`/req-test`

**触发 Skill**：`req-tester`

**输入**：`cleaned-requirements/index.md`

**处理流程**：
1. 站在测试人员角度分析需求
2. 识别影响测试用例设计的模糊点、缺失点和矛盾点
3. 生成结构化的问题清单（issues.md）

**产物**：`cleaned-requirements/issues.md`（结构化格式，详见下方）

**下一步**：测试工程师在 `issues.md` 中填写 `[产品回答]` 字段

---

### issues.md 格式说明

```markdown
# 需求评审问题清单

> **需求名称**: [需求标题]
> **生成时间**: 2026-01-21 10:30
> **问题总数**: 15 个
> **优先级分布**: P0: 5 | P1: 7 | P2: 3

---

## 问题 1 [前因后果 · critical]

**[位置]** 第3页 - 订单支付功能

**[来源]**
需求文档第3页提到"超时未支付订单自动关闭"，但未说明：
- 超时时长是多少？
- 关闭后库存是否恢复？

**[测试视角]**
作为测试人员，需要理解超时逻辑，才能：
1. 设计边界条件用例（临界时间点）
2. 验证库存回滚的正确性

**[需澄清]**
1. 超时时长是多少分钟？
2. 订单关闭后，库存是立即释放还是延迟释放？

**[回答建议]**
- 超时时长：15分钟 / 30分钟 / 其他
- 库存释放：立即 / 延迟5分钟

**[产品回答]**
<!-- 在此填写答案 -->

---

## 问题 2 [边界定义 · warning]
...
```

**分类枚举**：前因后果、边界定义、异常场景、交互细节、数据状态、性能兼容

**优先级枚举**：critical（不明确无法设计用例）、warning（影响场景识别）、info（补充说明）

**填写规则**：
- 在 `[产品回答]` 下方直接填写答案
- 可以是纯文本、列表或表格
- 必须针对 `[需澄清]` 中的问题逐一回答

---

### 阶段 3：需求完善 (Merge)

**命令**：`/req-merge`

**触发 Skill**：`req-merger`

**输入**：
- `cleaned-requirements/index.md`
- `cleaned-requirements/issues.md`（已填写答案）

**处理流程**：
1. 读取 `issues.md` 中的 `[产品回答]`
2. 将答案回填到需求文档的对应位置
3. 整合为完备需求文档

**产物**：`clarified-requirements/index.md`（完备需求文档）

**人工审核重点**：
- [ ] 所有问题的答案是否已回填
- [ ] 回填内容是否与原文档融合自然
- [ ] 有无遗漏或冲突的信息

**下一步**：确认需求完备后，进入测试规划阶段

**注意**：如需大幅调整，可直接编辑 `clarified-requirements/index.md`，或返回上一步重新执行

---

### 阶段 4：测试规划 (Plan)

**命令**：`/testcase-plan`

**触发 Skill**：`testcase-planner`

**输入**：`clarified-requirements/index.md`

**处理流程**：
1. 识别业务模块（ITEM）：按需求章节或业务实体划分
2. 识别测试场景（POINT）：基于场景法识别独立操作路径
3. 评估风险等级（Critical / High / Medium / Low）
4. 提炼测试关注点（边界值、异常情况、业务规则等）

**产物**：`test-case/plan.md`（结构化测试规划）

**测试规划格式**：

```markdown
# 测试规划

## 登录功能

### 用户名密码登录
> 风险等级: Critical
> 测试关注点: 密码错误、账户锁定、密码长度边界（8-20位）

### 手机验证码登录
> 风险等级: High
> 测试关注点: 验证码过期（60s）、错误次数限制（5次）

### 第三方登录（微信/支付宝）
> 风险等级: Medium
> 测试关注点: 授权失败、token过期处理

## 订单管理

### 创建订单
> 风险等级: Critical
> 测试关注点: 库存扣减、价格计算、并发下单

### 取消订单
> 风险等级: High
> 测试关注点: 库存回滚、退款流程、取消时间限制（30分钟内）
```

**层级说明**：
- **ITEM（测试项）**：业务模块或需求章节，如"登录功能"、"订单管理"
- **POINT（测试点）**：独立的操作路径，如"用户名密码登录"、"手机验证码登录"
- **备注（> 风险等级 / 测试关注点）**：规划阶段的思考，用于指导用例生成

**人工审核重点**：
- [ ] 测试项划分是否合理（无遗漏、无重复）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chyax98/twu](https://github.com/chyax98/twu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
