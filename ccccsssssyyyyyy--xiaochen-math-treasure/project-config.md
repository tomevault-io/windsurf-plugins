---
trigger: always_on
description: 面向非技术用户的安装入口是仓库根目录的 [`一键安装指令.md`](一键安装指令.md)——该文件就是设计给用户整段复制、转发给 AI 助手执行的。当用户给出仓库地址并要求安装时按它执行，要点：
---

# AGENTS.md - 本地化数学题库管理系统 开发与 AI 代理指南

## 0. 用户说「帮我装好并启动」时怎么做

面向非技术用户的安装入口是仓库根目录的 [`一键安装指令.md`](一键安装指令.md)——该文件就是设计给用户整段复制、转发给 AI 助手执行的。当用户给出仓库地址并要求安装时按它执行，要点：

- **下载**：优先用稳定地址 `https://github.com/ccccsssssyyyyyy/xiaochen-math-treasure/releases/latest/download/<asset>`；国内直连 GitHub 经常失败（实测返回 502），失败即前置 `https://gh-proxy.com/` 重试，并告知用户最终用了哪个地址。Release 资源名为 `xiaochen-math-treasure-Windows-x64.zip` / `xiaochen-math-treasure-macOS-AppleSilicon.zip` / `xiaochen-math-treasure-macOS-Intel.zip`（2.2.3 及更早为 `MathBank-*.zip`，两套名字的程序内解析都兼容）。若直链返回 404，改从 `releases/latest` 页面挑包并告知用户所选的包名。
- **选对 macOS 包**：2.2.1 起 macOS 按芯片分包，必须先判断芯片再下载。`uname -m` 输出 `arm64` 用 AppleSilicon 包，`x86_64` 用 Intel 包；Rosetta 下 `uname -m` 会误报 `x86_64`，以 `sysctl -n hw.optional.arm64` 是否为 `1` 为准。下错包启动器会直接提示该换哪个，不会静默失败。
- **不再需要 Python 与 pip**：三个便携包都内置 Python 运行时，解压双击即跑。**不要**再执行 `pip install`、`python -m venv` 或要求用户安装 Python；如果脚本报「内置运行时无法运行」，那是包与机器不匹配或解压不完整，按提示换包或重新解压。
- **启动必须脱离 agent 进程树**：macOS 用 `open "启动题库系统.command"`，Windows 用 `start "" "启动题库系统.bat"`。**禁止**在前台运行 `uvicorn`——会话结束后进程会被回收，用户只会看到「网页打不开」。
- **失败处理**：把完整报错原文交给用户，禁止静默重试、禁止跳过失败步骤继续。
- **macOS 先解除下载隔离（2.2.4 起为必备步骤）**：从浏览器下载的 zip 带 `com.apple.quarantine`，会连带阻止包内自带 Python 被执行，表现为「Apple 无法验证」或双击无反应。解压后先执行 `xattr -dr com.apple.quarantine "<解压目录>"`；`xattr` 不可用或报 Operation not permitted 时，改为引导用户在访达里 Control+点「启动题库系统.command」→「打开」→「打开」。启动器自身也会尝试一次性自愈，但**第一次双击之前它还没机会运行**。
- **禁止**代替用户申请或填写 API Key，只负责引导到网页「设置 → API 配置」。用户若问「不配 Key 能不能先用」，告诉他题库为空时顶部的「首次启动引导」卡片可一键载入 8 道内置示例题，无需 Key 即可体验组卷与导出源码。

修改下载地址、启动方式或安装步骤时，必须同步更新 `一键安装指令.md`、README「快速开始」与本节，三处不得出现不一致说法。

## 1. 项目概述
本项目是一个本地运行的半自动化数学题库管理工作台。核心目标是通过极简的本地化部署，实现高质量图文混排数学题目（尤其是高中及更高阶数学内容）的收集、标签化管理、OCR 识别以及 AI 辅助生成解析。

## 2. 核心技术栈
本项目追求极简配置与极致体验，严格遵循以下技术选型，**不要引入复杂的现代前端构建工具（如 Webpack/Vite/Node.js 生态）**：
- **后端**：Python + FastAPI。
- **后端渐进式模块架构**：根目录 `main.py` 继续作为 `uvicorn main:app` 兼容入口；后端领域能力统一集中在 `mathbank/`。`database.py` 提供 SQLite ORM 与 Session，`db_migrations.py` 提供版本化、备份优先的数据库迁移，`backup.py` 提供带清单校验的完整备份与恢复，`asset_security.py` 统一校验上传内容与本地资产路径，`task_manager.py` 提供有界异步任务、协作取消与临时资源生命周期，`health.py` 提供启动就绪诊断，`paper_helper.py` 提供 LaTeX/PDF 编译排版，`sync_helper.py` 只负责 JSON 同步导出与 AI 题库导出，`paths.py` 统一锚定持久化与捆绑路径，`page_block_split.py` 提供扫描页离线切块（行投影 + 分栏判定），`mistake_handout.py` 提供错题本 LaTeX 模板与编译入口，`mistake_vocabulary.py` 负责错因词表加载与归一，`duplicate_check.py` 是入库查重的唯一权威实现（阈值与预筛只允许在这一处定义）。`curriculums.py` 加载四套数学教材 JSON 与物化只读大纲，`prompts.py` 提供纯提示构建器，`ai_providers.py`、`ai_http.py`、`ai_json.py` 分别统一模型供应商解析、AI HTTP 请求与结构化输出解析。运维、迁移、检索与 Release 工具统一位于 `scripts/`，从项目根目录使用 `python3 -m scripts.<模块名>` 运行。严禁重新在根目录新增业务模块或复制供应商判断规则。
- **数据库**：SQLite + SQLAlchemy（轻量级，数据存储在本地 `.db` 文件中）。
- **前端页面**：纯 HTML + 原生 JavaScript。
- **前端脚本拆分**：前端 JS 采用无编译的“渐进式级联加载”架构，按 `math-render.js`、`api.js`、`editor.js`、`ocr.js`、`import.js`、`paper.js`、`mistake.js`、`onboarding.js`、`backup.js` 的顺序级联加载，**以 `static/index.html` 末尾的 `<script>` 顺序为唯一事实来源**。职责：`math-render.js` 是全站唯一 KaTeX 渲染入口（`delimiters` 配置只允许在此定义，禁止其它模块重复内联），`api.js` 负责 API/全局状态，`editor.js` 编辑与渲染，`ocr.js` OCR 图像交互，`import.js` 导入拆卷，`paper.js` 组卷工作台，`mistake.js` 错题工作台，`onboarding.js` 首次启动引导、依赖引导与升级向导，`backup.js` 设置页的备份与还原（函数显式挂 `window`，供 HTML onclick 调用）。加载顺序严格依存，不允许产生任何编译及捆绑动作。
- **前端样式与字体**：Tailwind CSS + FontAwesome 图标库 + Inter/Outfit 字体包（均已下载至本地 `/static/lib` 支持 100% 离线使用与跨平台系统降级）。
- **公式渲染**：KaTeX（已下载至本地支持 100% 离线数学公式渲染），必须支持题干与解析框实时解析、秒级渲染。
- **中转站模型 7:3 弹性 UI 布局与 Reasoning Effort 自动解析**：在系统 API 设置中选择中转站平台（`zhongzhan_gpt` 或 `zhongzhan_claude`）时，模型输入区域自动转换为 7:3 弹性比例（70% 模型名称，30% 推理强度）。后端由 `mathbank.ai_providers.parse_model_and_effort` 自动提取纯净模型名称并注入请求参数。
- **全局 Tooltip 提示系统**：基于纯原生事件代理接管 `title` / `data-tooltip` 浮现（详见第 6 节交互规范）。


> [!IMPORTANT]
> **开发规范单一来源规则**：
> 根目录 `AGENTS.md` 是本项目面向 AI 代理与开发者的唯一开发规范来源。进行系统更新、重构、功能新增或回滚（Rollback）时，如变更影响本文记录的技术设计、接口规范、验证方式或发布流程，必须同步更新本文件，确保规范与实际代码实现准确一致；禁止再维护内容重复的平行代理指南。

> [!IMPORTANT]
> **本地版本号与发布权限边界**：
> 全局系统版本号统一定义于 `mathbank/__init__.py` 的 `__version__`。AI 代理只按用户要求修改该本地版本号；仅在用户明确要求本地打包时运行 `scripts/build_release.py`，产物只保存在本地。Git Tag、GitHub Release、Release 草稿与附件上传全部由用户自行管理，AI 代理不得创建、移动、删除或推送 Tag，不得创建、编辑、发布或上传 GitHub Release。用户要求“同步到 GitHub”时，默认仅同步当前代码分支，不包含任何 Tag 或 Release 操作。

> [!IMPORTANT]
> **项目路径单一来源规则**：
> 所有持久化文件和捆绑资源路径必须从 `mathbank.paths` 获取，并以 `PROJECT_ROOT` 为锚点。禁止新增依赖当前工作目录的 `./data_backup`、`os.getcwd()` 或“脚本所在目录就是数据库目录”等隐式假设，确保从任意工作目录启动服务或 CLI 都访问同一份数据。

> [!WARNING]
> **JavaScript 语法防错与浏览器兼容性警示**：前端的五个脚本文件在浏览器中级联加载。任何人在修改 JS 代码时，必须遵循以下规则：
> 1. **确保无任何语法错误**：语法错误会导致浏览器停止解析后续脚本，挂起 `DOMContentLoaded` 事件，使界面死锁。修改后建议运行 `node -c static/js/*.js` 校验。
> 2. **禁用正则后行断言**：前端代码中**严禁使用正则后行断言 `(?<!...)` 和 `(?<=...)`**，此类语法在旧版浏览器（如 Safari < 16.4）或移动端 WebView 中会触发致命的 `SyntaxError` 中断加载。必须改用捕获组或字符串拆分逻辑。
> 3. **版本号缓存击穿 (Cache Busting)**：后端在首页路由 `read_index` 中根据前端 JS/CSS/Favicon 修改时间戳自动追加版本号后缀（如 `?v=时间戳`）。开发者无需手动修改 HTML 中的版本号。
> 4. **Tailwind 自定义色彩与透明度定义**：CSS 变量含逗号分隔符时（如 `124, 58, 237`），**严禁使用 `rgb(var(--brand-xxx-rgb) / <alpha-value>)`**。必须使用 `rgba(var(--brand-xxx-rgb), <alpha-value>)` 格式以保证所有浏览器的解析兼容。

## 3. 核心业务逻辑与模块设计
### 3.1 题目收集与存储
- **题干录入**：支持多行纯文本与 LaTeX 代码混合输入，界面配备实时 KaTeX 渲染预览区。
- **插图管理**：提供图片上传与 TikZ 绘图代码输入。图片保存在本地文件系统（`static/uploads/`），数据库存储相对路径。
- **插图排版位置联动与多图复合渲染**：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ccccsssssyyyyyy/xiaochen-math-treasure](https://github.com/ccccsssssyyyyyy/xiaochen-math-treasure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
