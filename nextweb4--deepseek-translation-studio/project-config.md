---
trigger: always_on
description: - 项目根目录为 `deepseek_translation_studio/`。
---

# AGENTS.md

## 1. 项目结构
- 项目根目录为 `deepseek_translation_studio/`。
- 入口文件是 `main.py`，应用组装在 `src/app.py`。
- 通用对话捕获剥离版入口是 `conversation_main.py`，窗口在 `src/ui/conversation_window.py`。
- UI 位于 `src/ui/`，业务服务位于 `src/services/`，引擎位于 `src/engines/`，数据库访问位于 `src/database/`，数据模型位于 `src/models/`。
- 默认 Prompt 模板位于 `src/resources/default_prompts.json`。

## 2. 运行命令
- 安装依赖后使用 `python main.py` 启动桌面应用。
- 使用 `python conversation_main.py` 启动剥离版通用对话捕获器。
- API 模式通过环境变量 `DEEPSEEK_API_KEY`、`DEEPSEEK_API_BASE`、`DEEPSEEK_MODEL` 配置。
- Web 模式默认打开 `https://chat.deepseek.com/`，可用 `DEEPSEEK_WEB_URL` 覆盖。
- 剥离版默认数据目录使用 `%LOCALAPPDATA%\DeepSeekConversationCapture`，可用 `DCC_APP_DATA_DIR` 覆盖；不得默认复用主程序的 `DST_APP_DATA_DIR`。

## 3. 测试命令
- 使用 `pytest` 运行单元测试。
- 修改 Prompt 拼接、SQLite repository、Web 捕获版本选择、导出逻辑后必须运行 `pytest`。

## 4. 构建命令
- 使用 `python scripts\build_exe.py` 构建 Windows exe。
- 使用 `python scripts\build_conversation_exe.py` 构建剥离版 `DeepSeek Conversation Capture`。
- 使用 `powershell -NoProfile -ExecutionPolicy Bypass -File scripts\publish_github_release.ps1` 发布到 GitHub；该脚本必须只从 `GITHUB_TOKEN`、`GH_TOKEN`、Git Credential Manager 或 Codex GitHub 集成 helper 临时读取 token，不得把 token 写入文件、Git remote 或日志。
- 构建产物必须位于 `dist\DeepSeek Translation Studio\DeepSeek Translation Studio.exe`。
- 剥离版构建产物必须位于 `dist\DeepSeek Conversation Capture\DeepSeek Conversation Capture.exe`。
- Web 模式打包产物必须同时包含 `_internal\playwright\driver\package\.local-browsers\chromium-*`，否则 exe 可以打开但无法启动 Playwright 浏览器。
- 修改打包入口、资源文件路径或 Playwright 引入方式后，必须重新构建并确认 exe 文件存在。
- 修改 `src/database/schema.sql` 后必须确认 `src/database/db.py` 中存在旧库迁移逻辑。

## 5. 代码风格
- 当前未发现 lint / format 命令。
- 核心类和跨模块函数必须保留类型注解。
- 新增功能优先使用标准库；引入新依赖前必须更新 README、requirements、pyproject，并说明许可证。
- 当前新增打包依赖为 PyInstaller；仅用于生成 Windows exe，不得把它引入业务运行路径。
- MSI 使用 WiX Toolset 3.14 binaries 作为构建期工具，下载到 `.build_tools`；不得把 WiX 二进制提交为源码或运行时依赖。
- PySide6 UI 文案可以使用中文；业务层异常信息应清晰可展示给用户。

## 6. 模块边界
- `src/ui/` 只做界面、信号和状态展示，不直接拼 SQL，不直接操作 Playwright 页面。
- `src/services/translation_service.py` 负责翻译任务编排和历史保存。
- `src/services/capture_service.py` 负责异常提示检测、版本去重和最长有效文本选择。
- `src/models/translation_task.py` 的 `task_mode` 决定任务语义；通用对话捕获必须使用 `TASK_MODE_GENERAL_CAPTURE`，不得依赖源文本是否像 SRT 来误启用 SRT 清理。
- `src/ui/conversation_window.py` 是剥离版 UI，只能创建 `ENGINE_WEB` + `TASK_MODE_GENERAL_CAPTURE` 请求；不得加入 API 设置、SRT 导出、任务类型下拉框或字幕补译入口。
- `src/services/browser_profile_service.py` 负责 Web 账号切换时的浏览器 profile 轮换；只能移动 `settings.app_data_dir` 内的 profile，不能删除或移动任意路径。
- `src/services/subtitle_translation.py` 只负责 SRT 检测和模型输出中的 SRT 代码块/网页控件文本清理，不自动改写 Prompt 为 JSON。
- `src/services/subtitle_translation.py` 可以做 SRT 完整性检查、剩余片段渲染和多次输出拼接；续译时必须保留原编号和时间轴，不得把未翻译的源字幕当作完成译文。
- SRT 合并和完成判定必须拒绝与源字幕正文相同的无效候选译文；但长度不超过 4、且不含拉丁/日文/韩文字符的中文数字、短中文、标点或符号类文本可以保留，因为它们翻译后可能天然不变。
- 短符号、音乐符号和标点类字幕（例如 `♪`、`...`）如果翻译后与源文本一致，可以作为有效译文保留；英文、日文假名、韩文等非可保留源文复制仍必须拒绝。
- 判断源文复制时必须与合并逻辑使用同一身份规则：先按编号+时间轴匹配，编号不一致时再按时间轴匹配；错编号但同时间轴的英文/日文/韩文等源文复制不得计入覆盖数。
- `src/services/translation_service.py` 负责在 Web 模式 SRT 输出不完整时优先在当前 DeepSeek 对话续发缺失片段；当前对话达到续译上限后，必须先用最新合并结果复核实际覆盖条数，再打开新的 DeepSeek 页面/对话补译缺失片段，并把结果合并回同一条历史记录。
- Web 模式 SRT 续译期间，原始续译捕获可以入库，但 UI 结果区必须显示已合并 SRT，不得用第二段裸输出覆盖第一段结果。
- Web 通用对话捕获模式必须复用 `src/engines/deepseek_web.py` 的可见文本捕获、自动继续生成和历史版本写入；不得绕过网页 DOM 去读取隐藏接口响应。
- Web 通用对话捕获模式不得启用 SRT 专用清理、SRT 完成判定、SRT 续译 Prompt 或新窗口补译；即使输入内容符合 SRT 结构，也应按普通网页回答保存。
- Web 通用对话捕获模式右侧状态必须显示内容长度，停止时整理当前捕获内容，不显示“正在整理 SRT”。
- 通用对话捕获模式必须隐藏或阻止 SRT 导出，只允许 TXT/Markdown 导出当前捕获内容。
- 剥离版 portable 启动脚本必须设置 `DCC_APP_DATA_DIR=%~dp0data`，不能设置主程序的 `DST_APP_DATA_DIR`。
- Web 模式第二次及后续发送 Prompt 前必须记录当前页面已显示译文作为本轮捕获基线；发送后旧译文不能被当成本轮新输出、不能触发稳定收尾，也不能让自动续译快速耗尽尝试次数并关闭浏览器。
- SRT 续译缺口定位必须优先按源字幕编号/时间轴的最高已覆盖位置判断尾部截断点；早期内部缺口只能在尾段完成后按“只补列出的缺失字幕”单独处理，不能导致从早期编号重译整段。当前输出含错误提示时必须先保留当前 DeepSeek 对话继续缺失尾段；只有当前对话续译上限耗尽且复核后仍不完整，才允许新开页面/对话补译缺失 SRT。
- 如果当前 SRT 捕获是稀疏片段（例如只出现 682、925、1162 等后段编号，前缀覆盖密度低于 50%），不得当成普通尾部截断继续往后补；必须从连续前缀后的第一条重新补译，避免覆盖数长期卡住。
- 如果当前 SRT 已显示到最后编号但存在大块内部缺口（例如从中段起缺失数百条），不得当成“小内部缺口列表补译”；必须从该大块缺口第一条开始重新补译后续字幕，再由合并器覆盖缺口。
- Web 模式 SRT 捕获一旦覆盖全部源字幕且无内部缺口，必须自动结束捕获并进入合并流程；不得继续因为页面 footer、点赞按钮或其他 DOM 变化持续记录 mutation。
- Web 模式 SRT 状态必须显示有效字幕条数 `已覆盖/总数`，不要用字符长度替代用户关心的字幕进度。
- Web 模式 SRT 捕获到拒答/异常提示时，不得把拒答文本显示到右侧或写入最终字幕；必须先保留当前对话并继续合并剩余 SRT，当前对话耗尽后只能用缺失片段打开新页面补译，禁止重跑整份源字幕。
- DeepSeek 显示“服务器繁忙，请稍后再试，或使用快速模式”并出现“继续生成”时，`src/engines/deepseek_web.py` 必须进入 `busy_retry` 状态，默认约 180 秒重试点击“继续生成”；不得点击“快速模式”，不得把普通 `max_continue_clicks` 当作服务器繁忙重试上限。
- Web 模式执行 SRT 任务时不得为了规避思考文本而强制关闭 DeepSeek “深度思考”；如果捕获到“正在思考/用户要求/这些要求”等非 SRT 内容，不能显示到右侧或计入字幕进度，必须通过清理、合并和当前对话重发严格 SRT 输出 Prompt 处理。
- Web 模式流式捕获 SRT 时，未完成的下一条字幕碎片（例如孤立编号加半截时间轴）不得并入上一条字幕文本；同一编号/时间轴出现多次时，合并结果必须优先使用较新的完整捕获。
- 清理 DeepSeek 思考/需求分析噪声时必须区分 SRT 结构位置；`这里要`、`用户要求`、`这些要求` 等文本如果位于字幕正文行内，必须保留为译文，不得触发后续字幕截断。
- 同一 DeepSeek 页面包含多段回答时，第一段 SRT 后如果出现 `已思考/我们被要求/开始输出` 等说明，且后面还有新的 SRT 块，清理逻辑必须跳过说明并继续提取后续 SRT；不得停在第一次截断位置。
- SRT 块之间的普通说明文字（例如 `下面是后续翻译结果`）不得进入右侧结果、历史记录或导出。
- 流式清理必须识别半截完整箭头时间轴，例如 `265` 后跟 `00:11:33,400 --> 00:11:34,5`，不得把它拼入上一条字幕正文。
- Web 模式捕获页面文本时，若 DOM 中存在 SRT 片段，应优先收集所有可见 SRT-like 节点并按 DOM 顺序合并，避免只选择较长的旧节点导致右侧进度停在早期字幕。
- 捕获版本高频写入时，服务层不得在每次 `record_version` 后重新全量读取该任务全部版本；当前任务的 best/error 状态必须用内存缓存维护，最终选择再读取完整版本。
- 结果区收到新的 Web/SRT 文本后必须滚动到末尾，保证用户看到最新同步进度。
- `src/engines/deepseek_web.py` 只捕获页面 DOM 中已经显示的文本，不读取隐藏接口响应。
- `src/engines/deepseek_web.py` 自动发送前必须验证 Prompt 已写入可见输入控件；不能只依赖 Playwright `fill()` 调用成功。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextWeb4/deepseek-translation-studio](https://github.com/NextWeb4/deepseek-translation-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
