---
trigger: always_on
description: Zotero 8/9 插件，调用 MinerU 官方 API 解析 PDF 为 Markdown 文件附件，并支持 LLM AI 中文总结。
---

# Zotero MinerU Plugin

Zotero 8/9 插件，调用 MinerU 官方 API 解析 PDF 为 Markdown 文件附件，并支持 LLM AI 中文总结。

## 项目结构

```
manifest.json          # 插件元数据 (Zotero 8.0 到 9.0.*, ID: zotero-mineru@example.com)
bootstrap.js           # 插件生命周期入口 (install/startup/shutdown/uninstall)
mineru.js              # 核心业务逻辑 (~2200 行，菜单注册、PDF 解析、AI 总结、Markdown→HTML)
preferences.js         # 设置面板控制 (加载/保存/连接测试)
preferences.xhtml      # 设置面板布局 (XUL/HTML 混合)
preferences.css        # 设置面板样式
prefs.js               # 偏好默认值
build-xpi.sh           # 打包脚本 → zotero-mineru-VERSION.xpi (使用 bsdtar)
updates.json           # 自动更新元数据
.github/workflows/
  release.yml          # GitHub Actions 发布工作流 (tag v* 触发)
.gitignore             # 忽略 *.xpi 文件
locale/
  en-US/zotero-mineru.ftl   # 英文菜单标签
  zh-CN/zotero-mineru.ftl   # 中文菜单标签
icon.svg / icon16.svg       # 插件图标
```

## 构建

```bash
bash build-xpi.sh
```

输出：`zotero-mineru-VERSION.xpi`（版本号读取自 manifest.json）。

版本号修改：编辑 `manifest.json` 中的 `"version"` 字段。

### 打包新版本流程

当用户要求"打包"或"打包为新版本"时：

1. 先将 `manifest.json` 中的 `"version"` 递增（patch +1，如 `0.1.43` → `0.1.44`）
2. 执行 `bash build-xpi.sh`
3. 确认输出的 `.xpi` 文件名包含新版本号

### 发布流程 (GitHub Actions)

`release.yml` 由 `v*` tag 触发：
1. 校验 tag 版本与 manifest.json 一致
2. 构建 XPI
3. 更新 updates.json 中的下载链接并提交回 main
4. 创建 GitHub Release（附带 XPI 文件）

### 手动发布新版本的实际顺序

当用户要求“发布新版本”时，按下面顺序执行，避免 tag 指向错误提交：

1. 先把 `manifest.json` 的 `"version"` 递增到目标版本
2. 如需本地验证，执行 `bash build-xpi.sh`，确认生成的 `.xpi` 文件名包含新版本号
3. 提交版本变更并先 `git push origin main`
4. 创建与 `manifest.json` 一致的 tag，例如 `git tag v0.1.49`
5. 推送 tag：`git push origin v0.1.49`
6. 等待 GitHub Actions `release.yml` 完成发布

补充说明：
- `release.yml` 会重新生成 `updates.json`，并仅在文件内容发生变化时自动提交回 `main`
- 如果 `main` 上的 `updates.json` 已经是目标版本和正确下载链接，workflow 不会再额外创建一个回写提交
- 真正触发 GitHub Release 和 XPI 上传的是 `v*` tag，不是 `main` 分支提交

## 架构要点

### 插件加载流程

1. `bootstrap.js` → 注册偏好面板（3 种策略依次回退）
2. `Services.scriptloader.loadSubScript()` 加载 `mineru.js`
3. `ZoteroMineru.init()` → `addToAllWindows()` → 注册右键菜单
4. `ZoteroMineru.main()` 异步启动

### 菜单注册（双轨制）

- **Zotero 8/9 MenuManager**（主路径）：`Zotero.MenuManager.registerMenu()` 注册 `ROOT_MENU_ID`，父菜单下包含解析、Markdown 转笔记、总结、翻译四个子项
- **XUL 回退**（旧版兼容）：`createXULElement("menu")` + `menupopup` + `popupshowing` 事件

上下文菜单现在是一个父菜单 + 四个子菜单项：
- `ROOT_MENU_ID`（"zotero-mineru-menu"）— 父菜单
- `CONTEXT_MENU_ID`（"zotero-mineru-parse-pdf"）— PDF 解析
- `MARKDOWN_NOTE_MENU_ID`（"zotero-mineru-markdown-to-note"）— 将 `#MinerU-Parse` Markdown 附件转为 Zotero 笔记
- `SUMMARY_MENU_ID`（"zotero-mineru-ai-summary"）— AI 总结
- `TRANSLATE_MENU_ID`（"zotero-mineru-ai-translate"）— AI 翻译

卸载时 `unregisterMenu(ROOT_MENU_ID)` 清理 MenuManager 注册。

### 偏好分支

`"extensions.zotero-mineru."` — 所有 pref key 均以此为前缀。

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| apiBaseURL | string | `https://mineru.net/api/v4` | MinerU API 地址 |
| apiToken | string | | API Token（自动去 `Bearer ` 前缀） |
| modelVersion | string | `pipeline` | `pipeline` 或 `vlm` |
| noteIncludeImages | bool | `false` | 将 Markdown 转为笔记时是否嵌入图片；默认关闭以减少 Zotero 卡顿 |
| pollIntervalSec | int | 3 | 轮询间隔（秒） |
| timeoutSec | int | 120 | 请求超时（秒） |
| noteTitlePrefix | string | `MinerU Parse` | Markdown 文件名前缀（也用于旧版笔记标题） |
| llmApiBaseURL | string | | LLM API 地址 |
| llmApiKey | string | | LLM API Key（自动去 `Bearer ` 前缀） |
| llmModel | string | | LLM 模型名称 |
| summaryLanguage | string | `中文` | AI 总结输出语言 |
| translateLanguage | string | `中文` | AI 翻译目标语言 |
| translateChunkSize | int | 20000 | 翻译分段字符数（长文分段翻译） |
| translateConcurrency | int | 3 | 翻译并发请求数（建议 1-4，过高可能触发限流） |
| translateRetryCount | int | 2 | 每段翻译自动重试次数，超出后再询问是否只重试失败段 |

`apiToken` 有 legacy `apiKey` 回退逻辑。

### 标签体系

| 标签 | 附加对象 | 用途 |
|------|---------|------|
| `#MinerU-Parse` | Markdown 附件 | 标识由 MinerU 解析生成的 Markdown 附件（旧版为笔记） |
| `#MinerU-Parsed` | 父条目 | 标识已完成解析的文献条目 |
| `#MinerU-Summary` | 笔记 | 标识由 AI 总结生成的笔记 |
| `#MinerU-Translation` | Markdown 附件 | 标识由 AI 翻译生成的 Markdown 附件 |

### 防重复机制

- **解析**：`collectPDFTasks()` 检查父条目子附件或子笔记是否含 `#MinerU-Parse` 标签（兼容新旧格式），有则跳过
- **Markdown 转笔记**：`collectMarkdownToNoteTasks()` 检查是否存在 `#MinerU-Parse` Markdown 附件，且父条目还没有 `#MinerU-Parse` 笔记
- **总结**：`collectSummaryTasks()` 检查是否含 `#MinerU-Summary` 笔记，有则跳过
- **翻译**：`collectTranslateTasks()` 检查是否含 `#MinerU-Translation` 附件，有则跳过
- 用户删除对应附件/笔记或标签即可重新触发

## PDF 解析流程

1. 读取本地 PDF → `IOUtils.read()`
2. 申请上传地址 → POST `/file-urls/batch`
3. 上传 PDF → PUT 到返回的 upload URL
4. 轮询结果 → GET `/extract-results/batch/{batchID}`（`pollMineruExtractResult`）
5. 下载 ZIP → 4 种下载策略回退（直连 / Bearer 认证 / HTTP→HTTPS / HTTPS+Bearer）
6. 提取 Markdown → 解压 ZIP，`pickMarkdownEntry()` 选择 `.md` 文件（优先匹配原始文件名）
7. 保存为 Markdown 附件 → `saveResultAsMarkdownAttachment()`：
   - 默认通过 `rewriteImagePathsForStorage()` 将图片引用重写为 `images/<filename>` 相对路径
   - 调用 `Zotero.Attachments.importFromFile()` 创建 stored file attachment
   - 在附件存储目录下创建 `images/` 子目录，写入所有图片文件
   - 给附件打 `#MinerU-Parse` tag，给父条目打 `#MinerU-Parsed` tag

存储结构：
```
Parent Item (论文条目)
├── 原始 PDF (附件)
├── MinerU Parse - xxx.md (stored attachment, tagged #MinerU-Parse)
│   └── storage/<key>/
│       ├── MinerU Parse - xxx.md
│       └── images/
│           ├── fig1.png
│           ├── fig2.png
│           └── ...
├── MinerU Parse xxx (笔记, tagged #MinerU-Parse)   # 可通过“Markdown 转笔记”菜单生成
└── AI Summary (笔记, tagged #MinerU-Summary)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lisontowind/zotero-mineru](https://github.com/lisontowind/zotero-mineru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
