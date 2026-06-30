---
trigger: always_on
description: 本文件用于约束 Codex 在本仓库中的 Git 操作、提交日志生成、版本发布流程。
---

# AGENTS.md

本文件用于约束 Codex 在本仓库中的 Git 操作、提交日志生成、版本发布流程。

本项目是一个在 GitHub 上维护的开源项目，日常开发场景主要包括：

- 优化已有功能
- 修复 bug
- 新增功能
- 更新文档
- 发布新版本

Codex 在执行 Git 相关操作时，必须严格遵守本文件中的规范。

---

## 一、AGENTS.md 使用规则

### 1. 文件命名规则

如果希望 Codex 自动读取项目规则，文件名应使用：

```text
AGENTS.md
```

不要将本文件改名为：

```text
GIT.md
SKILL.md
AGENT.md
agents.md
```

如果项目中已经存在 `AGENTS.md`，不要新建第二个同名文件，也不要直接覆盖原文件。应该把本文件中的 Git 规范、提交流程、发布流程合并到已有 `AGENTS.md` 中。

### 2. 多个 AGENTS.md 的处理方式

如果仓库里已经有其他 `AGENTS.md`，优先按以下方式处理：

1. 根目录已有 `AGENTS.md`：把本文件内容合并进去，建议放在 `Git 工作流规范` 章节。
2. 子目录已有 `AGENTS.md`：子目录规则只用于该子目录相关任务，根目录仍应保留全局 Git 规范。
3. 如果已有 `AGENTS.md` 中的规则和本文件冲突，以更具体的规则为准；如果都是全局规则，以用户最新明确要求为准。
4. 不要因为新增 Git 规范而删除已有项目规则、构建规则、测试规则或代码风格规则。

### 3. 推荐组织方式

推荐在根目录 `AGENTS.md` 中使用以下结构：

```text
# AGENTS.md

## 项目说明
## 开发规范
## 测试规范
## Git 工作流规范
## 发布流程规范
```

本文件主要提供：

```text
Git 工作流规范
发布流程规范
```

---

## 二、基础原则

### 1. 所有关键操作必须经过用户确认

以下内容必须先展示给用户，并等待用户明确确认后才能继续执行：

- commit 提交日志
- `git add`
- `git commit`
- `git push`
- release 分支创建或切换
- tag 创建
- GitHub Release 标题
- GitHub Release 内容
- GitHub Release 创建
- 合并 release 分支到 main

用户没有明确确认之前，不允许直接执行提交、推送、打 tag、创建 GitHub Release。

### 2. 禁止危险操作

除非用户明确要求，否则禁止执行以下命令：

```bash
git reset --hard
git push --force
git clean -fd
git branch -D
git push origin --delete
```

尤其注意：发布完成后不要删除 release 分支。

---

## 三、分支命名规范

### 1. 分支命名格式

分支名称统一使用：

```text
类型-简短描述
```

类型和描述之间使用 `-` 分隔，不使用 `/`。

### 2. 常用分支类型

```text
feature-xxx     新功能开发
fix-xxx         普通问题修复
hotfix-xxx      紧急问题修复
release-x.x.x   版本发布分支
docs-xxx        文档修改
refactor-xxx    代码重构
chore-xxx       构建、配置、依赖等调整
perf-xxx        性能优化
```

### 3. 分支命名示例

推荐：

```text
feature-ui-query
feature-screenshot-api
fix-click-position
fix-rootless-install
hotfix-launch-crash
release-1.1.0
docs-update-readme
refactor-element-parser
chore-update-makefile
perf-optimize-screenshot
```

不推荐：

```text
feature/ui-query
release/v1.1.0
release-v1.1.0
fix_click_position
Feature-Login
我的分支
update
```

### 4. main 分支规则

`main` 分支代表当前最新稳定版本。

发布完成后，release 分支必须合并回 `main`，保证 `main` 始终对应最新正式发布版本。

---

## 四、提交日志规范

### 1. 提交日志格式

提交日志统一使用：

```text
类型: 提交说明
```

提交说明使用中文，要求简洁、明确。

### 2. 常用提交类型

```text
feat      新功能
fix       修复问题
docs      文档修改
refactor  代码重构
style     代码格式调整，不影响逻辑
test      测试相关
chore     构建、配置、依赖、脚本等杂项
perf      性能优化
revert    回退提交
release   版本发布
```

### 3. 提交日志示例

```text
feat: 添加 UI 元素查询接口
feat: 添加截图接口
fix: 修复点击坐标偏移问题
fix: 修复 rootless 环境安装失败问题
docs: 更新 README 安装说明
refactor: 重构 AXRuntime 元素解析逻辑
chore: 调整 Makefile 打包配置
perf: 优化截图响应速度
release: 发布 v1.1.0
```

### 4. 不推荐的提交日志

```text
update
fix bug
提交代码
修改了一些东西
优化
修复
```

---

## 五、提交代码到仓库工作流

### 1. 触发方式

当用户输入以下任意内容时，进入“提交代码到仓库”流程：

```text
提交代码
提交代码到仓库
提交并推送
git 提交代码到仓库
生成提交日志
```

### 2. 执行流程

#### 第一步：检查当前状态

必须先执行：

```bash
git status
git diff --stat
git diff
```

如果有未跟踪文件，也需要在总结中说明。

#### 第二步：总结当前改动

根据当前未提交代码，总结本次改动。

输出格式：

```text
本次改动摘要：

1. xxx
2. xxx
3. xxx
```

#### 第三步：生成提交日志

根据代码改动自动生成一条符合规范的提交日志。

例如：

```text
fix: 修复点击坐标偏移问题
```

如果本次改动包含多个方向，优先判断主要目的。

判断规则：

```text
新增功能       -> feat
修复 bug       -> fix
文档修改       -> docs
代码重构       -> refactor
构建配置调整   -> chore
性能优化       -> perf
版本发布       -> release
```

#### 第四步：提交前必须让用户确认

在执行任何提交命令之前，必须展示以下内容：

```text
本次改动摘要：

1. xxx
2. xxx
3. xxx

建议提交日志：

fix: 修复点击坐标偏移问题

准备执行：

git add .
git commit -m "fix: 修复点击坐标偏移问题"
git push

是否确认提交并推送？
```

只有用户明确回复确认后，才能继续执行。

#### 第五步：用户确认后提交并推送

用户确认后执行：

```bash
git add .
git commit -m "用户确认后的提交日志"
git push
```

如果当前分支没有关联远程分支，则执行：

```bash
git push -u origin 当前分支名
```

---

## 六、版本发布工作流

### 1. 触发方式

当用户输入以下内容时，进入“版本发布”流程：

```text
发布 1.1.0
发布 v1.1.0
发布版本 1.1.0
release 1.1.0
创建 release 1.1.0
```

### 2. 版本识别规则

如果用户输入：

```text
发布 1.1.0
```

则识别为：

```text
版本号：1.1.0
release 分支：release-1.1.0
tag：v1.1.0
GitHub Release 标题：项目名 v1.1.0
```

如果项目名称可以从仓库名或 README 中明确识别，则使用项目名生成 Release 标题。

例如：

```text
iOS MCP v1.1.0
```

如果项目名称不明确，则在创建 Release 前让用户确认标题。

### 3. release 分支规则

release 分支统一使用：

```text
release-版本号
```

示例：

```text
release-1.0.0
release-1.0.1
release-1.1.0
```

不要使用：

```text
release/v1.1.0
release-v1.1.0
release_1.1.0
```

### 4. tag 规则

tag 统一使用：

```text
v版本号
```

示例：

```text
v1.0.0
v1.0.1
v1.1.0
```

### 5. GitHub Release 标题规则

GitHub Release 标题使用：

```text
项目名 v版本号
```

例如：

```text
iOS MCP v1.1.0
```

---

## 七、发布版本执行流程

### 第一步：检查仓库状态

发布前必须执行：

```bash
git status
git branch --show-current
git fetch --all --tags
```

如果存在未提交代码，必须停止发布流程，并提示用户先提交或处理当前改动。

如果 `git fetch --all --tags` 因网络、DNS、GitHub 连接、认证或沙箱权限失败，必须按以下规则处理：

1. 可以重试一次相同命令。
2. 如果是沙箱网络或 `.git` 写入权限导致失败，可以用提权方式重试。
3. 如果重试后仍失败，必须停止发布流程。
4. 停止时要明确说明当前本地状态、失败命令、失败原因，以及恢复发布时应继续执行的下一条命令。
5. 远端同步没有成功前，不允许继续创建 tag、推送 tag、创建 GitHub Release。

### 第二步：确认目标版本信息

根据用户输入生成版本信息，并展示给用户：

```text
准备发布版本：1.1.0

release 分支：
release-1.1.0

tag：
v1.1.0

GitHub Release 标题：
iOS MCP v1.1.0
```

### 第三步：检查 tag 是否已存在

执行：

```bash
git tag --list "v1.1.0"
```

如果 tag 已存在，必须停止发布流程，并提示用户该版本已经存在，不能重复发布。

### 第四步：创建或切换 release 分支

先切换到 main 并拉取最新代码：

```bash
git switch main
git pull
```

检查 release 分支是否存在：

```bash
git branch --list "release-1.1.0"
git branch -r | grep "origin/release-1.1.0"
```

如果本地和远程都不存在，则从 main 创建 release 分支：

```bash
git switch -c release-1.1.0
```

如果远程已经存在，则切换到已有 release 分支：

```bash
git switch release-1.1.0
git pull
```

### 第五步：检查并更新版本号


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [witchan/ios-mcp](https://github.com/witchan/ios-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
