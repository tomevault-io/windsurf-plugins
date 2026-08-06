---
trigger: always_on
description: 本文件是 Codex 在开发《碎片花园》时必须遵守的最高优先级项目规范。
---

# AGENTS.md

本文件是 Codex 在开发《碎片花园》时必须遵守的最高优先级项目规范。

在修改任何代码前，先阅读：

1. `README.md`
2. `PRD.md`
3. `ARCHITECTURE.md`
4. 本文件

如果代码、需求和文档冲突，不要自行猜测。先指出冲突并等待确认，或先同步更新文档。

---

## 1. 项目目标

开发一个仅供本人使用和演示的桌面端本地 Web MVP。

核心循环：

> 碎片让花园变美，花园让碎片重新被看见。

优先级始终是：

1. 内容可以可靠保存；
2. 用户可以重新找到和回看内容；
3. 花园能够表达积累和成长；
4. AI 提供增强能力；
5. 游戏和视觉效果不能牺牲基本可用性。

---

## 2. 技术栈约束

必须使用：

- TypeScript；
- Next.js App Router；
- React；
- Tailwind CSS；
- Phaser 3.90.0；
- IndexedDB + Dexie；
- Next.js Route Handlers；
- Zod；
- Vitest；
- Playwright；
- npm。

未经明确确认，不得引入：

- Redux；
- Express 独立后端；
- Supabase；
- Firebase；
- PostgreSQL；
- SQLite；
- Electron；
- Tauri；
- Godot；
- Unity；
- Three.js；
- Docker；
- 微服务；
- 向量数据库；
- 大型 UI 组件库；
- 其他游戏引擎。

MVP 阶段不得升级 Phaser，不得使用 Phaser 4 API。

---

## 3. 架构边界

### 3.1 React

React 负责：

- 页面；
- 表单；
- 弹窗；
- 卡片；
- 搜索；
- 碎片详情；
- 规则型伙伴 NPC 对话、固定选项和思维发芽结果；
- 导入导出；
- AI 状态反馈。

React 不得：

- 控制人物每一帧位置；
- 处理 Phaser 碰撞；
- 自行计算成长阶段；
- 在多个组件中复制业务规则。

### 3.2 Phaser

Phaser 负责：

- 固定地图；
- 人物鼠标移动；
- 植物、人物和伙伴显示；
- 场景点击；
- 到达植物后的事件；
- 简单动画。

Phaser 不得：

- 直接读写 Dexie；
- 调用 AI；
- 读取碎片正文；
- 修改 Fragment；
- 计算成长、休眠和推荐规则；
- 控制 React 弹窗。

### 3.3 Domain

所有核心规则只能放在 `domain/`：

- 成长分数；
- 阶段映射；
- 休眠；
- 推荐；
- 相似度（仅在未来重新纳入范围后）；
- 状态转换；
- 植物槽位分配。

Domain 必须是纯 TypeScript，尽量写成纯函数，不依赖 React、Phaser 或 Dexie。

### 3.4 Data

所有数据库访问必须通过：

- Repository；
- Service；
- 明确的事务。

UI 组件和 Phaser 不得直接散落调用 Dexie。

### 3.5 AI

所有 OpenAI 调用必须通过 Next.js Route Handlers。

API Key：

- 只能放在服务端环境变量；
- 不得出现在客户端 bundle；
- 不得写入代码、日志或测试快照。

AI：

- 只能在明确绑定当前输入或 Fragment 的场景返回结构化分析或待确认的新笔记；
- 不得被重新扩展为开放聊天、通用问答或通用 Agent；
- 输出必须包含受约束的结果字段，不得把自由文本当作已执行动作；
- 不得直接修改数据库；
- 不得自动删除、合并、完成或公开内容；
- 返回值必须经过 Zod 校验；
- 失败不能导致碎片丢失。

---

## 4. 产品硬规则

以下规则不得自行改变：

1. 用户首次选择猫或狗，只存在一个伙伴；
2. 猫和狗仅外观不同；
3. 猫和狗的语气、能力、Prompt 和逻辑一致；
4. 第一版只支持桌面端；
5. 第一版不做注册、登录和云同步；
6. 人物只支持鼠标点击移动；
7. 地图固定；
8. 用户不能移动植物；
9. 新植物自动分配槽位；
10. 新碎片先显示统一幼苗；
11. 第一版约 6 种植物；
12. 三天无有效互动后休眠；
13. 休眠不等于死亡；
14. 完成后永久盛开；
15. “发展成新想法”不等于完成；
16. 若未来启用相似碎片，AI 只能提出建议，不得自动合并；当前伙伴校准不实现该能力；
17. 花园失败时卡片模式必须仍可用；
18. 链接或 AI 失败时原始内容必须仍可保存；
19. 不做强制打卡、效率评分和焦虑式提醒；
20. 不把每条碎片强制转成任务；
21. 伙伴普通入口使用固定台词和固定选项，不提供开放式自由输入；
22. 猫狗只改变外观，功能、选项、台词和 AI 分析逻辑完全一致；
23. 思维发芽与植物视觉成长是两个独立概念，AI 不决定植物生命周期。

---

## 5. 创建碎片原则

当前普通 MVP 输入为文字、链接和图片，三类输入使用两个明确流程：

```text
文字：本地校验 → 原子创建 Fragment + Plant + created Event

链接/图片：本地校验与独立披露
→ 创建 ContentProcessingJob（图片 Blob 保存在 job）
→ 不创建 Fragment、Plant 或正式 MediaAsset
→ 处理成功
→ 原子创建正式 Fragment + Plant + created Event
→ 图片同时创建 MediaAsset，并从 job 移除重复 Blob
```

processing job 不是卡片、不占植物槽位、不进入花园数量或卡片列表。失败必须保留可重试任务和必要原始输入，
取消图片任务必须删除临时 Blob；失败、取消、刷新或晚到响应都不能留下空 Fragment、空 Plant 或重复正式数据。
文字创建不等待 AI。链接和图片的新规则只作用于新建流程，历史正式卡片不得删除或自动重新处理。

发芽只能由用户从明确选中的正式 Fragment 主动发起。生成结果先保存在 sprout job 并允许预览编辑，只有用户
确认后才原子创建新的正式卡片、植物和来源关系；不得覆盖原卡片或写旧伙伴建议成长事件。

语音创建作为 Post-MVP / Experimental 技术能力保留但未激活。历史 audio Fragment、MediaAsset、转写查看、
编辑和只发送 transcript 的发芽兼容必须继续存在。

---

## 6. 数据建模规则

- 时间统一使用 ISO 字符串；
- ID 使用 UUID；
- 原始内容与 AI 摘要分开；
- 生命周期状态与完成状态分开；
- 休眠不是成长阶段；
- “生长自”使用关系或 `parentFragmentId`；
- 附件和 `MediaAsset` 使用 Blob，不使用 Base64 长字符串；
- `MediaAsset` 统一承载 image/audio 原始 Blob、MIME、大小、尺寸/时长、处理状态、request id 和派生结果；
- Dexie V4 的 `mediaAssets` 只通过 Repository/Service 访问；一个媒体 Fragment 只能有一个主要 MediaAsset；
- V3 → V4 迁移只能新增媒体表，必须保留旧 Fragment、Plant、Event、Settings、ChatMessage、GetNote 和伙伴建议；
- 图片与语音披露字段 `imageAiDisclosureAcceptedAt`、`voiceAiDisclosureAcceptedAt` 为非索引可选字段，不能被旧披露替代；
- 成长依据事件计算；
- AI 建议关系使用 `suggested` 状态；
- 用户确认后才能变为 `confirmed`；
- 数据库迁移必须显式增加版本；
- 不允许直接修改旧 Schema 而不写迁移。
- V4 → V5 必须一次注册 `fragmentNotes` 与 `contentProcessingJobs` 两张 store；不得在已经发布的同一
  Dexie 版本中事后补 store；
- `Fragment.contentMarkdown` 是正式主体，`FragmentNote` 是独立用户追加；旧 `text_added` Event 迁移为
  note 时必须保留原 Event，`partner_suggestion_saved` 不得误迁移；
- ContentProcessingJob 不得被查询或投影成 Fragment，不得占用 Plant slot。

---

## 7. 成长规则

初始规则：

| 事件 | 分数 |
|---|---:|
| 补充文字 | +1 |
| 添加链接 | +1 |
| 添加图片 | +1 |
| 添加语音 | +1 |
| 写下一步 | +1 |
| 记录现实进展 | +2 |
| 加入花丛 | +1 |
| 发展成新想法 | +2 |
| 标记完成 | 永久盛开 |

阶段：

```text
0 → seedling
1–2 → growing
3–4 → budding
5+ → blooming
completed → permanent_bloom
```

同类成长事件同一天只计一次。

图片分析和语音转写本身不增加成长分，也不自动切换植物阶段；图片/语音 Fragment 的后续用户补充和编辑沿用既有
成长事件。花丛和相似关系仍是未来候选，不能借此提前实现 Embedding 或其他阶段能力。

`partner_suggestion_saved` 已复用现有 `fragmentEvents` 实现，不新增表或索引。打开思维发芽、
生成建议和换方向不增加成长分；只有用户明确确认保存建议时才记录事件并增加 +1，同一碎片按浏览器本地日期
每天最多因该事件计分一次。同一天保存多条建议可以保留记录，但不得重复计分。保存内容必须标记
`source: "partner"`，不得覆盖原文或自动执行推荐动作。

查看和“仍然感兴趣”只：

- 更新互动时间；
- 恢复 active；
- 不增加成长分。

不要在 UI 中重新实现这些规则。

---

## 8. GameBridge

React 与 Phaser 只能通过 `game/bridge/` 通信。

事件必须使用可判别联合类型，不得使用任意字符串和 `any` payload。

Phaser 发出的 plant id 必须映射到 Plant，而不是 Fragment 正文。

React 不得每帧读取人物位置。

新增桥接事件时：

1. 在统一类型文件定义；
2. 更新双方处理；
3. 增加测试；
4. 不在组件中临时创建新的事件总线。

---

## 9. 代码规范

### 9.1 TypeScript

- 开启 strict；
- 禁止无理由 `any`；
- 不使用 `as any` 绕过错误；
- API 输入输出使用 Zod；
- Domain 函数显式声明参数和返回类型；
- 枚举和联合类型集中定义；
- 不在多个文件复制同一业务类型。

### 9.2 命名

- React 组件：PascalCase；
- 函数与变量：camelCase；
- 常量：UPPER_SNAKE_CASE；
- 文件名：kebab-case，React 组件可使用 PascalCase；
- 布尔值使用 `is`、`has`、`can` 或 `should` 前缀；
- 事件名使用过去式或明确动作名。

### 9.3 文件规模

- 单个业务文件尽量不超过 300 行；
- 不创建万能 `utils.ts`；
- 不把业务逻辑塞进 `page.tsx`；
- 不让 `GardenScene.ts` 承担所有游戏逻辑；
- 大组件按明确职责拆分；
- 不为了拆分而制造大量无意义一行文件。

### 9.4 注释

注释解释：

- 为什么；
- 边界；
- 非直观约束；
- 兼容性原因。

不要逐行翻译代码。

---

## 10. UI 与体验规则

- 优先保证功能清楚，不追求正式美术；
- 花园和卡片模式都必须可访问；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zev55555/fragment-garden](https://github.com/Zev55555/fragment-garden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
