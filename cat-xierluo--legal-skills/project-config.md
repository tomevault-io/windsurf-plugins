---
trigger: always_on
description: 本项目旨在沉淀并分发面向律师办案的 Claude Skills，支持诉讼与非诉场景的 AI 协作。请所有 AI 代理遵循以下约定，确保技能可复用、可追溯、可扩展。
---

本项目旨在沉淀并分发面向律师办案的 Claude Skills，支持诉讼与非诉场景的 AI 协作。请所有 AI 代理遵循以下约定，确保技能可复用、可追溯、可扩展。

## 核心原则

- **中文优先**：**所有回复必须使用中文**，无论用户使用何种语言提问。这是强制要求，不是可选项。
- **技能导向**：每个技能独立成树（例：`med-extract/`），根目录直接包含 `SKILL.md`、`references/` 等资源和配套文档（`DECISIONS.md`、`TASKS.md`、`CHANGELOG.md`）。
- **文档即上下文**：关键决策、路线、任务、变更、日志必须记录在各技能目录下的对应文档文件中。
- **透明变更**：任何对用户或协作者有影响的修改都要写入对应技能目录下的 `CHANGELOG.md`，重要决策写入 `DECISIONS.md`。
- **技能级文档**：本项目的 `CHANGELOG.md`、`DECISIONS.md`、`TASKS.md` 等文档均为**技能级别**（位于各技能目录下），**不创建项目级别的文档**。当用户要求"更新文档"时，默认指当前操作涉及的特定技能的文档。
- **保留证据**：输出引用需可回溯到来源文件；缺失信息明确标注"未提及/待补充"，避免臆测。

## 目录约定（每个技能项目）

- 根目录：`SKILL.md`（必填，含 frontmatter），`config/`、`references/`、`scripts/`、`assets/`（按需），文档文件（`DECISIONS.md`、`TASKS.md`、`CHANGELOG.md`），原始材料。

## 依赖管理规范

依赖说明应直接写在 `SKILL.md` 的"依赖"章节中。

### SKILL.md 依赖章节格式

```markdown
## 依赖

### 系统依赖

| 依赖 | 安装方式 |
|------|----------|
| 软件名 | macOS: `brew install xxx`<br>Linux: `sudo apt-get install xxx` |

### Python 包

| 包名 | 用途 | 安装命令 |
|------|------|----------|
| `package-name` | 用途说明 | `pip install package-name` |
```

### 依赖包文件（可选）

如需管理大量 Python 依赖，可在 `assets/` 或 `scripts/` 目录下使用 `requirements.txt`：

```bash
pip install -r scripts/requirements.txt
```

### 脚本依赖防护要求

**所有包含外部依赖的脚本必须做优雅降级处理**，确保用户未安装依赖时不会遇到晦涩的 `ImportError` 或 `ModuleNotFoundError`。

**规则**：

1. **硬依赖**（脚本核心功能所需）：用 try/except 包裹 import，捕获后输出清晰的安装提示并退出
2. **可选依赖**（增强功能所需）：用 try/except 包裹 import，设置 `HAS_XXX = False` 标志，在对应功能处降级处理

**示例 — 硬依赖**：

```python
try:
    from docx import Document
    from docx.shared import Pt, Cm
except ImportError:
    print("❌ 缺少依赖: python-docx")
    print("   请运行: pip install python-docx")
    print("   或运行: pip install -r scripts/requirements.txt")
    raise SystemExit(1)
```

**示例 — 可选依赖**：

```python
try:
    from PIL import Image
    HAS_PIL = True
except ImportError:
    HAS_PIL = False
```

### SKILL.md 依赖声明要求

涉及脚本的技能，必须在 SKILL.md 中明确说明：

1. **哪些功能需要依赖**：区分"开箱即用"功能和"需安装依赖"功能
2. **安装方式**：提供一条命令即可完成安装（如 `pip install -r scripts/requirements.txt`）
3. **安装位置**：在用户首次需要运行脚本的位置（如"转换为 Word 文档"章节）给出安装说明，不要藏在文档末尾

## 许可证管理规范

### 技能许可证分类

1. **法律专业应用技能**：使用 CC BY-NC（署名-非商业性使用）
   - 涉及法律业务分析、文书生成、专业咨询的技能
   - 示例：`litigation-analysis`、`legal-proposal-generator`、`legal-text-format`、`legal-qa-extractor`

2. **通用工具类技能**：使用 MIT 许可证
   - 内容获取、格式转换、开发工具等通用功能
   - 示例：`mineru-ocr`、`funasr-transcribe`、`skill-manager`、`piclist-upload`、`course-generator` 等

3. **官方技能**：保持原有许可证不变
   - `skill-creator`、`pdf`

### SKILL.md Frontmatter 许可证字段

- **法律专业应用**：`license: CC-BY-NC`
- **通用工具**：`license: MIT`
- **官方技能**：`license: Complete terms in LICENSE.txt`

### LICENSE.txt 文件

每个技能目录应包含对应的 LICENSE.txt 文件，使用完整许可证文本：

- CC BY-NC 许可证使用标准 Creative Commons  Attribution-NonCommercial 4.0 International 文本
- MIT 许可证使用标准 MIT 许可证文本

### CC BY-NC 商用许可联系方式

使用 CC BY-NC 许可证的技能，LICENSE.txt 中的商用许可联系信息必须统一为：

```text
Commercial License

For commercial use licenses, please contact:
Email: secretxierluo@gmail.com
WeChat: ywxlaw (微信)
```

**注意**：创建或更新 CC BY-NC 技能时，必须确保 LICENSE.txt 中的联系方式与此规范一致。

### 版权信息规范

所有自研技能的 LICENSE.txt 文件必须使用统一的版权信息格式：

```text
Copyright (c) 2025 杨卫薪律师（微信ywxlaw）
```

**例外情况**：

- **官方技能**（skill-creator、pdf）：保持原作者版权信息
- **协作技能**：如 mineru-ocr、funasr-transcribe 等基于外部项目的技能，可保留项目特定的版权信息（如 "MinerU Skill Contributors"）

### README.md 许可证列

在 README.md 技能列表表格中应包含"许可证"列，明确标注各技能的许可证类型。

## 文件夹存放规范（主项目）

- 正式发布的技能放在 `skills/` 目录下（如 `skills/pdf/`），调试中的技能放在 `test/` 目录下。
- 技能目录名与技能 name 保持一致。
- 示例/原始材料与技能同级放置，命名清晰，避免混入其他技能资料。

## 标准作业流程（每个技能）

1) **选择目标**：阅读 `TASKS.md`，选择首个未完成目标作为当前任务。若无目标，补充并认领。
2) **分析与计划**：理解目标输出与验收标准；必要时内部规划步骤。
3) **执行与决策记录**：

   - 代码/文档修改同步在对应技能目录下完成。
   - 涉及重要取舍或涌现任务，写入 `DECISIONS.md`（说明背景、方案、理由）。
4) **更新文档**：

   - 变更：`CHANGELOG.md` 按类别记录。
   - 任务：完成项在 `TASKS.md` 勾选，新增子任务时及时登记。

## 写作与输出要求

- SKILL.md 使用祈使/不定式语气，明确何时触发、如何操作。
- 引用具体文件请使用相对路径（示例：`med-extract/SKILL.md`），避免粘贴大段内容。
- 发现矛盾或缺失要显式提示（如"缺少出院时间，需补充"）。

### CHANGELOG.md 格式规范

- **版本号规则**：

  - 测试版本（`test/` 目录）：使用 `0.x.x` 版本号
  - 正式版本（根目录）：使用 `1.x.x` 版本号
  - 禁止使用 `Unreleased` 作为版本号
- **版本记录格式**：

  ```markdown
  ## [版本号] - YYYY-MM-DD

  ### 新增
  - 新功能描述

  ### 改进
  - 优化内容描述

  ### 技术优化
  - 技术改进描述

  ### 待办事项
  - 后续计划描述
  ```
- **分类标签**：使用 `新增`、`改进`、`修复`、`技术优化`、`文档完善` 等分类
- **内容要求**：明确说明变更的性质、影响和原因，避免模糊描述
- **文件格式**：确保文件以单个换行符结尾

### 迁移类 Skill 的版本历史保留规范

当 Skill 从现有 Command 或提示词迁移而来时，应保留原有版本历史以支持完整追溯：

- **保留场景**：Skill 从 `/illustrate`、`/extract-course` 等 Command 转换，或从其他已维护的提示词迁移而来
- **保留要求**：

  - 在 CHANGELOG 底部添加「原始版本记录（Command 时期）」或类似章节
  - 保留完整的版本迭代历史，从 v1.0 到迁移前的最后一个版本
  - 每个版本记录日期和变更摘要，确保历史可追溯
- **格式示例**：

  ```markdown
  ## 原始版本记录（Command 时期）

  ### v3.0.3 (2025-12-28)
  - 强化逻辑性动态效果优先：明确箭头绘制动画和虚线框流动动画为最高优先级

  ### v3.0 (2025-12-28)
  - 重大升级：引入动态 SVG 效果，支持角色浮动动画、虚线框流动、指向性线条动画等

  ### v1.0 (2025-10-27)
  - 初版命令框架，确立基本设计原则和流程
  ```
- **版本号延续**：Skill 的正式版本从 v1.0.0 开始，不延续原始 Command 的版本号
- **价值说明**：保留原始版本历史有助于理解功能演进脉络、设计决策依据和潜在问题追溯

## Skill 开发指南

关于 Skill 的目录结构、Frontmatter 元数据、Progressive Disclosure 设计、文档编写最佳实践等详细规范,请参阅 [SKILL-GUIDE.md](./SKILL-GUIDE.md)。

## 多技能协作

- 新增技能：使用 Skill Creator 初始化后，按上述目录约定补齐文档文件。
- 避免跨技能污染：只修改当前技能树内文件，除非明确需要共享资源。

## 安全与合规

- 避免编造事实；无法确认的信息标记"未提及/待补充"。
- 如处理未脱敏材料，提醒用户审查隐私与合规。
- 不执行破坏性命令（如 `git reset --hard`），保持用户未提交的更改。

## 敏感信息安全规范

### 核心原则


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cat-xierluo/legal-skills](https://github.com/cat-xierluo/legal-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
