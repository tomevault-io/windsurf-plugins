---
trigger: always_on
description: 本文件是 Claude Cream 仓库根目录的项目级 AI Coding Agent 规范。
---

# AGENTS.md

本文件是 Claude Cream 仓库根目录的项目级 AI Coding Agent 规范。

适用工具：

- Codex：默认读取根目录 `AGENTS.md`
- Claude Code：通过根目录 `CLAUDE.md` 导入本文件
- 其他能读取 `AGENTS.md` 的编码智能体

使用约定：

1. 个人长期偏好放全局规范；本文件只写项目级约束。
2. 子目录另有 `AGENTS.md` 时，优先遵守距离当前文件更近的规则。
3. 不存在的命令、目录、文档不要写成已可用。

## 1. 项目概况

### 项目名称

```text
Claude Cream
```

### 项目定位

```text
暖色调设计 token 与主题资产库，为 Codex、Cursor / VS Code、Zed、Typora、Obsidian、Ghostty、Website 与插画生成提供统一视觉语言。
```

### 主要用户

```text
个人维护者；需要在编辑器、终端、网站与生成式插画间保持同一套奶油 + 珊瑚视觉体系。
```

### 核心目标

```text
以 tokens/tokens.json 为 Codex、Cursor / VS Code、Zed、编辑器与终端主题的单一真源，同时独立管理 Website 色板与图像生成规范。
```

### 非目标

当前阶段不处理：

- 不做 Web 应用、包管理发布流水线或在线主题商店
- 不引入 npm / 构建器作为默认安装路径（安装靠 `cp`）
- 不引入付费字体或必须联网才能生效的依赖

不要主动实现非目标中的内容。

---

## 2. 技术栈

### 前端

```text
暂无（主题为 CSS / 终端配置 / 客户端模板文件）
```

### 后端

```text
暂无
```

### 数据库

```text
暂无
```

### UI 与样式

```text
Design Tokens（JSON）→ Cursor / VS Code Theme JSON、Zed Theme JSON、Typora CSS、Obsidian CSS、Ghostty palette、CLI 配置模板
```

### 状态管理

```text
无
```

### 包管理器

```text
无；安装与分发通过文件复制
```

### 测试体系

```text
暂无自动化测试；以视觉核对、文件名约束检查、git diff --check 为主
```

### 部署方式

```text
本机 cp 安装到各客户端主题 / 配置目录
```

---

## 3. 常用命令

执行任务前先查看本节，不要猜测命令或混用包管理器。

### 安装依赖

```bash
暂无（无包管理器）
# 代码字体建议本机安装 JetBrainsMono Nerd Font Mono
```

### 本地开发

```bash
# 无 dev server；改 tokens / themes 后按 README 复制到目标客户端验证
```

### 类型检查

```bash
暂无
```

### 单元测试

```bash
暂无
```

### 集成或端到端测试

```bash
# Typora（macOS 示例）
cp themes/typora/*.css themes/typora/.claude-theme-base.css \
  "$HOME/Library/Application Support/abnerworks.Typora/themes/"

# Obsidian（按本机 vault 路径调整）
cp -R themes/obsidian "$HOME/Dev/obsidian-wiki/.obsidian/themes/Claude Cream"

# Ghostty
mkdir -p "$HOME/.config/ghostty/themes"
cp themes/ghostty/config.ghostty "$HOME/.config/ghostty/config"
cp themes/ghostty/claude-cream-light themes/ghostty/claude-cream-dark \
  "$HOME/.config/ghostty/themes/"

# Cursor（macOS / Linux）
mkdir -p "$HOME/.cursor/extensions"
rm -rf "$HOME/.cursor/extensions/kakarrot.claude-cream-0.2.0"
cp -R themes/vscode "$HOME/.cursor/extensions/kakarrot.claude-cream-0.2.0"

# Cursor / VS Code 主题静态验证
themes/vscode/scripts/validate-theme.sh

# 全平台主题静态验证
python3 scripts/validate.py

# Zed（macOS / Linux）
mkdir -p "$HOME/.config/zed/themes"
cp themes/zed/claude-cream.json "$HOME/.config/zed/themes/"

# Zed 主题静态验证
jq empty themes/zed/claude-cream.json
```

### Lint

```bash
git diff --check
# 如改 SVG：可用 xmllint 检查结构
```

### 构建

```bash
暂无（无构建产物流水线；token 到主题文件目前为手工映射）
```

### 格式化

```bash
暂无
```

### 数据库迁移

```bash
暂无
```

如果某项命令不存在，写「暂无」，不要自行创建新的工具链，除非任务明确要求。

---

## 4. 目录职责

```text
tokens/
  tokens.json             设计 token 单一真源（SSOT）
  README.md               token 分组与生成说明
themes/
  codex/                  Codex Light/Dark 可导入主题与说明
  vscode/                 Cursor / VS Code 五模式主题、验证脚本与视觉 Fixtures
  zed/                    Zed Light/Dark 本地主题与安装说明
  typora/                 Typora Light/Dark 主题 CSS
  obsidian/               Obsidian 主题 + Style Settings
  ghostty/                Ghostty 主题与主配置
  website/                Website Light/Dark 色彩主题
  image-generation/      插画、个人社交头像与壁纸生成风格及提示词
img/                      Logo、横幅、效果图
tasks/
  specs/                  功能规格
  plans/                  开发计划
README.md / README.zh-CN.md
AGENTS.md                 本仓规范（本文件）
CLAUDE.md                 本仓 Claude Code 入口
```

目录规则：

- 修改前先理解目录职责和现有调用关系。
- 新文件放入职责最匹配的现有目录。
- 不创建语义重复的目录。
- 不移动、重命名或整理与任务无关的文件。
- 不为了个人偏好调整现有目录结构。
- 跨层调用必须遵守项目现有架构边界。
- 改颜色 / 字体 / 间距时先改 `tokens/tokens.json`，再同步各平台产物。

---

## 5. 相关项目文档

开始相关任务前，按需读取：

- `README.md` / `README.zh-CN.md`：安装、结构、设计原则
- `tokens/README.md`：token 分组与生成约定
- `themes/codex/README.md`：Codex 主题导入、token 映射与兼容性说明
- `themes/vscode/README.md`：Cursor / VS Code 主题、GitHub 下载与跨平台安装说明
- `themes/zed/README.md`：Zed 本地主题安装、token 映射与验证说明
- `tasks/specs/`：已确认需求
- `tasks/plans/`：已确认实现计划

不存在的文件不需要创建，除非任务确实需要。当前无根级 `DESIGN.md`、`ARCHITECTURE.md`、`CONTRIBUTING.md`、`tasks/lessons.md`。

当本文档与更具体目录中的 `AGENTS.md` 冲突时，开发本仓以根目录本文件优先。

---

## 6. 实现原则

### 6.1 先理解再修改

修改代码前：

- 阅读目标文件及其直接调用方。
- 理解现有数据流、状态流和错误处理方式。
- 检查仓库中是否已有相同或相近实现。
- 确认项目现有命名、类型和组件模式。

不要在未理解现有实现时大范围重写。

### 6.2 最小实现

- 选择满足当前需求的最简单方案。
- 不增加需求之外的功能。
- 不提前设计尚未需要的扩展能力。
- 不为一次性逻辑创建复杂抽象。
- 不引入没有明确收益的配置项或依赖。
- 优先复用项目现有组件、工具函数和基础设施。

### 6.3 最小改动

- 只修改与当前任务直接相关的代码。
- 保持现有架构、风格、命名和格式。
- 不顺手重构相邻代码。
- 不格式化与本次任务无关的文件。
- 不清理原有死代码或历史问题，除非用户明确要求。
- 删除因本次修改产生的无用导入、变量、函数和文件。

每一处改动都应能够追溯到当前需求。

### 6.4 解决根因

修复问题时：

- 优先定位根因，不通过隐藏错误、吞掉异常或硬编码结果掩盖问题。
- 不使用仅对当前示例有效的临时补丁。
- 不为了让测试通过而削弱真实业务约束。
- 无法彻底解决时，明确说明限制和剩余风险。

---

## 7. 决策与询问边界

用户目标明确且风险可控时，直接执行，不重复确认。

出现以下情况时，应先说明问题并询问：

- 存在会明显改变结果的关键需求歧义。
- 需要删除数据、覆盖文件或执行不可逆操作。
- 涉及核心架构、数据库模型、权限或安全模型调整。
- 需要新增重量级依赖或替换现有技术栈。
- 实际工作范围明显超出用户原始要求。
- 多种方案在成本、兼容性或维护性上存在重大差异。
- 改动主色、字体策略或破坏多平台一致性的视觉决策。

轻微实现细节由 Agent 根据现有代码和最简单方案自行决定。

---

## 8. 任务规模与工作流

### 8.1 小任务

包括：

- 文案或样式微调
- 小配置修改
- 明确的单点 bug
- 单文件局部修改
- 用户已经给出准确修改位置

处理方式：

1. 定位相关代码。
2. 直接修改。
3. 运行最相关的验证。
4. 简短汇报结果。

执行前说明：

```text
目标：
改动位置：
验证方式：
```

不创建 Spec、Plan 或任务管理文件。

### 8.2 中型任务

包括：

- 新增局部功能
- 修改多个相关文件
- 调整非核心业务逻辑
- 需要增加测试的功能修改

处理方式：

1. 在回复中给出简短实施步骤和验证方式。
2. 直接执行，无需等待二次确认。
3. 按最小可验证单元修改。
4. 完成后运行相关检查。

通常不创建独立 Spec 或 Plan 文件。若影响面小、用户要求直接改，也可跳过，但必须说明假设和验证方式。

### 8.3 大型或高风险任务

包括：

- 跨模块或跨应用改动
- 核心架构调整
- 数据库结构变更
- 登录、权限、安全、支付
- 多端联动
- 生产数据迁移
- 需求边界存在重大歧义

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kakarrot-dev/claude-cream](https://github.com/kakarrot-dev/claude-cream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
