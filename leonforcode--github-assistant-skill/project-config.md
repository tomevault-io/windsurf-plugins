---
trigger: always_on
description: GitHub Trending查看、项目搜索、Star/Fork/Watch操作、Issues管理、Pull Request操作、代码内容获取、评论管理等完整助手。当用户需要查看GitHub Trending排行榜、搜索GitHub项目、对仓库进行操作、管理Issues/PR、获取代码内容时使用此技能。
---


# GitHub Assistant

> **🌐 默认输出语言：中文**
> 除非用户明确要求使用其他语言，所有回复、Trending 展示、搜索结果、操作反馈均使用中文。
> 项目描述、贡献者用户名、仓库名等原始英文内容保持不变。

通过 GitHub REST API + Playwright 浏览器双模式，提供 GitHub Trending、搜索、仓库操作、Issues/PR管理、代码内容获取等完整能力。

## 能力概览

1. **Trending 查询** — 查看每日/每周/每月热门项目
2. **项目搜索** — 搜索仓库、查看详情
3. **仓库操作** — Star/Unstar/Fork/Watch/Unwatch、创建仓库、列出Forks
4. **Issues 管理** — 创建、列出、关闭、重新打开、评论、标签管理、锁定
5. **Pull Requests** — 创建、列出、关闭、合并、审查、评论、请求审查者
6. **代码内容** — 获取文件、目录、README、创建/更新文件
7. **分支管理** — 列出、创建、删除分支
8. **Releases 管理** — 列出、创建、更新、删除 Release
9. **Actions 操作** — 列出工作流、触发/取消/重新运行工作流
10. **用户操作** — 获取用户信息、列出仓库、关注/取消关注
11. **通知管理** — 列出通知、标记已读
12. **组织操作** — 列出组织、获取组织信息、列出成员
13. **评论管理** — Issue/PR 评论
14. **账户管理** — 登录/登出/状态检查

---

## Onboarding（首次使用必读）

### 环境准备

```bash
cd <skill-dir>/scripts
pip install -r requirements.txt
```

**安装浏览器引擎（仅浏览器模式需要）**

#### 方式1：智能安装脚本（推荐）

```bash
python3 install_browser.py
```

**脚本功能：**
- ✅ 自动检测前10秒下载进度
- ✅ 无进度时提示选择国内镜像源
- ✅ 自动验证浏览器安装状态
- ✅ 支持淘宝/清华/华为云镜像

#### 方式2：默认安装

```bash
python3 -m playwright install chromium
```

#### 方式3：使用国内镜像源（下载失败时使用）

Playwright 支持通过环境变量 `PLAYWRIGHT_DOWNLOAD_HOST` 设置浏览器下载镜像源。

**Mac/Linux:**
```bash
# 使用淘宝镜像（推荐）
PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright python3 -m playwright install chromium

# 或使用清华大学镜像
PLAYWRIGHT_DOWNLOAD_HOST=https://mirrors.tuna.tsinghua.edu.cn/playwright python3 -m playwright install chromium
```

**Windows (PowerShell):**
```powershell
$env:PLAYWRIGHT_DOWNLOAD_HOST="https://npmmirror.com/mirrors/playwright"
python -m playwright install chromium
```

**Windows (CMD):**
```cmd
set PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright
python -m playwright install chromium
```

**常用镜像源：**
- 淘宝：`https://npmmirror.com/mirrors/playwright`
- 清华：`https://mirrors.tuna.tsinghua.edu.cn/playwright`
- 华为云：`https://mirrors.huaweicloud.com/playwright`

### 登录 GitHub（二选一）

#### 方式A：浏览器手动登录（推荐）

告诉用户："我来帮你打开浏览器登录 GitHub，请在弹出的窗口中手动输入账号密码完成登录。"

执行步骤：
1. 运行 `python3 github_login.py browser`
2. 等待用户在浏览器中完成登录
3. 登录成功后会自动保存 session

> 注意：浏览器模式仅用于 Trending 页面抓取。API操作（Star/Fork/Watch）仍需 Token。

#### 方式B：Token 登录（支持全部操作）

支持两种 Token 类型：**Fine-grained PAT**（推荐）或 **Classic PAT**

---

**选项 1：Fine-grained Personal Access Token（推荐）**

更安全的权限控制，可为每个仓库单独配置权限。

1. **打开设置页面**
   - 访问 https://github.com/settings/tokens?type=beta
   - 或 GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens

2. **生成新 Token**
   - 点击 "Generate new token"
   - 输入 Token 名称（如：GitHub-Assistant-Skill）
   - 设置过期时间（建议 90 天或更长）
   - 描述：用于 GitHub Assistant Skill 的完整操作

3. **配置权限（重要！）**

   **Repository permissions（仓库权限）- 全部设置为 Read and Write：**
   
   | 权限 | 用途 |
   |------|------|
   | **Actions** | 读取工作流运行状态 |
   | **Administration** | 读取仓库管理信息 |
   | **Checks** | 读取检查运行状态 |
   | **Commit statuses** | 读取提交状态 |
   | **Contents** | 读取/创建/更新/删除文件 |
   | **Dependabot alerts** | 读取安全警报 |
   | **Deployments** | 读取部署信息 |
   | **Discussions** | 读取/创建讨论（如需要） |
   | **Environments** | 读取环境信息 |
   | **Issues** | 读取/创建/更新/关闭 Issues |
   | **Metadata** | 读取基础仓库信息（必需）|
   | **Packages** | 读取包信息 |
   | **Pages** | 读取 GitHub Pages 信息 |
   | **Pull requests** | 读取/合并/审查 PR |
   | **Repository security advisories** | 读取安全公告 |
   | **Secret scanning alerts** | 读取密钥扫描警报 |
   | **Workflows** | 读取工作流 |

   **Account permissions（账户权限）：**
   - 保持默认即可

4. **选择仓库访问范围**
   - **All repositories** - 访问所有仓库（推荐）
   - **Only select repositories** - 仅访问指定仓库

5. **生成并保存 Token**
   - 点击 "Generate token"
   - **⚠️ 立即复制 Token**（页面关闭后无法再次查看）
   - 运行 `python3 github_login.py token <TOKEN>`
   - 验证成功后 Token 会被安全存储到本地

---

**选项 2：Classic Personal Access Token**

传统的 Token 类型，使用 scopes 权限模型。

1. **打开设置页面**
   - 访问 https://github.com/settings/tokens
   - 或 GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)

2. **生成新 Token**
   - 点击 "Generate new token (classic)"
   - 输入 Token 名称（如：GitHub-Assistant-Skill）
   - 设置过期时间（建议 90 天或更长）

3. **选择 Scopes（权限范围）**

   **必需 Scopes：**
   
   | Scope | 用途 |
   |-------|------|
   | `repo` | 完全控制私有仓库（包含所有仓库操作）|
   | `public_repo` | 访问公共仓库 |
   
   **推荐 Scopes（根据需要使用）：**
   
   | Scope | 用途 |
   |-------|------|
   | `workflow` | 更新 GitHub Actions 工作流文件 |
   | `read:org` | 读取组织成员信息 |
   | `read:user` | 读取用户资料信息 |
   | `read:discussion` | 读取团队讨论 |

4. **生成并保存 Token**
   - 点击 "Generate token"
   - **⚠️ 立即复制 Token**（页面关闭后无法再次查看）
   - 运行 `python3 github_login.py token <TOKEN>`
   - 验证成功后 Token 会被安全存储到本地

---

**Token 类型对比：**

| 特性 | Fine-grained PAT | Classic PAT |
|------|-----------------|-------------|
| 权限粒度 | 按仓库、按权限细粒度控制 | 全局 scopes |
| 安全性 | 更高，可限制访问范围 | 较低，全局权限 |
| 组织仓库 | 需要组织管理员批准 | 直接可用 |
| 推荐度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 适用场景 | 生产环境、团队协作 | 个人使用、快速测试 |

### 验证登录状态

运行 `python3 github_login.py check` 查看当前登录状态。

---

## 功能指令映射

### 1. 查看 Trending（核心功能）

**触发词**: trending、热门、排行榜、最火

| 参数 | 值 |
|------|-----|
| since | daily（默认）/ weekly / monthly |
| language | python / javascript / go / rust 等（可选）|

**Agent 执行流程**：

> ⚠️ **必须使用浏览器模式** 以获取完整数据（含贡献者和期间Star数）。
> API 模式仅作为 fallback，不包含 `built_by` 和 `period_stars`。

```bash
# 浏览器模式（推荐，自动滚动获取所有项目）
python3 github_trending.py daily "" browser       # 今日所有Trending项目

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonforcode/GitHub-Assistant-Skill](https://github.com/leonforcode/GitHub-Assistant-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
