---
trigger: always_on
description: Swift + SwiftUI 写的 macOS 应用。点击顶部刘海胶囊呼出聊天窗口，对话对象可以是：
---

# HermesPet — macOS 顶部刘海桌宠 + AI 聊天客户端

Swift + SwiftUI 写的 macOS 应用。点击顶部刘海胶囊呼出聊天窗口，对话对象可以是：
- **Hermes Gateway**（OpenAI 兼容 HTTP API）
- **Claude Code CLI**（spawn `claude -p` 子进程，能读写文件 / 跑命令）

---

## 文件分工

| 文件 | 职责 |
|---|---|
| `HermesPetApp.swift` | AppDelegate，统筹各 controller / 全局热键 / 菜单栏 / 语音热键串联 |
| `ChatViewModel.swift` | `@MainActor @Observable`，多对话状态 + 流式请求 + 持久化 |
| `ChatView.swift` | 聊天主界面，包含 header / messages / 输入栏 / 对话胶囊 / 快捷启动卡片 |
| `ChatComponents.swift` | MessageBubble、ChatInputField、SendButton、SendOnEnterTextEditor、ImageThumb |
| `DynamicIslandController.swift` | 顶部刘海胶囊（NSWindow + SwiftUI），右耳任务状态指示器 |
| `ChatWindowController.swift` | 聊天窗口（NSWindow，从胶囊位置展开/收回动画） |
| `IntelligenceOverlay.swift` | 按住语音时全屏 **Apple Intelligence 风格**彩色光环 |
| `VoiceInputController.swift` | 按住说话录音 + SFSpeechRecognizer 实时识别（zh-CN） |
| `ScreenCapture.swift` | ScreenCaptureKit 截屏（必须，CGDisplayCreateImage 已失效） |
| `GlobalHotkey.swift` | Carbon Event Manager 注册全局热键（含 down/up 双事件） |
| `Models.swift` | ChatMessage / Conversation / AgentMode / API 数据类型 |
| `StorageManager.swift` | `~/.hermespet/conversations.json` 持久化 |
| `APIClient.swift` | Hermes Gateway 流式 HTTP 客户端 |
| `ClaudeCodeClient.swift` | spawn `claude -p`，解析 stream-json 增量输出 |
| `MarkdownRenderer.swift` | 自定义 Markdown 渲染（代码块带复制按钮、行内代码等） |
| `SettingsView.swift` | Form(.grouped) macOS 系统设置风格 |
| `AnimationTokens.swift` | 全局 spring 动画 token（snappy / smooth / bouncy / exit / breathe） |

---

## 全局快捷键

| 组合 | 功能 |
|---|---|
| `Cmd+Shift+H` | 切换聊天窗口显示/隐藏 |
| `Cmd+Shift+J` | 截屏并附加到当前对话 |
| `Cmd+Shift+V` | **按住说话**（push-to-talk），松开自动发送 |

---

## 三个 Shell 脚本

| 脚本 | 用途 | 签名方式 | 权限稳定性 |
|---|---|---|---|
| `build.sh` | 仅构建 `.app` | 自动选 Apple Development 证书，没有就 ad-hoc | — |
| `install.sh` | 构建 + 覆盖装到 `/Applications` + 启动（**日常用这个**） | Apple Development | **永久稳定** |
| `make-dmg.sh` | 生成给别人分发的 DMG | ad-hoc | 接收方升级要重新授权 |

---

## 关键技术决策（含踩过的坑）

### 1. 截屏必须用 ScreenCaptureKit
- macOS 15+ 上 `CGDisplayCreateImage` 已**返回 nil**（即便有权限）
- 改用 `SCShareableContent` + `SCScreenshotManager.captureImage`
- **不要预检 `CGPreflightScreenCaptureAccess`** —— ad-hoc 签名换 CDHash 后会假返回 false。直接尝试 SCK，让它自己决定。
- 返回值用 enum 区分 `.success` / `.needsPermission` / `.failed`

### 2. 签名：用本地证书让 TCC 权限稳定
- ad-hoc 签名（`codesign --sign -`）每次构建 CDHash 都变 → TCC 把每次构建当成新 app → 权限丢失
- 用户已有 **Apple Development 证书**（`1050246343@qq.com`, Team ID `R34KL4X4D9`），TCC 认 (TeamID + BundleID)，永久稳定
- `build.sh` 自动用 `security find-identity` 选证书

### 3. Claude Code spawn 必须加 permission 参数
spawn `claude -p` 时默认 permission-mode 会拒绝所有需要确认的工具调用，导致 AI 看不到附带图片、写不出文件。必须传：
```
--permission-mode acceptEdits
--add-dir ~/Library/Caches/HermesPet
--add-dir ~/Desktop
```

### 4. Swift 6 并发：避免 @MainActor 类的 closure 被传到后台线程
- `@MainActor` 类的内部 closure 会被自动推断为 @MainActor 隔离
- 把这种 closure 传给 SFSpeechRecognizer / `installTap` / SCStream 等系统 API → 回调在**后台线程**执行 → Swift 6 runtime 检测到 isolation 不匹配 → **SIGTRAP 必崩**
- **大量后台回调的 controller 必须改成 `final class XXX: @unchecked Sendable`**，可变状态用 NSLock 保护
- 已踩过坑的：VoiceInputController（按住 ⌘⇧V 必崩，已修）

### 5. 跨窗口动画的嵌套 layout 坑
- `ChatWindowController.show/hide` 内的 setFrame **不能同步触发别的 window 的 setFrame**
- 否则 NSHostingView.windowDidLayout 触发嵌套 layout cycle → macOS 26 抛 NSException → 必崩
- 跨窗口同步用 `DispatchQueue.main.async` 隔到下一个 runloop（已踩过坑的：灵动岛 compact 形态联动）

### 6. UI 设计：HIG 输入栏
- 输入栏用 Capsule(20pt 圆角) 容器，包输入框 + 28pt 圆按钮
- **Capsule 半径 = height/2，内容必须避开左右半圆**，所以 leading/trailing padding 至少等于半径（容器高 36 → 半径 18 → leading 14pt 起）
- Placeholder 用 1-2 字名词（"消息"），HIG 反对长 hint
- focus 反馈克制（参考 iMessage，不加亮眼描边，靠 NSTextView caret 自己表达）

### 7. ad-hoc 签名要避免 ChatView 的 frame 跟 Window 不一致
- 之前 `.frame(minWidth: 360, minHeight: 360)` + ChatWindow hide 动画缩到 100×30 → SwiftUI 反向请求 window 改 frame → 嵌套 layout → 崩
- 必须用 `.frame(maxWidth: .infinity, maxHeight: .infinity)` + 让 NSWindow.contentMinSize 控制最小尺寸

### 8. 三个 AgentMode 各自怎么传图片（容易漏！）
| Mode | 图片传递方式 |
|---|---|
| **Hermes** | OpenAI 兼容 multimodal：`APIMessage.content` 用 `[{type:"text"},{type:"image_url"}]` 数组，base64 data URL |
| **Claude Code** | `ClaudeCodeClient.saveImagesToTemp()` 把图写到 `~/Library/Caches/HermesPet/`，prompt 里告诉 Claude "图片在 /xxx/xxx.png，请用 Read 工具查看"。**必须配 `--add-dir`** |
| **Codex** | `codex exec -i <path1> -i <path2> -- "prompt"` 原生视觉参数。⚠️ **`-i <FILE>...` 是 clap greedy flag**，会吞掉后面所有参数当图片路径，**必须用 `--` 显式终止**才能让 PROMPT positional 参数被识别，否则 codex 转去等 stdin 报 "No prompt provided via stdin"（踩过两次） |

加新 AgentMode 时务必检查图片传递路径，别只拼文本 prompt 就完事。

### 9. 图片持久化方案（image Data + imagePaths 双写）
- `ChatMessage` 同时持 `images: [Data]`（内存，UI 显示用）+ `imagePaths: [String]`（磁盘绝对路径，序列化用）
- **encode 只写 imagePaths**（避免 base64 让 JSON 爆 MB），**decode 时从 imagePaths 还原 images**
- 落盘位置：`~/.hermespet/images/<groupID>-<idx>.png`
- 写盘 / 删盘统一走 `StorageManager.persistImages()` / `deleteImageFiles()`
- 用户附图 → 在 `sendMessage` 创建 user message 前就 persist
- Codex 生成的图 → 在 stream 完成后从 `~/.codex/generated_images/` diff 拿到，再 persist

### 10. ViewModel 状态变更必须在 UI 有对应渲染（errorMessage 教训）
踩过的坑：`ChatViewModel.errorMessage` 设了 10+ 处，UI 完全没渲染 → 出错用户看不见。
- 任何 `@Observable var` 添加后**立刻确认 View 层有对应的 UI 渲染**
- 错误类的状态推荐用 toast 显示（`ErrorToast` 已经做好）+ `didSet` 自动 3.5s 清空

### 11. codesign 报 "resource fork / Finder information not allowed"
原因：.app 内部有扩展属性（xattrs，如 `com.apple.FinderInfo`）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basionwang-bot/HermesPet](https://github.com/basionwang-bot/HermesPet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
