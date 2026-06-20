---
trigger: always_on
description: Transform content with Qiaomu's conversational style using TWO-PASS refinement. Pass 1: Rewrite with Qiaomu style. Pass 2: Compare with original, supplement missing content. Sequential workflow with automatic execution.
---


# 乔木内容解读 v3.3

## 概述

将任意长文本内容转化为乔木风格文章，采用**真正的两遍精炼**工作流和**并发配图生成**。

**🚨 重要更新（v3.3）**：配图逻辑优化 + 文件组织规范化
- **配图提示词生成**：基于H2**对应段落内容**理解核心观点，而非仅看标题
- **文件组织规范**：草稿放工作区（content_interpretations/），终稿放根目录
- **路径修复机制**：自动检测和修复图片路径问题（papers/ → content_interpretations/）
- **完整示例流程**：从读取段落→提炼观点→设计视觉隐喻的详细步骤

**🚨 重要更新（v3.2）**：新增中文规范检查
- **标点符号**：强制使用中文标点（，。：？！），禁止英文标点
- **中英翻译**：除人名/产品名外，所有英文必须翻译成中文
- **质量检查**：在步骤1、步骤2、质量检查清单中全面添加规范要求

**🚨 重要更新（v3.1）**：优化命名和内容质量
- **主题命名**：工作区用文章主题命名，不用时间戳
- **文件命名**：用"主题-初稿/改进.md"，不用draft_v1/v2
- **禁止编造**：明确禁止编造故事、案例、数据
- **非中文处理**：增加翻译步骤

**🚨 重大更新（v3.0）**：修复workflow理解错误
- **不是两种模式**，而是**依次执行的两步workflow**
- **步骤1（初稿）**：用乔木风格重写原文（专注风格转换）
- **步骤2（改进）**：对比原文和初稿，补充遗漏内容（专注内容完整性）

**核心思想**：
- 第一遍：专注于风格转换，可能会遗漏一些细节
- 第二遍：专注于内容完整性，补充遗漏的重要内容
- 两遍结合：既有乔木风格，又有完整内容
- **绝不编造**：只使用原文的例子和数据

**核心特点**：
- ✅ 全自动执行，无需中途确认
- ✅ 真正的两遍精炼：风格转换 + 内容补充
- ✅ 并发配图生成，速度提升3倍
- ✅ 对话式语言，深度洞察
- ✅ 严格去AI感表达
- ✅ 《纽约客》风格配图

**输出**：
- 与原文等长的乔木风格文章（±10%）
- 配《纽约客》风格插画（每个H2标题一张）
- 完整工作档案（draft_v1初稿 + draft_v2改进版）
- 可对比两版，看风格转换和内容补充的过程

---

## 自动化工作流程

**🚨 CRITICAL - 执行原则（必须严格遵守）**：

1. ✅ **全自动执行**：步骤0→1→2→3→4→5，不跳过任何步骤
2. ✅ **不询问用户**：不问"是否需要配图"、"是否需要XX"，直接执行
3. ✅ **先保存文件再展示**：每一步生成内容后，**立即保存到文件**，不要只在对话中输出
4. ✅ **必须生成配图**：步骤3必须执行，不要跳过或询问用户
5. ✅ **实时更新进度**：每完成一步立即更新TodoWrite状态

**❌ 禁止行为**：
- ❌ 不要只在对话中输出文章内容而不保存文件
- ❌ 不要问用户"是否需要配图"、"是否需要XX"
- ❌ 不要跳过任何步骤
- ❌ 不要在未完成所有步骤前就告知用户"完成"

**推荐顺序**：步骤0 → 1 → 2 → 3 → 4 → 5（严格按顺序）

---

### 初始化：创建进度追踪

**第一步**：创建todo list，让用户看到进度

```python
TodoWrite([
    {"content": "准备工作空间", "status": "in_progress", "activeForm": "正在准备工作空间"},
    {"content": "生成初稿（第一遍）", "status": "pending", "activeForm": "正在生成初稿"},
    {"content": "生成改进版（第二遍）", "status": "pending", "activeForm": "正在生成改进版"},
    {"content": "生成纽约客风格配图", "status": "pending", "activeForm": "正在生成配图"},
    {"content": "保存最终文件", "status": "pending", "activeForm": "正在保存最终文件"}
])
```

**每完成一步**，立即更新状态为`completed`，下一步为`in_progress`

---

### 步骤0：准备工作空间和内容预处理

**目标**：创建工作目录，保存原始内容，如果是非中文则翻译，自动判断内容类型和目标路径

**执行步骤**：

#### 步骤0.1：自动判断内容类型并选择目标路径

**🚨 新增功能：自动识别内容类型，无需询问用户**

根据输入内容自动判断类型，并选择正确的保存路径：

| 内容类型 | 关键词 | 目标路径 | 配图风格 |
|---------|-------|---------|---------|
| **YouTube/视频访谈** | YouTube、视频、interview、访谈、对话 | `20-29 学习/23 播客转录/` | 纸雕水彩封面（4张） |
| **公众号文章** | 公众号、文章、解读、分析 | `03.项目/AI不插电/` 或 `11 公众号-向阳乔木推荐看/` | 纽约客风格插画 |
| **论文/学术** | paper、论文、arxiv、PDF、学术 | `25 论文库/21.02 论文解读/` | 学术风格配图 |
| **小红书** | 小红书、xhs、图文 | `13 小红书/` | 手绘配图 |
| **默认** | 其他 | `20-29 学习/23 播客转录/` | 纸雕水彩封面 |

**自动判断逻辑**：

```python
# 伪代码：自动判断内容类型
def get_content_type(user_input):
    if "youtube.com" in user_input or "YouTube" in user_input or "访谈" in user_input:
        return "podcast"  # 播客访谈
    elif "公众号" in user_input or "文章" in user_input:
        return "wechat"   # 公众号文章
    elif "论文" in user_input or "arxiv" in user_input or ".pdf" in user_input:
        return "paper"    # 学术论文
    elif "小红书" in user_input or "xhs" in user_input:
        return "xiaohongshu"
    else:
        return "podcast"  # 默认当播客访谈处理
```

**🚨 重要**：判断出内容类型后，**不要询问用户**，直接使用对应的路径和配图风格

#### 步骤0.2：提炼文章主题，创建工作目录

**重要**：不要用时间戳，要用文章核心主题命名

```bash
# 根据内容类型选择基础路径
base_path = get_base_path(content_type)  # 如 "20-29 学习/23 播客转录/"

cd "/Users/joe/乔木新知识库/{base_path}"

# 1. 快速阅读原文，提炼核心主题（3-5个字）
# 例如："AnthropicCEO访谈"、"大鹏的十年"、"AI时代的选择"

# 2. 用主题+日期创建工作目录（避免重名）
work_dir="content_interpretations/停下来才能想清楚"
mkdir -p "$work_dir/images/illustrations"
```

#### 步骤0.3：保存原文并判断语言

**使用.md格式保存原文**：

```python
# 保存原始内容（使用.md格式）
Write(
    file_path="{work_dir}/original_content.md",
    content="{user_provided_content}"
)
```

**判断语言**：
- 如果是中文：直接进入步骤1
- 如果是非中文（英文等）：执行步骤0.3

#### 步骤0.3：翻译非中文内容（如需要）

**如果原文是非中文**，先翻译成中文：

```markdown
请将以下内容翻译成中文，保留原文的完整信息：

{original_content}

翻译要求：
- 准确传达原意
- 保留所有细节、数据、引用
- 自然流畅的中文表达
- 不要添加原文没有的内容
```

**保存翻译版本（使用.md格式）**：

```python
Write(
    file_path="{work_dir}/translated_content.md",
    content="{translated_content}"
)
```

**后续步骤使用translated_content作为"原文"**

**输出结构**（工作区 + 终稿）：

```
AI不插电/                                      ← 根目录
├── 写文案就像做菜.md                          ← 终稿（步骤4生成，无H1标题）
│
└── content_interpretations/                   ← 工作区（所有中间产物）
    └── 停下来才能想清楚/                      ← 用主题命名
        ├── original_content.md                ← 原始内容（可能是英文）
        ├── translated_content.md              ← 翻译版本（如果需要）
        ├── 停下来才能想清楚-初稿.md           ← 初稿（第一遍，保留H1）
        ├── 停下来才能想清楚-改进.md           ← 改进版（第二遍，保留H1）
        ├── visual_config.json                 ← 配图配置
        └── images/
            └── illustrations/                 ← 配图目录
                ├── illustration_1.png
                ├── illustration_2.png
                └── ...
```

**🚨 关键点 - 文件组织原则**：

1. **草稿在工作区**：
   - 所有中间版本（初稿、改进版）都放在`content_interpretations/{主题}/`
   - 保留H1标题，方便对比不同版本
   - 包含所有配图和配置文件

2. **终稿在根目录**：
   - 步骤4用`finalize_markdown.py`脚本处理
   - 提取H1标题作为文件名
   - 从文章中删除H1行
   - 图片路径指向工作区的`content_interpretations/{主题}/images/`

3. **路径引用规范**：
   - 工作区内的文件：相对路径`images/illustrations/illustration_1.png`
   - 根目录的终稿：相对路径`content_interpretations/{主题}/images/illustrations/illustration_1.png`
   - 脚本会自动处理路径转换

**完成后**：记录工作目录路径，更新todo状态


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joeseesun/qiaomu-content-interpreter](https://github.com/joeseesun/qiaomu-content-interpreter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
