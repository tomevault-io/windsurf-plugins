---
trigger: always_on
description: 日语学习 App，交互参考 Duolingo。使用 React Web 技术栈，通过 Tauri v2 打包桌面端和 Android。
---

# 日学 (Benkyo AI) - AI 工程指南

日语学习 App，交互参考 Duolingo。使用 React Web 技术栈，通过 Tauri v2 打包桌面端和 Android。
支持 AI 生成个性化课程、闯关练习、练习中心、语法教程、单词本、每日任务、徽章/御守/道具收集、TTS 日语语音和 UI 音效。

本文件只保留 AI 快速理解代码所需的信息。实现细节以源码为准，修改前先读目标组件、store 和相邻工具函数，不要只凭本文假设行为。

---

## 技术栈

| 层级 | 技术 |
|------|------|
| UI | React 19 函数组件 + Hooks |
| 构建 | Vite 8，`base: './'` 适配 Tauri |
| 样式 | TailwindCSS v4，通过 `@tailwindcss/vite` 集成 |
| 动画 | GSAP 3 + `@gsap/react` |
| 状态 | Zustand 5 + persist |
| 路由 | React Router DOM 7，必须使用 `HashRouter` |
| AI | Vercel AI SDK 6，多 provider |
| Schema | Zod 4，主要作为结构参考 |
| 客户端 | Tauri v2，桌面端 + Android |

常用命令：`npm run dev`、`npm run lint`、`npm run build`、`npm run tauri:dev`、`npm run tauri:build`、`npx tauri android build --apk --split-per-abi`、`npm run android:release -- -KeystorePath .\android-signing\benkyo-ai-release.jks -KeyAlias benkyo-ai`。

不要扫描或编辑 `src-tauri/target/`、`src-tauri/gen/android/app/build/` 等构建产物。

---

## 目录地图

```text
src/
├── pages/
│   ├── HomePage.jsx                    首页章节地图
│   ├── LessonPage.jsx                  章节闯关路由入口
│   ├── VocabPage.jsx                   练习中心
│   ├── VocabBookPage.jsx               单词本详情
│   ├── ListeningPracticePage.jsx       听力练习
│   ├── CourseReviewPracticePage.jsx    课程巩固
│   ├── WordReviewPracticePage.jsx      单词复习
│   ├── WrongReviewPracticePage.jsx     错题重练
│   ├── GrammarPage.jsx                 语法教程
│   ├── ShopPage.jsx                    商店
│   ├── ProfilePage.jsx                 我的
│   ├── ProfileSetupPage.jsx            首次设置
│   └── SettingsPage.jsx                AI + TTS 设置
├── components/
│   ├── Layout/                         MainLayout / BottomNav
│   ├── Map/                            地图、章节横幅、课程生成弹层
│   ├── Lesson/                         通用闯关题型、反馈、结算、复活
│   ├── Practice/                       练习中心特殊玩法组件
│   ├── Shop/                           御守扭蛋与商店特殊组件
│   ├── Profile/                        编辑资料、头像裁剪、背包、徽章
│   └── UI/                             通用组件、悬浮组件、音频按钮
├── store/
│   ├── userStore.js                    用户、心心、金币、道具、XP/金币加成、御守收藏
│   ├── gameStore.js                    章节/通用练习闯关状态
│   ├── courseStore.js                  AI 生成课程
│   ├── badgeStore.js                   徽章解锁与累计进度
│   ├── dailyTaskStore.js               每日任务状态
│   ├── aiStore.js                      大模型配置
│   ├── ttsStore.js                     TTS 配置
│   ├── vocabStore.js                   单词本
│   ├── wrongQuestionStore.js           错题库
│   ├── appearanceStore.js              图标皮肤
│   ├── listeningPracticeStore.js       听力练习状态
│   ├── wordReviewPracticeStore.js      单词复习状态
│   ├── autoGenStore.js                 后台补齐关卡运行态
│   └── nextChapterGenStore.js          下一章节生成运行态
├── lib/
│   ├── generate-chapter.js             课程生成流水线
│   ├── course-wire.js                  AI JSON 传输协议与兼容解码
│   ├── badge-progress.js               徽章实时进度计算
│   ├── equipment-effects.js            护身符/装备实际特效入口
│   ├── *-practice.js                   练习中心抽题/构题工具
│   ├── giftbox-rewards.js              礼物盒掉落与开启奖励
│   ├── judge-answer.js                 AI 误判申诉
│   ├── tts.js                          TTS 请求与 IndexedDB 缓存
│   ├── japanese-speech-player.js       日语语音播放控制
│   ├── sound-effects.js                UI 音效类型和播放
│   └── schemas/course.js               课程 Zod 结构参考
└── data/                               静态示例、商店与御守数据
```

Android 自定义入口：`src-tauri/gen/android/app/src/main/java/com/benkyo/ai/MainActivity.kt`。

---

## 路由

```text
/                                  HomePage，MainLayout
/shop                              ShopPage，MainLayout
/vocab                             VocabPage，MainLayout，底部导航显示“练习”
/vocab/book                        VocabBookPage，MainLayout
/profile                           ProfilePage，MainLayout
/setup                             ProfileSetupPage
/lesson/:chapterId/:levelId        LessonPage
/practice/listening                ListeningPracticePage
/practice/course-review            CourseReviewPracticePage
/practice/word-review              WordReviewPracticePage
/practice/wrong-review             WrongReviewPracticePage
/grammar/:chapterId                GrammarPage
/settings                          SettingsPage
```

- `App.jsx` 使用 `HashRouter`，不要改为 `BrowserRouter`。
- `RequireProfile` 在 profile 为空时强制跳转 `/setup`。
- `AppInit` 启动时同步连续签到、心心、XP 加速和每日任务。
- `XpBoostWidget`、`SoundEffectProvider`、`DailyTaskToast` 在 `App.jsx` 全局渲染。
- 练习中心入口在 `VocabPage.jsx`；单词本内容已拆到 `VocabBookPage.jsx`。

---

## Zustand Store

| Store | 持久化 key | 核心职责 |
|-------|------------|----------|
| `userStore` | `benkyo-ai-user` | profile、连续天数、心心、金币、背包、装备状态、签到、XP/金币加成、学习档案、御守收藏 |
| `gameStore` | `benkyo-ai-progress` | 持久化 `levelProgress`、`totalXp`；临时保存章节闯关和通用练习 `lesson` |
| `courseStore` | `benkyo-ai-courses` | AI 生成的 `chapters` |
| `dailyTaskStore` | `benkyo-ai-daily-tasks` | 每日任务、完成 toast 队列、宝箱领取状态 |
| `badgeStore` | `benkyo-ai-badges` | 徽章解锁状态和累计计数 |
| `aiStore` | `benkyo-ai-ai-config` | provider、API Key、模型、Base URL、思考深度 |
| `ttsStore` | `benkyo-ai-tts-config` | TTS provider、API Key、模型、音色 |
| `vocabStore` | `benkyo-ai-vocab` | 单词本 |
| `wrongQuestionStore` | `benkyo-ai-wrong-questions` | 错题库，按章节+关卡+题目稳定去重 |
| `appearanceStore` | `benkyo-ai-appearance` | 当前图标皮肤，默认 `benkyochan` |
| `listeningPracticeStore` | 不持久化 | 听力练习特殊玩法状态、心心、失败/复活、结算 |
| `wordReviewPracticeStore` | 不持久化 | 单词复习特殊玩法状态、心心、失败/复活、结算 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [idiotbaka/benkyo-ai](https://github.com/idiotbaka/benkyo-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
