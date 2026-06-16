---
trigger: always_on
description: EchoChat 项目上下文与开发记忆 - 每次新对话自动加载
---


# EchoChat 项目上下文

## 快速恢复上下文

**新对话开始时，必须先读取以下文件恢复项目记忆：**

1. `docs/progress/CURRENT_STATUS.md` — 项目开发进度、已完成 Task、关键技术决策、下一步工作
2. 当前阶段的实施计划文档（位于 `docs/plans/` 目录下）

## 当前进度

- **Phase 1（基础设施与用户认证）**：✅ 全部完成（11 个 Task）
- **Phase 2a（WebSocket 实时通讯与联系人管理）**：✅ 全部完成（13 个 Task + 后期 Bug 修复 3 项）
- **Phase 2b（即时通讯消息系统）**：✅ 全部完成（10 个 Task + 代码审查修复 7 项 + 用户测试修复 8 项），设计文档 `docs/plans/2026-03-03-phase2b-design.md`
- 分支：`feature/phase2b-instant-messaging`
- **Phase 2c（群聊与已读回执）**：✅ 全部完成（14 个 Task + 代码审查修复 14 项 + 浏览器测试修复 21 项），设计文档 `docs/plans/2026-03-04-phase2c-design.md`
- 分支：`feature/phase2c-group-read-receipt`
- **Phase 2d（消息类型扩展）**：✅ 全部完成（14 个 Task + 代码审查修复 2 项 + 富媒体 Bug 修复 4 项 + UX 优化 1 项），设计文档 `docs/plans/2026-03-04-phase2d-design.md`
- **⚠️ 分支说明**：Phase 2d 的实际开发工作误用了 `feature/phase2c-group-read-receipt` 分支，`feature/phase2d-message-types` 分支未承载 2d 代码。此次偏差不做回溯修复；**Phase 2e 起必须为每个阶段新建独立分支**（`feature/phase2e-xxx`）。
- **Phase 2e（会议与通知系统）**：🚧 规划完成，拆分为三子阶段，设计文档 `docs/plans/2026-04-20-phase2e-design.md`
  - 2e-1 通知系统（3-4 天）✅ **已完成**：统一通知中心 + 11 种类型 + Pusher 接口注入 + 30 天清理 + 管理员广播 + 前端 5 分类 Tab
    * 单端 WS 连接架构（沿用），不做多端已读同步（设计文档 §3.1/§3.5/§九 已修订，多端改造推迟到 Phase 2f/二期）
    * 专用设计：`docs/plans/2026-04-20-phase2e-1-design.md`；实施计划：`docs/plans/2026-04-20-phase2e-1-implementation.plan.md`；验证报告：`test-report-phase2e-1-notification.md`
    * API 文档：`docs/api/frontend/notify.md`
  - 2e-2 会议 MVP（10-14 天）📋 待开发：mediasoup Node.js 独立服务 + 即时会议（≤8 人）+ 音视频控制
  - 2e-3 会议增强（7-10 天）📋 待开发：预约会议 + 定时提醒 + 会议邀请（复用 2e-1）
  - 分支：`feature/phase2e-meeting-notification`（基于 `origin/feature/phase2c-group-read-receipt`）
  - **关键技术锁定**：维持 mediasoup SFU 架构（非 Mesh），前端用 mediasoup-client，信令复用现有 WS Hub
  - **显式推迟清单**（必须留档，见设计文档 §九）：
    * → Phase 2f：WS Hub 多端连接改造、会议管理后台、通知广播发布 UI、管理端仪表板、操作日志页、系统配置管理、通知分类开关
    * → 第二期：屏幕共享、会议录制、虚拟背景、微信登录、互动直播、视频消息、表情包、消息转发/引用
    * → 第三期：微服务拆分、K8s、跨服 Worker 集群、AI 语音转文字
- **开发者运维脚本（2026-04-20）**：新增 `scripts/{start,stop,status}.sh` 三件套，`scripts/dev-setup.sh` 补齐 MinIO 健康检查；遵循 setup/start 职责分离（Rails/Django 社区模式）
- **聊天页 UX 优化（2026-04-20）**：单聊/群聊引入「新消息悬浮提示 + 按需已读」机制（贴底自动滚+已读，远离底部显示"N 条新消息"悬浮按钮），消除"视图未见但对方已读"的 UX 错位；**后续修复**：watch 误将"加载更多历史消息"计入 newMsgCount → 改为对比「末尾消息标识（tail id/client_msg_id）」而非 `messages.length`，真实双账号（duanlingyun ↔ bojinyuan）端到端验证通过
- **语音消息 H5 兼容（2026-04-20）**：修复 PC Chrome 无法按住"按住说话"的问题，链路涉及 4 层（事件 + 录音 API + 上传 + 后端校验）：VoiceRecorder 并行监听 touch/mouse 事件；新增 `H5Recorder` 类（基于 `MediaRecorder + getUserMedia`）作为 uni 录音接口在 H5 端的回退；`uploadVoice` 增加 blob 分支走 `fetch+FormData` 精确控制 filename；后端 `allowedVoiceExts` 扩展 `.webm/.ogg`。Playwright 端到端验证：录音计时正常（3s→5s 跳动）、Blob 上传成功、数据库 `im_messages` type=3 记录写入
- **语音未听红点（2026-04-20，仿微信）**：语音需"听"才算真消费，仅通用已读不能准确反映 → 为「对方发来的」语音叠加独立状态。`chat store` 新增 `voicePlayedMap` + 4 API（`markVoicePlayed/isVoicePlayed/loadVoicePlayedState/resetVoicePlayedState`），localStorage 按 userId 隔离（`echo:voice-played:{userId}`），不同步后端（私人视图态）；`MsgVoice.vue` 自主消费 store（单聊/群聊同构），模板加 `.unplayed-dot`（10rpx 红圆），`onTogglePlay` 点击即标记清除；`user.logout()` 补 `resetVoicePlayedState()` 防串用户。Playwright 验证：duanlingyun(13) ↔ bojinyuan(7) 会话 6，初始 5 个红点（对方 5 条语音） → 点击 2 条 → 红点减至 3 → 页面刷新仍为 3（持久化生效）
- **已读状态刷新持久化（2026-04-20）**：修复"刷新后所有已读标签变未读 / 群聊 N人已读消失"的体验倒退 bug，根因是前端 `readStatusMap`/`groupReadCountMap` 仅由 WS 事件填充、刷新后无 API 补回。扩展 `HistoryMessageResponse` 新增 `peer_last_read_msg_id`（单聊）+ `read_count_map`（群聊仅含自己发送消息）；`GetHistoryMessages` 按会话类型分支填充（`GetPeerUserID`+`GetMember` 或 `readRecorder.GetReadCountBatch`）；前端 `loadHistoryMessages` 仅在首次加载时回填（避免加载更多时覆盖 WS 增量）。Playwright 双端验证：单聊会话 6（peer_last_read=202）首屏 20 已读+1 未读、群聊 8 刷新后 13 条自己消息全部显示"N人已读"（11×1人 + 2×2人）；不新增 API，不增加请求次数，向前兼容
- 范围：图片/语音/文件消息完整流程 + 管理端消息管理（列表+统计+撤回+删除）+ ECharts 仪表板
- **跨模块通信模式**：接口注入标准（ws.FriendIDsGetter / im.FriendChecker / im.UserInfoGetter / notify.UserInfoResolver → contact.FriendshipDAO，im.OfflineMessagePusher → ws.Handler，contact.OnlineChecker → ws.OnlineService，im.GroupInfoGetter → group.GroupDAO，im.MessageReadRecorder → group.MessageReadDAO，group.UserInfoProvider → auth.UserDAO，group.MessageWriter → im.MessageDAO，admin.MessageManageService → im.ConversationDAO + ws.PubSub，**contact.NotifyPusher / group.NotifyPusher → notify.NotifyService**，**ws.NotifyConnectHook → notify.NotifyService**）

## 项目概述

EchoChat 是一个实时音视频通讯平台，包含三个子项目：
- `backend/go-service/` — Go 后端（Gin + GORM + Wire + Redis + gorilla/websocket）
- `frontend/` — 前台用户端（uni-app + Vue 3.4 + Pinia 2.x）
- `admin/` — 后台管理端（Vue 3.5+ + Element Plus + Pinia 3.x）

已实现模块：auth（认证）、contact（联系人）、ws（WebSocket）、im（即时通讯）、group（群聊）、file（文件上传）、admin（管理端含消息管理）、im（即时通讯 + 已读回执）、group（群聊管理）、file（文件上传/MinIO）
前端常量：`frontend/src/constants/group.js`（GROUP_ROLE / GROUP_STATUS / JOIN_REQUEST_STATUS，与后端 constants/group.go 对齐）

## 核心开发规则

1. **前端设计**：必须使用 openskills 安装的 `ui-ux-pro-max` 技能包，脚本绝对路径为 `/Users/bojinyuan/.agent/skills/ui-ux-pro-max/scripts/search.py`。**严禁使用** `.cursor/skills/ui-ux-pro-max.bak/` 目录下的任何文件，该目录已废弃。禁止手动设计系统
2. **工作流**：使用 superpowers 流程控制开发节奏
3. **两端差异**：`frontend/` 和 `admin/` 是完全独立的项目，技术栈不需要统一
4. **模块系统**：前端统一使用 ESM（`export`/`import`），禁止 CommonJS
5. **Go 常量命名**：camelCase（`UserStatusActive`），非大写下划线
6. **API 响应**：统一 `{ "code": 0, "message": "success", "data": ... }`
7. **JWT 策略**：有状态 JWT，Token 存 Redis（按 clientType 隔离：`echo:auth:token:{frontend|admin}:{user_id}`），前后台互不影响

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bojinyuan00/EchoChat](https://github.com/bojinyuan00/EchoChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
