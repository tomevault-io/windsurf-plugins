---
trigger: always_on
description: > 给在本项目里写代码的 AI（Cursor / Claude / Codex 等）看的长期记忆。人看的使用说明在 `README.md`。
---

# RawPhotos · 工程记忆（AI 助手专用）

> 给在本项目里写代码的 AI（Cursor / Claude / Codex 等）看的长期记忆。人看的使用说明在 `README.md`。
> 本文件侧重「整体结构 / 为什么这么设计 / 改代码要注意什么」。

## 一句话

Electron + Vue 3 桌面应用：通过任意 **OpenAI 兼容的中转 / 代理接口** 做 **文生图 / 文生视频 / 图生图 / AI 对话**，并带 **多接口管理、用量统计、中转额度、运行日志、主题**。
**面向中转/聚合代理（不是本机服务）**——写文案/默认值别默认「本地」，用「中转接口」这类中性措辞。
所有网络与文件操作都在**主进程**，渲染层只通过 `window.api`（preload 暴露）调用，**绝不**在渲染层直连接口或读写磁盘。

## 技术栈 / 命令 / 风格

- Electron 42 · electron-vite · Vue 3.5（`<script setup>`）· electron-builder 26 · `marked`(对话 Markdown)。Node 18+。
- 开发 `npm run dev`（electron-vite dev：改主/预加载会**自动重启**主进程，改渲染层是 HMR）；构建校验 `npm run build`；打包 `npm run dist:win`。
- 代码风格：**2 空格、单引号、不写分号**，跟现有文件一致。
- 改了主进程（`src/main` / `src/preload`）需要重启 dev 才生效（一般自动重启）；纯渲染层热更新。

## 进程 / 文件地图

```
src/main/index.js      主进程：设置读写+迁移、全部 IPC、调接口(图/视频/图生图/对话/优化)、
                       额度/用量/日志/对话历史、托盘、通知、rawmedia 协议、窗口
src/preload/index.js   contextBridge 暴露 window.api（含纯函数 mediaUrl）
src/renderer/src/
  store.js             reactive：settings + 本次会话 results；THEMES/applyTheme/setTheme/setCustomColor
  App.vue              外壳：侧栏导航(生成/对话/画廊/统计/日志/设置/关于) + 主题色点 + 左下角余额 +
                       状态卡 + 关闭弹窗(托盘/退出) + 额度预警检查(定时器)
  components/
    GenerateView.vue   图片/视频切换 + 接口下拉 + 模型下拉(自动匹配/手动) + 提示词 + AI优化 +
                       图生图参考图 + 生成队列(顺序执行) + 结果网格
    ChatView.vue       AI 对话：左历史会话列表 + 右对话；底部独立「接口」「模型」下拉；
                       图片/文本上传、Markdown 渲染、停止、清空、导出 md、复制
    GalleryView.vue    本地图片(dataUrl)+视频(rawmedia://) 网格 + 全部/图片/视频筛选 + 另存为
    StatsView.vue      中转额度卡 + 本地累计用量(byKind/byModel/byDay) + 成功率 + 7天柱状（60s 定时刷新）
    LogsView.vue       运行日志（出图/出片/优化/测试/对话 + 原始返回，可清空/打开文件）
    SettingsView.vue   多接口增删改/设为当前(即时持久化)/测试连接 + 外观主题(含自定义色) +
                       通用(数量/目录) + 额度预警(开关+阈值)
    AboutView.vue      关于（版本 app:version + 网站链接 + QQ 群 1105572960 复制 + 功能一览）
    MediaCard.vue      会话结果卡（图片<img>/视频<video>，保存/另存为/复制提示词）
    Lightbox.vue       放大预览（图片或 controls 视频，按 kind）
    Dropdown.vue       themed 下拉（替代原生 select；下方空间不足自动向上弹；options 支持 string[]/{value,label}[]）
    Icon.vue           内置 SVG path 图标库（加图标进 PATHS；填充图标进 FILLED）
    TitleBar.vue       无边框标题栏（品牌 + 网站链接 + 最小化/最大化/关闭(emit close-request)）
    ToastHost.vue      全局 toast
  composables/useToast.js
```

## 持久化文件（都在 `app.getPath('userData')`）

| 文件 | 内容 |
| --- | --- |
| `settings.json` | 全部设置（见下） |
| `rawphotos.log` | 运行日志（**JSONL**，每行一个完整 entry，启动读回 ≤300 行） |
| `rawphotos-usage.json` | 累计用量（byKind/byModel/byDay/ok/fail/firstAt，长期累计） |
| `rawphotos-chats.json` | 对话历史（每会话含完整 messages，≤80 个；**立即落盘**不 debounce） |

## 设置数据结构（settings.json）

```js
{
  providers: [{
    id, name, baseUrl, apiKey,
    imageModel, videoModel, optimizeModel,   // 三类模型分开
    imageSize, videoSize, videoSeconds,
    editPath: '/images/edits',               // 高级：图生图端点
    videoPath: '/videos/generations',        // 高级：出片端点
    videoPollPath: ''                        // 高级：异步出片轮询（{id} 占位）
  }],
  activeProviderId,        // 生成页用的当前接口
  chatProviderId,          // 对话页用的接口（可与生成不同分组！）
  chatModel,               // 对话模型
  defaultCount,            // 默认出图数量
  saveDir,                 // 图片/视频保存目录（空=图片/RawPhotos）
  theme,                   // 'sky'(默认) | 'green' | 'dark' | 'custom'
  customColor,             // 自定义主题强调色 hex
  closeAction,             // 'ask'(默认) | 'tray' | 'quit'
  alertEnabled, alertThreshold  // 额度预警
}
```

- `migrate()`（主进程）：旧扁平配置→provider；补全字段；非法 theme→sky；历史「本地 CLIProxyAPI」名→「中转接口」；`grok-2-image`→`grok-imagine-image`；空 videoModel→`grok-imagine-video`。
- 改设置统一 `settings:set`（renderer 传完整字段，主进程 `writeSettings` 合并+再 migrate）。`persistSettings(patch)` 只传变化字段，其余保留。

## IPC 一览（preload → 主进程）

| window.api | channel | 说明 |
| --- | --- | --- |
| getSettings/saveSettings | settings:get / :set | 读写设置（含迁移） |
| testConnection(provider) | connection:test | 测 `/models`（会记日志） |
| listModels(provider) | models:list | 静默拉 `/models`（不记日志，给下拉填充） |
| generateImage / editImage | image:generate / image:edit | 文生图(JSON) / 图生图(multipart `/images/edits`) |
| generateVideo | video:generate | `videoPath`，同步抽取或异步轮询 |
| optimizePrompt({prompt,mode}) | prompt:optimize | 用 `optimizeModel` 扩写提示词（按 mode 切系统提示） |
| chatSend({providerId,model,messages}) | chat:send | 对话，转发完整 messages（vision content 数组）；可被 chat:abort 停止 |
| chatAbort | chat:abort | 停止当前对话（`activeChatAbort`；requestJson 支持外部 signal） |
| chatsList/Get/Save/Delete | chats:* | 对话历史 CRUD（立即落盘） |
| saveMedia/saveMediaAs | media:save / :saveAs | 保存图片/视频（payload 支持 b64/url/path） |
| saveTextFile | file:saveText | 保存文本（对话导出 md） |
| listGallery | gallery:list | 画廊：图片 dataUrl + 视频 path |
| mediaUrl(absPath) | （纯函数） | 转 `rawmedia://media/?p=<encodeURIComponent(path)>` |
| getUsage/resetUsage | usage:get / :reset | 累计用量 |
| getQuota(provider?) | quota:get | 中转额度（见下）；**失败返回 `{error}` 不抛异常**（防控制台刷屏） |
| getLogs/clearLogs/openLogFile · onLog | logs:* | 运行日志 |
| openExternal/quitApp/getVersion/notify | app:* | 外链/退出/版本/系统通知 |
| pickDir/openPath/defaultSaveDir | app:* | 目录 |
| window.* | window:* | 最小化/最大化/隐藏(=收起托盘)/状态 |

## 关键设计 / 约束（改前必读）

1. **接口返回格式很杂，统一 `pickMediaItems(json)` 抽取**媒体（data[].b64_json/url、url/video_url/output/result.url…）。新增供应商优先扩展它。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yz46bbbqqz-rgb/RawPhotos](https://github.com/yz46bbbqqz-rgb/RawPhotos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
