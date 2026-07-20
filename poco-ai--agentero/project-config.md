---
trigger: always_on
description: Agentero 是一个基于 Tauri 2 + React 19 的本地优先科研工作台。Vault 中：人的笔记与 source 以 Markdown/文件为准；论文集合与结构化 metadata 以 `.agentero/catalog.sqlite` 为准（可导出 `PAPERS.md` / BibTeX，非默认落盘）。离开应用后笔记与源文件仍可被外部工具读取。
---

# AGENTS.md

## 项目概览

Agentero 是一个基于 Tauri 2 + React 19 的本地优先科研工作台。Vault 中：人的笔记与 source 以 Markdown/文件为准；论文集合与结构化 metadata 以 `.agentero/catalog.sqlite` 为准（可导出 `PAPERS.md` / BibTeX，非默认落盘）。离开应用后笔记与源文件仍可被外部工具读取。

## 当前应用形态

- 前端：`src/`（React、TypeScript、Tailwind CSS 4、shadcn/ui、AI Elements）。
- Host：`src-tauri/`（Rust、Tauri commands、本地文件系统、Wiki 索引、ACP Client）。
- CLI：`cli/`（package `agentero-cli`，bin **`agentero`**）— headless Vault/Catalog；path 依赖 `agentero_lib`；**无 BYOA / 无 paper-reader**（见 `docs/development/cli.md`）。
- 工作台布局：
  - 左侧：Vault 文件树（顶部虚拟 **Library** + 其下 **Recycle Bin**、魔棒、新建文件/文件夹；右键 **Finder 显示 / 终端打开 / 删除**）+ 选中论文时 **Paper Info**；
  - 中间：无 Vault 时欢迎页；有 Vault 时默认 **全库论文表格**；单击非 paper 文件夹在**同一 Library tab** 就地按路径筛选（不新建 tab）；或标签页打开的 **PDF / HTML / 图片** / Markdown 笔记；关光文档 tab 后回到全库；
  - 右侧 Notes：**仅**打开具体论文且 PDF/HTML 时显示该篇 `NOTES.md`（WYSIWYG，无独立预览栏）；
  - 可选右侧栏：`Agent` 或 `Backlinks`（与左栏均为 **常驻 collapsible**，`preserve-pixel-size`）。
  - **全局错误 Toast**（右上角 Sonner）：操作失败经 `notifyError`（`src/lib/notify.ts`）弹出；表单就地校验除外。
  - **Agent 禅模式**（`⌥⌘Z` / 标题栏 Layout「面板」菜单）：仅全屏 Agent 对话，复用 AI Elements `AgentPanel`（`variant="zen"`），不 remount 丢会话；左侧栏 Quest 式弱对比（新建 + 单行历史）；主区顶栏仅 Agent 切换（无 1/2/3 标签）；对话区全宽滚动 + AI Elements；标题栏返回图标退出。精读 / PDF 划词等后台运行不进对话历史。
  - **文档标签页**（浏览器式多 tab）位于**标题栏**（与 Layout / 侧栏图标同行）：可同时打开多个 paper / PDF / HTML / Markdown / Library（全库或文件夹作用域），切换、关闭、拖拽重排；每个 tab **常驻挂载**，切换保留 PDF 滚动/缩放与编辑器状态。快捷键：`⌘W`（仅剩全库时关窗，否则关 tab；关空后自动全库）、切换 `⌥⌘←/→`。**分屏（split）** 仍规划见 roadmap V0.6。
- 论文库：`paper_list` 读 catalog 一次进内存；表头排序；横向/纵向滚动；**tags** 列 + chip 筛选；**文件夹作用域**按 `paper.path` 前缀过滤（不扫盘、无 per-folder RPC）。虚拟路径 `agentero:library` 不写盘。
- 标签：Paper Info 增删与 **Apple 风格 8 色** → Host `paper_set_tags`（catalog `tags_json` 权威；元素为字符串或 `{name,color}`）；Library 染色 chip 与筛选；CLI `paper set-tags` / `list --tag` / `tags`（CLI 仅名称）。
- 魔棒入库：默认下载 PDF 到 **论文文件夹根目录** `{paper}/{id}.pdf`；arXiv 另解压 e-print LaTeX 到 `source/`。入库成功后刷新树并 `openPaper`，左侧文件树**展开并滚到新论文**。paper 行缺 PDF，或既无 TeX 也无 `PAPER.md` 时显示 Download（hover 说明原因）；Library 行可批量补下。
- **Zotero Connector 兼容**（MVP）：设置 → 通用开关（默认关）；Host 在 `127.0.0.1:23119` 收官方浏览器扩展 `saveItems` + **`saveAttachment`**（浏览器上传登录墙 PDF）→ 当前 Vault；组织子文件夹可选；与 Zotero 桌面端口互斥。见 `docs/backend/connector.md`。
- **可读正文**：TeX 与 `PAPER.md` 有其一即可（优先 TeX）。无 TeX 时下载后 liteparse 生成 `PAPER.md`；有 TeX 不强制 `PAPER.md`。
- **精读工作流**：设置 → Agent **`autoPaperReader`**（**默认关**）；开启后魔棒入库 / 单篇 Download 资源就绪且未读时自动 paper-reader。资源齐全且 `is_read === false` 时文件树 **Zap** 可手动。写入 `NOTES.md`，成功后 `is_read = true`。进度在左下角后台任务条（**hover 实色不透明**）。Skill 运行时语法按 Agent：**Claude `/id`**、其它（含 Codex）仅注入 `SKILL.md`。
- **Agent 权限**：设置 → Agent **全局权限模式**（`restricted` 默认 / **`ask` 每次询问** / `auto` 自动批准）；非 per-provider YOLO。`ask` 时弹权限对话框 → `agent_respond_permission`。
- **个人偏好提示词**：设置 → Agent **`agentPersonalPrompt`**（多行，默认空）；非空时经 `runOnce` → Host `build_prompt` 注入 envelope（所有 workflow）；留空不注入，Chat 不展示该块。
- **Agent 面板工作流**：空态建议 chips → `summary` / `qa` / `related_work`（Summarize、Ask library、Draft Related Work 等）；目标为当前聚焦 paper。Composer：**当前论文默认加入**上下文（实心 chip + paper-name/标题，无加号切换；可 X 移除）；`@` 提及（论文文件夹 + 目录 + paper 外 Markdown；空 `@` 优先最近路径与浅层目录树；行右 **›** 进入子目录、‹/`←`/`Esc` 返回；论文标签与文件树 `paperTreeLabelMode` 一致）与从文件树**拖入**路径均为可移除 context chip（路径引用，非 AI Elements Attachments 二进制）；chip 展示虚拟名（paper-name），prompt 仍用 Vault 相对路径；图标见 `context-path-icon`（**paper** → `ScrollText`，其它文件夹 → `Folder`，文件按扩展名）。
- **笔记写后审阅**：Agent 改写目标笔记后 `agent:notes-review` **统一 Diff**，Keep / Revert。
- **命令面板**（`⌘K` / `⌘P`）：论文 quick-open + Vault Markdown 全文搜索（`vault_search`）。
- 文件树：右键 / `⌥⌘R` 在 Finder 中显示（无双击）；右键 / `⌥⌘T` 在终端中打开（文件夹 = 自身 cwd，文件 = 父目录；系统默认终端）；**`⌘←` 折叠选中文件夹**、**`⇧⌘←` 折叠至默认**（只展开 `papers/`，不展开其子目录）；多选（⌘/Shift）+ 拖拽移动；**删除**走回收站（`path_trash` → `.agentero/.trash/`，**无确认 / 无 Undo toast**）；Library 下方虚拟节点 **Recycle Bin**（`agentero:trash`）打开中间栏回收站视图（恢复 / 永久删除 / 清空）。打开 Vault 时默认只展开 `papers/` 及其一级子目录；激活文档变化时树展开祖先并滚到对应行。Paper 行标签默认 **标题 · 作者**（设置 → 通用 `paperTreeLabelMode`，展示用、不改磁盘名）；同目录排序默认 **显示名称 A–Z**（与 `paperTreeLabelMode` 标签一致；`paperTreeSortMode`：标题 / 作者 / 年份 / 添加时间等预设，展示用）。
- 论文库：**Rescan**（`paper_rescan`）从 `papers/` + `metadata.json` 补齐盘上有、catalog 无的条目。
- PDF：Vault **任意路径** `.pdf` → `blob:` 预览；论文单元本地优先 → 自动下载 → 远程回退；**页码导航 / 适应宽·整页 / 大纲 / ⌘F 查找**；真实 scale 渲染 + 平滑划词覆盖层；划词操作菜单（高亮 / 批注 / 提问 / 翻译，见 `docs/development/pdf-ask.md`）。**批注** = 高亮 + 内联评论（`comment`），带 comment 的高亮显示页边批注针，右侧 **批注** 面板列出当前 PDF 的批注卡（跳转 / 编辑 / 删除），不写入 `NOTES.md`。
- **翻译服务**：应用级可插拔 `TranslateService`（免费 MT + BYOA Agent，无付费 API），设置 → 翻译；PDF 划词等为消费方；见 `docs/development/translate.md`。
- 图片：常见格式任意路径 `blob:` 中间栏预览。
- **Markdown 内嵌图片**：粘贴 / 工具栏 → `{mdDir}/assets/` + `![](./assets/…)`；选中显示源码；删节点且无引用时 GC（`src/lib/markdown-image.ts`）。
- **外部/Agent 改动自动重载**：Host `notify` → `vault:file-changed`（`watcher.rs` / `fs-watch.ts`）；打开中的 `.md`/`NOTES.md` 磁盘变化：**无未存改动则重载**；**有未存改动则 toast 提示**（不静默覆盖）；内容相等抑制自写回声；create/remove/rename 去抖刷新文件树。
- **Wiki 索引**：`.md` 变更防抖重建（`scheduleWikiRebuild`，~900ms），Backlinks/Graph 保持新鲜。
- **保存冲突**：写盘前比对上次落盘内容；磁盘已被外部改则中止写入并警告（`diskConflict.saveBlocked`）。
- **运行日志**：Host `tauri-plugin-log` + 前端 `logger` + CLI `env_logger`；关键操作 op start/end（见 `docs/development/logging.md`）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poco-ai/Agentero](https://github.com/poco-ai/Agentero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
