---
trigger: always_on
description: **每执行一步前必须先向用户汇报计划，等待确认后再执行。**
---

# jdnotes 项目说明

## 重要原则

**每执行一步前必须先向用户汇报计划，等待确认后再执行。**
不得连续执行多个破坏性或不可逆操作（如 git push、release 发布、tag 删除）而不经用户同意。

**git commit 信息必须使用中文。**
**git commit 不要加 Co-Authored-By 行。**

---

## 项目概况

- **名称**：Lapis — 简洁高效的本地笔记应用
- **技术栈**：Tauri v2 (tauri 2.9.5, tauri-build 2.5.3) + Vite + React + TypeScript
- **包管理器**：pnpm
- **版本**：2.2.1
- **标识符**：com.jdnotes.app
- **窗口**：1556x887（默认=最小，逻辑内容尺寸；无边框窗口外框会大 16x9 隐形拉伸边框），无边框 (decorations: false)
- **前端开发端口**：5173
- **GitHub**：zexadev/lapisnote
- **品牌**：Zexa (zexa.cc)

---

## 基建

- **CI/CD**：GitHub Actions 构建发布（`.github/workflows/release.yml`，推 `v*` tag 触发，windows-latest）。Release body 从 `docs/src/content/changelog.mdx` 里手写的 `## vX.Y.Z` 小节抽取——**更新日志始终人写，不用 commit 自动生成**，抽不到直接让 workflow 失败。`latest.json` 由 tauri-action 生成上传，不再手工拼
- **文档站**：Nextra 4 (Next.js)，位于 `docs/`，静态导出
- **文档部署**：Cloudflare Pages，域名 jdnotes.zexa.cc
- **数据库**：SQLite（通过 tauri-plugin-sql；前端 db.ts 直接执行 SQL，plugin 在 Rust 侧原生跑）
- **Tauri 插件**：log, notification, sql(sqlite), dialog, fs, opener, updater, process
- **发布状态**：v2.0.0 已于 2026-07-28 正式发布。仓库最终名 `zexadev/lapisnote`（jdnotes → lapis → lapisnote，lapis 与既有 GitHub 项目及商业软件撞名故加 note 后缀；应用品牌仍叫 Lapis）。Release 资产齐全（exe/msi + 双 sig + latest.json）、updater 端点已验证 200。旧地址 `zexadev/jdnotes`、`zexadev/lapis` 均由 GitHub 自动重定向，**这两个旧名永不复用**（复用即断老版本 updater 的重定向链）。

---

## 关键文件路径

| 文件 | 说明 |
|------|------|
| `src-tauri/tauri.conf.json` | Tauri 配置、版本号 |
| `src-tauri/Cargo.toml` | Rust 依赖、版本号 |
| `src-tauri/src/db.rs` | 配置管理、AI 来源、数据库路径 |
| `src-tauri/src/commands.rs` | Tauri 后端命令 |
| `src-tauri/src/lib.rs` | 插件注册、命令注册；发布版禁 WebView2 CDP 注入（防君子）：run() 开头 remove_var 清 WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS 等三个变量——Playwright 靠它塞 --remote-debugging-port 开调试端口接管页面，创建 webview 前清掉即失效；仅 release，debug 保留供本机 CDP 测试。（不做注册表扫描/进程巡检那套军备竞赛，防不了同权限攻击者，数据真正保护靠 DB 落盘加密[roadmap]） |
| `src-tauri/src/sync.rs` | 多设备同步内核（局域网 TCP + iroh 跨网 + 同步包文件、三路合并、设备 ID 持久化、probe、mDNS 自动发现、持久 fingerprint） |
| `src-tauri/src/attachments.rs` | 图片附件内容寻址存储（sha256） |
| `src-tauri/migrations/004_sync.sql`·`005_sync_merge.sql`·`006_private.sql` | 同步 uuid + 三路合并基准/冲突标记 + 私有笔记标记 |
| `src/pages/SettingsPage.tsx` | 设置页左侧导航容器（应用实际使用的设置 UI） |
| `src/pages/settings/SyncSettings.tsx` | 设置「设备同步」页（mDNS 自动发现 / 跨网设备列表 / 同步包 / 清理图片） |
| `src/components/modals/NoteSelectModal.tsx` | 局域网笔记多选同步弹窗（搜索/全选/单选/卡片勾选，自动排除私有笔记） |
| `src/components/modals/PairingCodeModal.tsx` | 首次配对码弹窗（双方各算 6 位数字防中间人） |
| `src/lib/pairing.ts` | 配对码工具（SHA256 派生 + localStorage 白名单） |
| `src/hooks/useSettings.ts` | AI 多来源配置 Hook（useAIConfig / useSettings） |
| `src/components/modals/ChangelogModal.tsx` | 应用内更新日志（CHANGELOG_DATA 数组） |
| `src/components/ai/chat/` | AI 侧栏组件族（分块 memo 流式 Markdown、ThinkingBlock 思考折叠、ToolCallCard、ChatInput 一体化输入卡——模型选择器/占用进度条在卡内底行+粘贴/拖拽附图+图片预览+挂载自动聚焦、ModelPicker 带窗口 badge、ConversationSwitcher 对话下拉——行内重命名/删除两段确认/底部新建、CompactDivider 压缩点分隔线、useStickToBottom 粘底、字符级平滑排字在 useChat；侧栏开合为宽度滑入滑出动画） |
| `src/hooks/useChat.ts` | 聊天状态机（thinking/text/tool 段、完成/停止/出错统一落库到流开始时捕获的 streamTargetRef、skipPersistRef 竞态防护、平滑排字 drain 循环、上下文压缩 runCompaction/自动压缩/contextUsage 指示、切笔记/切对话统一中断保存 interruptStreamAndSavePartial、对话自动命名 maybeAutoTitleConversation——仅默认「对话 N」标题才起名） |
| `src/hooks/useAIStream.ts` | 模型调用层（4 provider 流式+工具循环、429/5xx 指数退避重试 callWithRetry、旧工具结果 microcompact 折叠、Anthropic prompt cache + 旧模型 max_tokens 400 回退、generateOnce 单次生成供压缩用） |
| `src/lib/contextBudget.ts` | 上下文预算（token 估算 CJK=1/字·其他/3.5、每模型真实窗口表 inferContextWindow、手填 contextWindow 优先、64k 兜底、自动压缩阈值 0.7、图片/工具 schema 开销、压缩器系统提示） |
| `src/lib/tagColor.ts` | 标签颜色（标签名 djb2 哈希 → 12 色盘，零存储、处处一致；侧栏图标与 TagsInput chip 共用） |
| `src/components/layout/Sidebar.tsx` | 左侧导航（标签区按使用数降序、默认 Top8+激活钉住、展开全部带筛选——标签上百平铺列表失控） |
| `src/contexts/ThemeContext.tsx` | 主题（切换用 View Transitions：toggleTheme 可传扩散原点——主题开关传自己中心 → clip-path 圆形揭示 700ms（data-theme-vt=circle 关掉默认交叉淡化，否则打架）；无原点入口如命令面板 → 整页交叉淡化。theme-switching 瞬态禁全部元素级过渡——各处 transition-colors 时长不一逐个变色显得零碎；主题开关自身动画豁免） |
| `src/hooks/useEditorAI.ts` | 编辑器内联 AI（Cursor 式就地 diff：原文标 aiOld 红删除线保留、新文本 aiHighlight 绿标紧随流式生长、接受/放弃/重试/追加指令按范围操作；插入必须用显式 marks 建文本节点——tr.insertText 会继承插入点 marks 导致红标漏进正文；换行语义：开头 \n 丢弃、\n\n 用 tr.split 真分段（range.to +2）、单 \n 硬换行、尾部换行悬挂跨 chunk） |
| `src/components/ai/AIReviewToolbar.tsx`·`AIInlinePrompt.tsx`·`AIOldMark.ts` | 浮动审查条（跟随生成位置，Tab/Ctrl+Enter 接受、Esc 放弃、重试、追加指令）·Ctrl+J 输入条（快捷动作 chips，为唯一 AI 面板——气泡菜单/斜杠「自由提问」都只是它的入口）·原文红标 mark |
| `src/lib/aiTools.ts` | AI 工具层（结果必须带 id、读取带截断分页 offset、append_note/list_notes、Gemini 空 schema 兼容） |
| `src/lib/chatParts.ts` | assistant 消息 parts JSON 解析（UI 渲染与回传模型共用，回传只取 text 段） |
| `src/lib/db.ts` | 前端数据库操作、初始化欢迎笔记 |
| `src/components/editor/Editor.tsx` | Tiptap 编辑器主组件（userTouchedRef：打开后扩展的规范化事务如 fixTables 不当作编辑上报——否则没编辑就刷 updated_at；CodeBlock language 属性 parseHTML 必须回退 language-xxx class，只认 data-language 会把 markdown 代码块语言洗成 plaintext；代码粘贴在 handlePaste 拦截——vscode-editor-data 直接建带语言代码块、含 \`\`\` 围栏的纯文本走块级 insertContent，默认 clipboardTextParser 的开放 slice 会把代码块拍成行内裸文本） |
| `src/components/editor/EditorToolbar.tsx` | 编辑器固定工具栏（格式/列表/待办/图片） |
| `src/components/editor/ResizableImage.tsx` | 图片节点组件（预览/缩放/删除） |
| `src/components/editor/SlashCommand.tsx` | 斜杠命令菜单（编辑器命令 + AI 命令；过滤词=编辑器里 / 后的真实文本——中文/IME 天然支持，keywords 拼音/英文别名，键盘 capture 拦截防 Enter 漏进编辑器换行） |
| `src/hooks/useSlashCommand.ts` | 斜杠命令逻辑（位置计算/命令执行/slashQuery 产出/光标移出即关） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zexadev/lapisnote](https://github.com/zexadev/lapisnote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
