---
trigger: always_on
description: ClawHub Skill 提交指南 - 从零开始创建、测试、发布 Skill 到 ClawHub 官方市场，包含完整的自动化流程、手动步骤、常见坑点解决方案，适用于无经验用户和经验开发者
---


# ClawHub Skill 提交完全指南

**版本**：v1.0.0
**更新日期**：2026-03-25
**目标用户**：无经验小白 → 成功发布 Skill 的完整路径

---

## 📋 目录

1. [流程概览](#流程概览)
2. [准备工作](#准备工作)
3. [自动化步骤（AI 完成）](#自动化步骤ai-完成)
4. [手动步骤（用户完成）](#手动步骤用户完成))
5. [常见坑点与解决方案](#常见坑点与解决方案)
6. [完整操作流程](#完整操作流程)
7. [检查清单](#检查清单)

---

## 流程概览

### 整体流程图

```
准备阶段（自动化）
    ↓
GitHub 仓库创建（自动化 + 手动确认）
    ↓
文档准备（自动化）
    ↓
敏感信息脱敏（自动化）
    ↓
推送到 GitHub（自动化）
    ↓
创建 GitHub Release（手动）
    ↓
提交到 ClawHub（手动）
    ↓
等待审核（被动）
    ↓
审核通过（完成）
```

### 角色分工

| 阶段 | AI 助手 | 用户作者 |
|------|---------|----------|
| 准备工作 | ✅ 检查 Skill 文件完整性 | - |
| 文档准备 | ✅ 生成中英文文档 | - |
| 敏感信息脱敏 | ✅ 自动检测和脱敏 | - |
| GitHub 仓库 | ⚠️ 建议创建信息，需用户确认 | ✅ 创建仓库 |
| 推送代码 | ✅ 自动推送 | - |
| GitHub Release | - | ✅ 创建 Release 和上传截图 |
| ClawHub 提交 | ⚠️ 生成提交表单内容 | ✅ 填写并提交 |
| 等待审核 | - | ✅ 跟进审核状态 |

---

## 准备工作

### 1. 检查 Skill 文件完整性

#### AI 自动检查清单

**必需文件**：
- ✅ `SKILL.md` - Skill 核心配置文件
- ✅ 至少一个实现文件（脚本/代码）

**推荐文件**：
- ✅ `README.md` - 中文使用文档
- ✅ `README_EN.md` - 英文使用文档
- ✅ `LICENSE` - 开源协议（推荐 MIT）
- ✅ `CHANGELOG.md` - 版本更新日志
- ✅ `.gitignore` - Git 忽略配置

#### 检查命令（AI 执行）

```bash
# 列出 Skill 目录下的所有文件
ls -la ~/.workbuddy/skills/your-skill-name/

# 检查必需文件是否存在
test -f ~/.workbuddy/skills/your-skill-name/SKILL.md && echo "✅ SKILL.md 存在" || echo "❌ SKILL.md 缺失"
```

### 2. 检查 SKILL.md 配置

#### 必需字段

```yaml
---
description: 你的 Skill 描述文字，100 字以内
---
```

**检查要点**：
- ✅ `description` 字段必须存在
- ✅ 描述简洁明了，100 字以内
- ✅ 避免特殊字符和表情符号
- ✅ 使用中性、专业的语言

#### 示例

```yaml
---
description: 3D 词云可视化工具 - 将对话历史或其他文本数据自动转换为炫酷的 3D 地球词云，支持多格式文件导入（JSON/MD/TXT），自动中文分词和词频统计，生成 TOP30 高频词的 3D 可视化效果
---
```

### 3. 检查 Skill 实现

#### 实现方式

ClawHub 支持多种 Skill 实现方式：

1. **Python 脚本**（最常见）
   - 单文件或多文件
   - 必须有 `__main__` 入口
   - 依赖通过 `requirements.txt` 或 `pyproject.toml`

2. **Shell 脚本**
   - Bash 脚本
   - 必须有执行权限

3. **Node.js 脚本**
   - JavaScript/TypeScript
   - 依赖通过 `package.json`

4. **纯配置型 Skill**
   - 主要是 SKILL.md 和文档
   - 实现由 WorkBuddy 核心 API 完成

---

## 自动化步骤（AI 完成）

### 步骤 1：生成完整文档

#### AI 操作

```python
# 生成 README.md（中文）
def generate_readme_cn(skill_info):
    template = """# {name}

{description}

## 功能特性

{features}

## 使用方法

{usage}

## 技术栈

{tech_stack}

## 许可证

MIT License

## English Version

[English Documentation](README_EN.md)
"""
    return template.format(**skill_info)

# 生成 README_EN.md（英文）
def generate_readme_en(skill_info):
    template = """# {name_en}

{description_en}

## Features

{features_en}

## Usage

{usage_en}

## Tech Stack

{tech_stack_en}

## License

MIT License

## 中文文档

[中文文档](README.md)
"""
    return template.format(**skill_info)
```

#### 输出内容

- ✅ `README.md` - 完整的中文使用文档
- ✅ `README_EN.md` - 完整的英文使用文档

### 步骤 2：生成 LICENSE

#### AI 操作

```python
# 生成 MIT License
def generate_mit_license(author, year):
    template = f"""MIT License

Copyright (c) {year} {author}

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
"""
    return template
```

#### 输出内容

- ✅ `LICENSE` - MIT 开源协议文件

### 步骤 3：生成 CHANGELOG.md

#### AI 操作

```python
# 生成版本更新日志
def generate_changelog(version, date):
    template = f"""# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [{version}] - {date}

### Added
- Initial release
- {feature_list}

### Features
- {detailed_features}
"""
    return template
```

#### 输出内容

- ✅ `CHANGELOG.md` - 版本更新日志

### 步骤 4：敏感信息检测与脱敏

#### AI 自动检测

**检测目标**：

1. **硬编码路径**
   - `.workbuddy/memory/`
   - `MEMORY.md`
   - `C:\Users\Administrator\`
   - `/home/username/`

2. **敏感信息**
   - API Keys
   - 密码
   - 邮箱地址
   - 个人信息

3. **工具名称硬编码**
   - 具体的工具名称（如 `python`、`npm`）
   - 绝对路径的脚本路径

#### 脱敏规则

```python
# 脱敏映射表
DESENSITIZE_MAP = {
    '.workbuddy/memory/': 'WorkBuddy 工作记忆目录',
    'MEMORY.md': '长期记忆文件',
    'C:\\Users\\Administrator\\': '~/',
    '/home/username/': '~/',
    'sk-proj-': 'sk-proj-***',
}
```

#### AI 操作

```python
def desensitize_content(content):
    for sensitive, replacement in DESENSITIZE_MAP.items():
        content = content.replace(sensitive, replacement)
    return content
```

### 步骤 5：生成 GitHub 仓库信息

#### AI 操作

```python
def generate_github_info(skill_name, github_username):
    return {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xf4vul/clawhub-skill-submission-guide](https://github.com/0xf4vul/clawhub-skill-submission-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
