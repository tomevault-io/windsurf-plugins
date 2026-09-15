---
trigger: always_on
description: **对本项目任何文件做任何修改之前，必须先向用户说明要改什么、怎么改、为什么改，等用户明确同意后再动手。** 包括但不限于：编辑 SKILL.md、修改网页、调整协议流程、删除内容、重构章节。禁止"先改再说"。讨论和分析不需要确认，但一碰文件就必须停手等用户点头。
---

# Debate-Coach Project

## ⛔ 最高优先级：修改前必须先确认
**对本项目任何文件做任何修改之前，必须先向用户说明要改什么、怎么改、为什么改，等用户明确同意后再动手。** 包括但不限于：编辑 SKILL.md、修改网页、调整协议流程、删除内容、重构章节。禁止"先改再说"。讨论和分析不需要确认，但一碰文件就必须停手等用户点头。

## 🏗️ 开发工作区架构（最高优先级）
**本项目（`Debate-Coach-Backup/`）是唯一开发工作区。所有日常开发、修改、测试、构建在此进行。**

```
Debate-Coach-Backup/          Debate-Coach/
  (开发工作区 ← 当前)           (纯净发布仓库)
  日常所有工作在此              只在稳定版本时更新
                               更新后 commit + push GitHub
```

- **禁止在 `C:/Claude/Project/Debate-Coach/` 修改任何文件** — 那边是纯净发布仓库
- 稳定版本发布流程：Backup 完成 → 打包 handoff → 切换到 Debate-Coach 会话 → 读取 handoff → 更新那边文件 → commit + push
- 对比本地与 GitHub 一致性时，只对比 `Debate-Coach/`（git 仓库）↔ GitHub

## 🗂️ 文件关系速查（新会话先读 · 开发基准）

**网页版开发基准**（v9.2.1+：编辑对应真源 → 通过共享 `standalone-pages` / 专用同步脚本写入 `DC_PAGES` → 浏览器回归 → 发布）：

| 基准文件 | 身份 | 对齐状态 |
|---|---|---|
| `debate-coach-web.html` | **正式单文件网页母版 / GH 发布基准**。v9.2.1+ 使用 `DC_PAGES` 可读字符串内嵌 ZH/EN/JUDGE/TOOLBOX/LANG/JUDGE_ENTRY/EXAM 七个完整 HTML；Toolbox 内以 `TOOL_HTML` 内嵌 9 工具×中英，运行时由同源 `iframe.srcdoc` 加载。禁止恢复根部 Base64 + `atob/TextDecoder/document.write` 旧加载链。 | 当前发布线 = v9.2.1 |
| `toolbox.html`（独立壳）+ 13 工具页 | **线上 InfinityFree 专用外置形态**（1MB 限制）；工具页唯一来源 = Backup 根目录（manifest 指纹校验） | ⚠️ 仅线上使用，GH 已撤除 |
| `Output/裁判所2.0.html` | **裁判所青春版单页母版**（发布时同步到 `DC_PAGES.JUDGE`） | 独立真源，发布前由 verify 检查 |
| `Skill-Web.md` | 教练网页（`DC_PAGES.ZH`）知识库源（已含 8 候选重构） | 发布前由 verify 检查 |
| `Skill-Judge.md` | 裁判页对应独立 skill（v9.0.0-Final-B 两轮提示词） | ⚠️ 未与 GH 裁判页对齐，勿当裁判页源 |

**单文件交付铁原则（用户规定）**：GH/下载/本地/APK 必须一文件全功能——正式母版 `DC_PAGES.TOOLBOX` 内的 `TOOL_HTML` 内嵌全部工具（iframe/srcdoc 隔离），不得依赖外部工具页文件。InfinityFree 在线是唯一 1MB 限制例外（保持外置壳+独立页）。

**Agent 单文件交付 Skill 开发基准**：
- `SKILL.md`（合一版，当前 178KB）：Coach 全套知识库 + 阶段 C 青春版裁判所 C1-C11 协议，**Agent 端单文件交付 Skill 的开发基准文件**。四副本（Backup 根 / `Debate-Coach/SKILL.md` / `.claude/skills/debate-coach/` / `Test/.claude/skills/debate-coach/`）哈希一致 = `9b846074`（2026-08-15 术语统一后同步）。旧版 Test 安装副本（195KB，`58334411`，7/21）已归档至 `Output/归档-SKILL-260816/`。
- ⚠️ `Debate-Judge/SKILL.md` 是 Backup 根旧教练基座（无 C 阶段移植）的残留拷贝，勿混用。

## ⛔ 最高优先级：禁止推送 Git
**未经用户明确同意，严禁执行 `git push`、`git commit`、`git tag` 或任何修改 Git 历史的操作。** 只允许只读命令（`git log`、`git diff`、`git status`、`git remote -v` 等）。违反此规则将导致项目不可逆损坏。

## ⛔ 最高优先级：唯一路径与网络纪律（禁止新建任何工作区域）
**本地 Git 工作路径只有 1 个：`C:\Claude\Project\Debate-Coach\`**（唯一 git 仓库，remote=MoonTzai/debate-coach）。**开发工作区只有 1 个：本项目（`Debate-Coach-Backup/`）。**
- 禁止 `git init`、`git clone`、`git worktree`、新建任何其他 git 仓库/工作区/克隆目录
- 禁止新建任何平行工程（APK 唯一工程为 `APK/`）
- 历史遗留仓库（Debate-Grill*/worktree/web-check）已归档至 `Output/归档-Git-260815/`，不得恢复为工作区
- **推送/联网前先跑 `node scripts/proxy.cjs --set`**（自动探测可用代理端口 → 写入 git 全局 → 清除发布仓库 local 残留；`--check` 只探测不修改）
- **网络故障协议**：网络失败时**禁止新建目录、克隆、备用工作区、新仓库、换路径**。固定流程：`node scripts/proxy.cjs --check` 检查代理 → 原路径重试 → 仍失败则停下向用户报告，等待指示
- 与 GH 对比时：**先 `git fetch origin` 再用 `origin/main`**（本地跟踪引用会过期，直接看本地 refs 会误判远程状态——2026-08 已因此误判过一次）

## 🔒 最高优先级：受保护目录禁止修改
**`Output/milestone-*-protected/` 及所有里程碑目录中的文件禁止任何修改、删除、覆盖。** 只允许读取、复制到新位置、打开查看。修改保护文件需要用户明确说出"授权修改保护目录"。文件系统已设只读（chmod 444）。

## 🚫 最高优先级：禁止 Python 脚本修改代码
**禁止用任何 Python 脚本（patch_gfl.py、rebuild_en.py、build_zones.py 等）修改 JS/HTML。** 转义层级不可控，已导致循环坏档和 API Token 浪费。唯一安全方式：Edit 工具手改母版 + 浏览器验证 + node 做纯 base64 编码。

## ⛔ 最高优先级：禁止推送到 GitHub 不存在的文件
**同步到 Debate-Coach 发布仓库时，只更新 GitHub 已存在的文件。** GitHub 已明确删除的文件（TERMINOLOGY.md、debate-coach-web-zh.html、debate-coach-web-en.html）禁止重新推送。新增文件需用户逐次明确授权后才能加入 Git 追踪。判断标准：`git ls-tree -r --name-only HEAD` 的输出 = 可更新白名单。

## 📦 APK 打包（唯一工程 `APK/`，唯一命令 `scripts/package.cjs`）
**唯一工程**：`APK/`（`android/` gradle 工程 + `www/` web 资产 + `capacitor.config.json`）。所有历史残留（根 `android/`、根 `www/`、`APK/app/`、旧安装包等）已归档至 `Output/归档-APK-260815/`，**禁止重建任何平行工程**。

**JDK 位置（不在 Program Files，在用户目录！）：**
- JDK 21：`C:/Users/Moon/Java/jdk-21.0.11+10`（capacitor 8.x 需要 21）
- Android SDK：`C:/Users/Moon/AppData/Local/Android/Sdk`

**唯一命令（全部内含断言，禁止手工删拷）：**
```bash
node scripts/package.cjs --copy-only   # 同步 master → APK/www + 字节/tag 断言（日常同步）
node scripts/package.cjs --gradle      # 同步 + cap copy → gradle 构建 → APK 内提取复核
```
内部流程：先删后拷破 gradle 文件锁 → 字节/tag 断言 → `npx cap copy`（写 `APK/android/app/src/main/assets/public/`，**勿用 cap sync**——会重置 versionCode）→ `gradlew assembleDebug --rerun-tasks`（JAVA_HOME=JDK21 自动设置，破增量缓存）→ 从 APK 提取 `assets/public/index.html` 与 master 逐字节比对。
产物：`APK/android/app/build/outputs/apk/debug/app-debug.apk` → 复制为根目录 `Debate-Coach-APK-v8.0.7.apk`（发布基准）。

**禁止**：不要在 `C:\Program Files` 下找 JDK；不要手工 `rm/cp` APK/www 或 assets（gradle 守护进程文件锁会静默失败返回0）；不要 `npx cap sync`（会重置原生工程 versionCode）；不要在根目录重建 capacitor 工程。

## 🛠️ 发布与检视模块（scripts/ —— 发布前必跑 verify）

**规则：发布/检视一律用下方 node 模块，禁止再写一次性 Python 脚本**（旧 dump_*/analyze_*/patch_*/push_*/test_* 等 79 个已归档至 `Output/归档-脚本-260815/`，根目录仅保留 build_inf_v805.py / extract_docx.py / count_docx.py）。

| 模块 | 用法 | 职责 |
|---|---|---|
| `node scripts/verify.cjs` / `node scripts/verify.cjs --release` | 两级验证门 | 默认=本地预检：严格校验 `DC_PAGES` 7 页面、旧根 B64/loader 为 0、Skill-Web/ZH、C 协议、字典、工具 manifest 等；尚未同步的发布镜像仅 WARN。`--release`=发布门：APK、htdocs、纯净发布仓库等下游镜像也必须一致，否则 FAIL。共享解析统一走 `scripts/standalone-pages.cjs`。 |
| `node scripts/inspect.cjs` | `节点名 [行区间]` 或 `节点名 --grep <正则>` | 检视解码页，替代旧 dump_*.py。`--list` 列块 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonTzai/debate-coach](https://github.com/MoonTzai/debate-coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
