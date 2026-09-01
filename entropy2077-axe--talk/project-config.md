---
trigger: always_on
description: - 联系人创建协议现在会生成结构化 `pastExperiences`；保存到 `contactExperiences`，`kind='past'` 且进入长期记忆。经历可以同时关联多个 `contactIds`，因此AI之间的共同经历只保存一份共享事实，各参与者都能检索到。
---

# Talk — AI 聊天软件项目记忆

## 联系人经历系统
- 联系人创建协议现在会生成结构化 `pastExperiences`；保存到 `contactExperiences`，`kind='past'` 且进入长期记忆。经历可以同时关联多个 `contactIds`，因此AI之间的共同经历只保存一份共享事实，各参与者都能检索到。
- 用户超过一小时后再次发消息时，`chatEngine.ts` 调用 `experiences.ts`，用多功能模型结合自由人设、自然语言日程、地点、绑定世界书和合法关系候选补全离线经历。系统只校验时间、参与者、手机可用性和地点一致性，不写死职业活动。普通经历进入短期记忆，重要度达到70才进入长期记忆。
- 经历提示词遵循“属性词条 → `buildExperiencePromptSlice()` → 整体聊天提示词”；按过去正史、重要人生经历、最近生活分别限量，不做关键词检索。绑定世界书条目和底层世界观直接注入经历生成。
- 经历可由模型判断是否值得发朋友圈；朋友圈 `createdAt` 使用经历区间内的时间，而不是用户回来发消息的时间。
- 联系人 Beta 时间分支已经移除。数据库始终使用正式 `talk-db`；不要重新引入按 localStorage 切换整库的测试分支。
- 从世界书创建人物时先结构化提取既有关系、共同过往和过去经历，再交给人物生成；提取出的正史经历必须合并进创建初稿，并以 `sources=['worldbook']`、`sourceRefIds` 追踪来源。
- 添加联系人支持直接导入 SillyTavern V2/V3 JSON 或 PNG 角色卡；角色卡中的内嵌世界书一并导入和绑定，首条开场白保存为联系人会话的第一条 AI 消息。
- 联系人创建页会预览准备写入经历系统的过去经历；联系人名片也可以查看完整经历。
- 同一会话的消息入库统一经过 `nextMessageTimestamp()`，保证 AI 回复时间严格晚于触发它的用户消息，避免 PC 端相同毫秒排序反转。
- 工资不再由启动时后台结算：用户在“我”页面每天主动领取一次，领取时同时为全部在职联系人发放当日工资；工资交易使用按日期的幂等键，中断后重试会补齐未完成的联系人发薪。
- ComfyUI 支持导入 API Format JSON 自定义工作流；发送前只在克隆副本里注入正/负提示词、seed、采样参数和尺寸，不修改用户保存的原工作流。

## 项目定位
React 核心的仿微信风格 AI 对话应用。用户"添加联系人"（通过问卷让对方自动生成人设+名字，一次性确认，之后不能再改人设），
与 DeepSeek 模型进行拟人化聊天，并随聊天积累记忆和关系。内置待办/委托/货币/商城/仓库这套小游戏化系统，以及朋友圈系统（AI之间也有关系链、会互相点赞评论）。目标：安卓适配良好、PC 浏览器可直接调试、后续可选打包为原生 APK。

## 技术栈与关键决策
- **构建**: Vite + React + TypeScript。`vite.config.ts` 设置 `server.host: true`，方便手机在同一局域网通过 `http://<PC局域网IP>:5173` 直接访问调试。
- **样式**: Tailwind CSS v4（`@tailwindcss/vite` 插件，无需 tailwind.config.js，通过 `src/index.css` 里 `@import 'tailwindcss'` 引入）。整体白色简约风格。头像统一用圆角矩形（`Avatar` 组件默认 `rounded="lg"`）。
- **路由**: `react-router-dom`，使用 `HashRouter`（而非 BrowserRouter）—— 是为了以后 Capacitor 原生打包时用 `file://` 协议加载也不会有路由 404 问题。
- **状态管理**: Zustand，`useSettingsStore` 持久化到 localStorage（API Key、模型、说话风格提示词、用户昵称头像等资料、朋友圈封面图）。
- **本地数据库**: Dexie（IndexedDB 封装），`src/db/db.ts`，目前到 version(5)。表：`contacts`（人设+记忆+关系+朋友圈字段）、`conversations`、`messages`、`stickers`、`todos`、`commissions`、`inventory`、`moments`/`momentComments`/`momentLikes`、`contactRelations`。`locations`/`tasks` 两张表在 version(2) 加过、又在 version(3) 里用 `null` 显式删除了（Dexie删表的正确写法就是在新版本 `.stores()` 里把该表设为 `null`）——这是地图/日程功能被整体移除留下的痕迹，别奇怪。
- **安卓策略**: 响应式 Web（`.app-shell` 在 PC 端居中显示手机宽度的卡片，移动端全屏铺满），同时**已经真正打包成 Capacitor 原生 APK**（见下面"Capacitor 安卓打包"章节，不再是占位阶段）。**用户本地已装好 Android Studio，路径 `C:\Projects\AndroidStudio`**，Android SDK 在默认的 `%LOCALAPPDATA%\Android\Sdk`。
- **API Key 处理**: DeepSeek/Tavily/Pexels 三个 key 都写在根目录 `.env`，已加入 `.gitignore`。**打包APK发布前必须把 `.env` 里的key清空重新构建**，见下面章节——Vite 会把 `import.meta.env.VITE_*` 在构建时原样内联进编译后的JS里，APK本质是个zip包，解压就能看到明文key，绝对不能带着真key构建要对外分发的包。
- **独立路由页面的高度陷阱（踩过坑，别再犯，而且真的又犯过一次）**：不在 `TabLayout` 里的整页路由（`ChatPage`、`ContactAddPage`、`ProfileEditPage` 等，需要内容区滚动+底部按钮/输入框固定）**不能用 `min-h-full`**。`.app-shell` 只有 `min-height`，不是 `height`，导致子元素的百分比高度解析不确定，`flex-1` 拿不到真实剩余空间，底部固定栏会紧跟在内容后面而不是贴在可视区域底部。正确写法：根容器用 `h-[var(--app-height)] flex flex-col overflow-hidden`，中间滚动区用 `flex-1 overflow-y-auto`，底部栏保持普通flow（`--app-height`是什么、为什么不直接用`h-dvh`，见下面"Capacitor 安卓打包"章节里的踩坑记录）。**这条教训曾经只被应用到`ChatPage`/`ContactAddPage`/`GroupAddPage`/`GroupInfoPage`/`ProfileEditPage`/`SkyEyePage`这几个页面，但`ContactCardPage`/`MomentsPage`/`RelationshipsPage`/`SettingsPage`/`ShopPage`/`StickersPage`/`TodoPage`/`WarehousePage`/`WorldSettingsPage`这9个页面从一开始就没套用这个规则、一直用的是`min-h-full`**——这些页面早期内容比较少、恰好没超出屏幕高度，问题被长期掩盖，直到这个session陆续给这些页面加了很多新section（Tavily/Pexels设置、AI自主行为开关、管理员模式开关等）之后终于超出可视区域，`.app-shell`的`overflow:hidden`直接裁切掉超出部分且不可滚动触及——**用户反馈"测试连接按钮/AI自主行为开关/管理员模式开关都莫名其妙消失了"，一度怀疑是具体某个按钮的bug，用Playwright实测才确认是这个全局布局问题**（`.app-shell`实际渲染高度远超视口，且document级别虽然技术上能滚动露出内容，但整个"手机卡片"外壳会跟着一起滚走，体验上等于"看起来彻底消失了"）。已经把这9个页面全部统一改成`h-[var(--app-height)] flex-col overflow-hidden`+内部`flex-1 overflow-y-auto`。**以后新建任何独立整页路由，第一件事就是检查根容器是不是这个模式，不要抄`min-h-full`，TabLayout内部的5个tab页面（`MessagesPage`/`ContactsPage`/`DiscoverPage`/`MePage`）除外——那几个在`TabLayout`自己的`overflow-y-auto`包裹层里面，`min-h-full`在那种场景下是安全的。**
  - **这条"TabLayout内部安全"的结论后来被推翻了一半**：用户反馈`TodoPage`（TabLayout内部页面）待办事项一多，需要往下滑整个页面才能看到底部菜单栏——用Playwright真机同款复现（种40条待办）证实：`.app-shell`本身的渲染高度会超过视口（757.5px vs 700px视口），把`BottomNav`直接推到可视区域外面。根因是`.app-shell`自己只有`min-height: var(--app-height)`（这是原本为了"PC宽屏上手机卡片可以比视口高"这个场景设的），`min-height`允许元素被内容撑得比这个值更高——`TabLayout`的`Outlet`包裹层虽然自己有`flex-1 min-h-0 overflow-y-auto`，但它的"可用空间"是从父级(`.app-shell`)算出来的，父级本身如果没有硬性高度上限，这一整条"子级overflow-y-auto内部滚动"的链路就会连锁失效，变成父级跟着子级内容一起被撑高。**修复**：`.app-shell`的`min-height`改成`height`（含桌面端媒体查询里的那份），因为这个应用现在所有路由的内容都被设计成"在一个有边界的`flex-1 overflow-y-auto`区域内部滚动"，没有任何场景真的需要`.app-shell`自己比视口更高。**教训**：`min-h-full`只是这类问题的其中一种表现形式，真正的必要条件是"从叶子节点到`.app-shell`这条链路上，每一层都要有硬性高度上限（`height`而不是`min-height`），只要中间有任何一层是`min-height`，子级的`overflow-y-auto`就会失去意义"——以后如果同类"某个tab页面内容一多菜单栏就消失"的反馈再出现，先怀疑`.app-shell`本身或TabLayout链路上有没有类似的`min-height`遗漏，而不是只看具体页面自己的类名。

## Capacitor 安卓打包（`android/` 目录本身被 `.gitignore` 排除，不进仓库）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Entropy2077-axe/talk](https://github.com/Entropy2077-axe/talk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
