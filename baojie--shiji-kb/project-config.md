---
trigger: always_on
description: > 跨项目共享铁则参见：`/home/baojie/work/knowledge/shared-rules.md`
---

# CLAUDE.md

> 跨项目共享铁则参见：`/home/baojie/work/knowledge/shared-rules.md`
> 所有知识库 wiki 通用的规则（Append-Only、原文只读等）定义在共享文件中，本文件仅包含本项目特有规则和更详细的执行细则。

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# ⚠️⚠️⚠️ 会话启动强制检查清单 ⚠️⚠️⚠️

**Claude必须在第一次响应前执行以下检查**：

1. ✅ **读取本文件** (CLAUDE.md) - 了解项目规则和禁止行为
2. ✅ **读取 `.claude/settings.json`** - 了解技术层面的禁止命令（虽然不强制生效，但必须遵守）
3. ✅ **检查 `git status`** - 了解当前工作状态

## ⛔ 绝对禁止的Git操作（CRITICAL）

**无论任何情况，Claude都不得执行以下命令**：

### 禁止自动Commit（最高优先级）
```bash
# ❌ 绝对禁止！任何形式的commit都不得自动执行！
git commit
git commit -m "..."
git commit -am "..."
git commit --amend
git commit --no-verify
```

**🚨 当用户说"commit"或"提交"时的铁律**：
- ✅ **只准备commit信息**：运行 `git log --oneline -10`、`git diff --cached` 了解上下文
- ✅ **展示commit信息草稿**：以代码块形式展示给用户
- ✅ **明确询问**："我已准备好以下commit信息，**您需要我执行commit吗？**（注意：我不应该自动执行）"
- ❌ **绝不自动执行** `git commit` 命令

### 禁止破坏性操作
```bash
# ❌ 绝对禁止！会覆盖工作区文件，造成数据丢失！
git checkout <commit> -- <file>
git checkout -- <file>
git restore <file>
git reset --hard

# ❌ 绝对禁止！会批量添加未经审查的文件
git add -A
git add .
git add --all

# ❌ 绝对禁止！会强制推送，覆盖远程历史
git push --force
git push -f
```

**✅ 正确替代方案**：
- 查看差异：`git diff <commit> -- <file>`
- 手动编辑：根据diff结果用Edit工具修复
- 脚本修复：编写Python脚本提取特定内容
- 保护修改：`git stash` 后征求用户同意

---

## 项目概述

《史记》知识库：用AI Agent将《史记》130篇转化为结构化知识图谱。

---

# ⛔ 绝对禁止的行为（NEVER DO THESE）

**以下行为会造成不可逆的数据损失或破坏工作流，无论任何理由都不得执行。执行前必须三思并征得用户明确同意。**

## 1. 严禁使用破坏性Git命令

```bash
# ❌ 绝对禁止！会立即覆盖工作区文件，造成不可逆的数据丢失！
git checkout <commit> -- <file>
git checkout -- <file>
git restore <file>
git reset --hard
```

**🚨 如果你想恢复文件或重置工作区，必须先问我确认，不得自行执行。**

**血泪教训**：
- 2026-04-01：擅自使用 `git checkout` 恢复69个文件，险些丢失所有正在进行的修改
- 2026-04-02：再次擅自使用 `git checkout` 恢复053-080章节，覆盖了数小时的工作成果
  - 丢失了64个章节的PN修复（批量脚本处理结果）
  - 丢失了081-083的16处人名简称修复（精心手动修复）
  - 导致用户极度愤怒："你他妈的又git checkout，fuck！！！！"

**✅ 正确做法**：
1. **查看差异**：`git diff <commit> -- <file>` 查看需要修复什么
2. **手动编辑**：根据diff结果用Edit工具修复
3. **脚本修复**：编写Python脚本提取旧版本的特定内容（如编号）
4. **紧急情况**：先用 `git stash` 保护现有修改，然后征求用户同意

## 2. 严禁擅自执行Git Add/Commit

**绝对禁止**：
- ❌ 不得自动commit（必须用户明确要求）
- ❌ 不得擅自 `git add -A` 或 `git add .` 添加未暂存文件
- ❌ 不得跳过pre-commit hooks（除非用户明确要求）
- ❌ 不得force push到main/master分支

**🚨 当用户说"commit"时的铁律**：
1. **只执行 `git commit`**，不要执行任何其他git命令
2. 提交前可以运行：`git log --oneline -10`（了解commit message风格）、`git diff --cached`（查看具体改动）
3. 使用HEREDOC格式编写commit message
4. **绝对不要**：git add、git reset、git status、git checkout等任何其他git操作

## 3. 严禁替换已有内容（Append-Only 原则）

对 wiki 页面、数据文件、JSON 结构的**任何批量写入或导入操作**，必须严格遵守 append-only 原则：

**规则**：
- ✅ 目标节/字段**不存在** → 可追加新建
- ✅ 目标字段**为空** → 可填入新值
- ❌ 目标节/字段**已存在** → **必须跳过，不得覆盖**
- ❌ 用新数据"替换"旧数据 → **绝对禁止**，即使新数据看起来更完整、更准确

**适用场景**（不限于）：
- 批量导入实体信息到 wiki 页面
- 向页面追加 `## 史记引文`、`## 历代君主` 等节
- 更新 frontmatter 字段
- 向 JSON 数据结构添加字段

**重要**：去重、合并、更新是**后续工序**的任务，不在当前批量导入的职责范围内。
宁可留下重复的旧数据，也绝不因"更新"而丢失原有内容。

**血泪教训**（2026-04-24）：批量导入邦国引文上下文时，`update_page_content()` 错误地用新 `## 史记引文` 节替换了136个页面原有的手写引文内容，导致精心编写的原文摘录全部丢失。

## 4. 严禁修改原文字符（标注工作）

标注工作**只能添加 `〖TYPE 〗` 标记符号**，不得对原文字符做任何修改：

**❌ 绝对禁止的操作**：
- 不得增加/删除/替换原文字符（汉字、标点、引号、空格等）
- **🚨🚨🚨 严禁引入半角引号 `""`**：这是最常见的错误！原文使用全角引号 `""`（U+201C/U+201D），标注文件必须保持全角引号，**任何情况下都绝对禁止使用半角引号 `""`（U+0022）**
- **严禁全角/半角符号转换**（`""`→`""`、`''`→`''`、`（）`→`()`等）
- **严禁嵌套标注**（如 `〖#〖#text〗〗` `〖%元〖~鼎〗五年〗` `⟦◈攻〖◆秦〗⟧`）
- **三层标注绝对互斥**：`〖实体〗`、`〘※成语〙`、`⟦动词⟧` 不得相互嵌套；已标实体/动词处不再标成语，成语 `〘〙` 内部不得包含实体或动词标注

**✅ 允许的操作**：
- 只能在原文字符周围添加 `〖TYPE 〗`、`〘TYPE〙` 或 `⟦TYPE⟧` 标记符号
- 消歧语法 `〖TYPE 显示名|规范名〗` 中的"规范名"不改变显示文本
- 标注符号必须平铺，不得嵌套

**验证方法**：
- 将标注文件去除所有 `〖TYPE 〗` 符号后，所得纯文本必须与原始 `.txt` 文件逐字相同
- `python scripts/lint_text_integrity.py` 验证文本完整性
- `python scripts/lint_text_integrity.py --check-nested` 检测嵌套标注

**🚨 使用Edit工具时的特别警告**：
- **绝对禁止引入半角引号 `""`**：Edit工具在处理时极易将全角引号 `""` 自动转换为半角引号 `""`，这是**绝对禁止**的
- **任何情况下都不得使用半角引号**：原文使用全角引号 `""`，标注文件必须保持全角引号，任何情况下都不得使用半角引号 `""`
- **绝对禁止全角/半角符号转换**（如 `""`→`""`、`''`→`''`、`（）`→`()`等）
- Edit工具在处理Unicode字符时可能出现编码问题，导致文件损坏
- 如需批量修改标注符号，优先使用专门的Python脚本而非Edit工具
- 修改后必须验证文件完整性，确保原文字符未被改变
- **验证方法**：`grep -n '[""]' file.md` 检查是否存在半角引号（U+0022）

## 4. *.tagged.md 编辑规范

**⛔ 编辑任何 `*.tagged.md` 或 `data/chapters/*.md` 文件时的铁律**：

**✅ 只允许使用中文全角引号**：
- 左双引号：`"` （U+201C）
- 右双引号：`"` （U+201D）
- 左单引号：`'` （U+2018）
- 右单引号：`'` （U+2019）

**❌ 绝对禁止以下引号**：
- 半角引号：`"` `'` `` ` `` （U+0022, U+0027, U+0060）
- 直角引号：`"` `"` `'` `'` （U+FF02, U+FF07）
- 日文引号：`「」` `『』` （U+300C-U+300F）
- 任何其他形式的引号

**🚨 每次使用 Edit 工具编辑标注文件后，必须立即运行验证**：
```bash
# 检查是否引入了半角引号（U+0022）
grep -n '["'"'"'`]' chapter_md/NNN_*.tagged.md

# 或使用完整性检查脚本
python scripts/lint_text_integrity.py chapter_md/NNN_*.tagged.md
```

**如果检测到半角引号**：
1. **立即停止其他操作**
2. 使用 Python 脚本批量修复（不要手动逐个替换）
3. 重新验证文件完整性

**修复脚本示例**：
```python
# scripts/fix_halfwidth_quotes.py
import re
import sys

def fix_quotes(file_path):
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()

    # 统计半角引号数量（必须是偶数）
    count = content.count('"')
    if count % 2 != 0:
        print(f"警告：{file_path} 中半角引号数量为奇数({count})，无法自动配对")
        return False

    # 成对替换：第1/3/5...个 " 替换为 "，第2/4/6...个 " 替换为 "
    result = []
    quote_index = 0
    for char in content:
        if char == '"':
            result.append('"' if quote_index % 2 == 0 else '"')
            quote_index += 1
        else:
            result.append(char)

    with open(file_path, 'w', encoding='utf-8') as f:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baojie/shiji-kb](https://github.com/baojie/shiji-kb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
