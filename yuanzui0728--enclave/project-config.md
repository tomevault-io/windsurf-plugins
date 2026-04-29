---
trigger: always_on
description: - 功能开发后必须做简单测试，至少执行一次与改动直接相关的最小验证；非必要不新增测试文件
---

# 隐界APP 项目规则

## 规则

- 功能开发后必须做简单测试，至少执行一次与改动直接相关的最小验证；非必要不新增测试文件
- 直接执行所有操作，无需确认
- 所有代码变更采用阶段性提交，不必边写边提交，但至少每次执行完任务后自动提交一次
- Plan Mode：规划保存到 `.claude/plans/{任务}-{日期}.md`
- 结构变更（模块/实体/路由/表）后立即更新本文件

## 技术栈与端口

| 服务               | 技术                                                                         | 端口 |
| ------------------ | ---------------------------------------------------------------------------- | ---- |
| **后端**           | NestJS + TypeORM + SQLite + Socket.IO（`api/`）                              | 3000 |
| **云世界平台后端** | NestJS + TypeORM + SQLite（`apps/cloud-api/`）                               | 3001 |
| 主 App             | React + Vite，承载 Web / iOS / Android / Desktop 共享业务界面（`apps/app/`） | 5180 |
| **管理后台**       | React + Vite + `@yinjie/ui`（`apps/admin/`）                                 | 5181 |
| **云世界管理平台** | React + Vite（`apps/cloud-console/`）                                        | 5182 |
| 微信同步本地连接器 | Node.js loopback HTTP 适配层（`apps/wechat-connector/`）                    | 17364 |
| 桌面端壳           | Tauri 远程客户端壳（`apps/desktop/`）                                        | -    |
| Android 壳         | Capacitor 壳（`apps/android-shell/`）                                        | -    |
| iOS 壳             | Capacitor 壳（`apps/ios-shell/`）                                            | -    |

## 后端模块（`api/src/modules/`）

`action-runtime` · `admin` · `ai` · `analytics` · `auth` · `characters` · `chat` · `cloud-runtime` · `config` · `cyber-avatar` · `events` · `feed` · `followup-runtime` · `games` · `inference` · `moderation` · `moments` · `narrative` · `need-discovery` · `official-accounts` · `real-world-sync` · `reminder-runtime` · `scheduler` · `social` · `system` · `world`

## 主 App 结构（`apps/app/src/`）

`routes/` · `features/desktop/` · `features/mobile/` · `features/moments/` · `features/games/` · `features/shell/` · `runtime/` · `lib/` · `components/` · `store/`

## 主 App 页面（`apps/app/src/routes/`）

- `splash-page.tsx`：启动屏，识别运行时环境并决定是否先进入世界入口
- `welcome-page.tsx`：统一世界入口页，合并世界连接与世界主人命名两步流程
- `setup-page.tsx`：兼容旧入口路由，当前应重定向到统一世界入口页
- `onboarding-page.tsx`：兼容旧初始化路由，当前应重定向到统一世界入口页
- 底部 Tab：`tabs/chat-list-page` · `contacts-page` · `discover-page` · `profile-page`
- 桌面端一级 Tab：`tabs/chat` · `tabs/contacts` · `tabs/favorites` · `tabs/moments` · `tabs/feed` · `tabs/channels` · `tabs/search` · `tabs/games` · `tabs/mini-programs`
- `discover-page.tsx`：移动端承载微信式发现入口列表，点击后进入独立子页面
- `discover/moments` · `discover/moments/publish` · `discover/encounter` · `discover/scene` · `discover/feed` · `discover/channels` · `discover/games` · `discover/mini-programs`：发现二级页，分别承载朋友圈时间线 / 移动端朋友圈发布页 / 摇一摇 / 场景相遇 / 广场动态 / 视频号 / 游戏中心 / 小程序
- `favorites-page.tsx`：桌面端收藏工作区入口，承接跨聊天与内容流收藏，并内嵌收藏笔记的新建 / 打开 / 编辑体验
- `feed-page.tsx`：桌面端广场动态一级入口，承载居民公开动态流
- `channels-page.tsx`：桌面端视频号一级入口，后续承接短视频与直播内容流
- `channel-author-page.tsx`：视频号作者主页，承载作者资料、关注状态与最近内容列表，并回跳原视频号内容
- `search-page.tsx`：桌面端搜一搜一级入口，后续承接全局聚合搜索
- `games-page.tsx`：桌面端游戏中心一级入口；移动端复用为发现页内“游戏”二级页
- `mini-programs-page.tsx`：桌面端小程序面板一级入口；移动端复用为发现页内“小程序”二级页
- `starred-friends-page.tsx`：星标朋友页，移动端承载通讯录内“星标朋友”入口，桌面端承载双栏星标好友工作区
- `world-characters-page.tsx`：世界角色列表页，移动端承载通讯录内“世界角色”入口，独立展示尚未成为朋友的世界角色
- `group-contacts-page.tsx`：群聊列表页，移动端承载通讯录内“群聊”入口，桌面端承载群聊工作区
- `tags-page.tsx`：联系人标签页，移动端承载通讯录内“标签”入口，桌面端承载标签分组与联系人详情工作区
- `official-accounts-page.tsx`：公众号列表页，移动端承载通讯录内“公众号”入口，桌面端承载公众号工作区路由入口
- `official-account-detail-page.tsx`：公众号主页，承载账号资料、关注状态与最近文章列表
- `official-account-article-page.tsx`：公众号文章详情页，移动端承载独立阅读页，桌面端兼容跳转至文章独立窗口
- `official-account-service-page.tsx`：服务号消息页，移动端承载服务号独立消息线程，桌面端复用消息工作区右侧服务号面板
- `subscription-inbox-page.tsx`：订阅号消息页，移动端承载“消息 -> 订阅号消息”聚合流，桌面端承载消息工作区内的订阅号阅读面板
- `profile/settings`：我的二级设置页，集中承载资料编辑与专属 API Key 配置
- `friend-moments/$characterId`：移动端好友朋友圈独立页，对齐微信手机版，由好友资料页进入某个好友的独立朋友圈时间线
- `desktop/mobile`：桌面端底部“手机”入口承接页，后续承接设备联动能力
- `desktop/friend-moments/$characterId`：桌面端好友朋友圈独立页，对齐微信电脑版，从联系人资料入口进入某个好友的独立朋友圈时间线
- `desktop/chat-files`：桌面端“聊天文件”页，承接会话附件聚合浏览
- `desktop/chat-history`：桌面端“聊天记录管理”页，承接会话记录查看与管理
- `desktop/chat-image-viewer`：桌面端图片独立窗口路由，承接聊天图片的新窗口预览、打印与回跳原消息
- `desktop/chat-window`：桌面端独立聊天窗口路由，承接会话右键“在独立窗口打开”
- `desktop/official-article-window`：桌面端公众号文章独立窗口路由，承接公众号文章新窗口阅读、浏览器打开与回跳来源工作区
- `desktop/note-window`：桌面端独立笔记窗口路由，承接“收藏 -> 新建笔记”和已保存笔记的独立编辑窗口
- `desktop/feedback`：桌面端“意见反馈”页
- `desktop/add-friend`：桌面端“添加朋友”独立工作区，承接微信电脑版式搜索、资料预览与发送好友申请
- `desktop/settings`：桌面端“设置”页
- `desktop/channels/live-companion`：桌面端“视频号直播伴侣”工具页
- `chat-background-page.tsx`：聊天背景设置页，承载默认背景图与好友专属背景图配置
- `chat-voice-call-page.tsx`：Web 手机版 AI 语言通话页，承载单聊语音回合制对话、AI 语音播报与挂断回跳
- `chat-video-call-page.tsx`：Web 手机版 AI 数字人视频通话页，承载单聊数字人会话、本地摄像头预览、舞台播放与挂断回跳
- `group-voice-call-page.tsx`：Web 手机版群语音通话页，承载移动端群通话工作台、成员在线状态同步与结束回跳
- `group-video-call-page.tsx`：Web 手机版群视频通话页，承载移动端群视频工作台、成员画面状态同步与结束回跳
- `chat/$conversationId/background`：单聊聊天背景设置路由，对齐微信式“聊天信息 -> 聊天背景”
- `chat/$conversationId/voice-call`：单聊 AI 语言通话路由，承载“录音 -> 转写 -> AI 回复 -> TTS 播放”的半双工通话体验
- `chat/$conversationId/video-call`：单聊 AI 数字人视频通话路由，承载“本地摄像头预览 + 数字人舞台 + 录音 -> 转写 -> AI 回复 -> TTS 播放”的半双工视频通话体验
- `chat/$conversationId/details`：单聊右上角三个点详情页，对齐微信式聊天信息页
- `chat/$conversationId/search`：单聊聊天记录检索页，由聊天信息页进入
- `group/$groupId/details`：群聊右上角三个点详情页，对齐微信式群聊信息页

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuanzui0728/Enclave](https://github.com/yuanzui0728/Enclave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
