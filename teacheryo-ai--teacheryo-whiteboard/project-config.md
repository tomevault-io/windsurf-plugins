---
trigger: always_on
description: > 这份文件是给**要动手改这个项目的 AI 助手**看的（Claude Code / Cursor / Codex / WorkBuddy 都适用）。
---

# AGENTS.md — 项目改动手册

> 这份文件是给**要动手改这个项目的 AI 助手**看的（Claude Code / Cursor / Codex / WorkBuddy 都适用）。
> 只是想把项目跑起来用 → 看 [`README.md`](README.md)；
> 想让 AI 帮你上手使用 → 看 [`AI-PROMPT.md`](AI-PROMPT.md)；
> **要改代码** → 就是本文件，请先完整读完再动手。

---

## 一句话定位

**零构建、可自托管的课堂互动工具。** 纯 HTML / CSS / 原生 JS，无 npm、无打包步骤、
无框架；数据层双模式（浏览器 localStorage ↔ Cloudflare Worker API + D1）。

技术栈就这么简单，所以**任何"现代化改造"（给前端引入构建工具、框架、npm 依赖）都视为破坏**。
唯一的例外：`server/` 目录（数据 API Worker）允许用 npm / TypeScript / Drizzle——
那是部署工具链，不属于前端页面；**前端仍然双击 `index.html` 就能跑**。

---

## 硬约束（违反后会静默坏掉，不会有报错）

### 1. 零构建不可破坏

前端（`index.html` / `student.html` / `js/` / `css/`）不引入 npm / webpack / vite /
TypeScript / 任何构建工具或运行时依赖。必须始终保持**双击 `index.html` 就能跑**。
构建工具只允许出现在 `server/`（数据 API Worker）里。

### 2. 数据层是双模式的 —— 加接口必须两边都加

`js/db.js` 启动时探测 `config.js` 的 `api` 字段：

```
api 留空  →  挂 window.TY_LOCAL（js/store.local.js，localStorage）
api 有值  →  挂 window.TY_API（js/store.api.js，fetch 到 teacheryo-api Worker + D1）
```

两套实现**必须同签名**，返回字段统一 snake_case
（`course_id` / `created_at` / `liked_by` / `part_idx`）。
**只改一边 → 本地模式会静默失效**（不报错，功能直接没有）。

数据 API Worker（`server/`，Cloudflare D1）三张表与接口语义：
- 表：`courses` / `boards` / `submissions`（`server/src/schema.ts`，JSON 列用 TEXT 存 JSON）
- uid：浏览器 localStorage（`ty.api.uid`）生成，每次请求经 `x-ty-uid` 头带上；
  `toggleLike` 在 Worker 里用**原子 SQL**（条件 UPDATE + json 函数），
  不要改回「读-改-写」（并发点赞会丢计数）
- CORS：`server/wrangler.jsonc` 的 `ALLOWED_ORIGINS` 白名单（逗号分隔），
  新增部署域名必须加进去，否则浏览器端表现为 `TypeError: Failed to fetch`

现有接口：`listCourses` `getCourse` `createCourse` `updateCourse` `deleteCourse`
`listBoards` `getBoard` `getBoardByCode` `createBoard` `updateBoard` `deleteBoard`
`moveBoard` `archiveBoard` `reopenBoard`
`listSubmissions` `submit` `toggleLike` `removeSubmission` `updateSubmission`

### 3. 便签墙用世界坐标，不要退回归一化

便签位置存 `data.px` / `data.py`（世界像素，**可为负**），画布 24000×24000，
`NC_ORIGIN = 12000` 为原点偏移。旧数据 `data.x/y`（归一化 0-1）渲染时自动换算，
一拖动即升级为新格式。

**不要引入任何「归一化 + 固定画布」的写法** —— 那正是当初"假无限白板"（缩小后框外贴不进去）的根因。

### 4. 会被周期性调用的恢复逻辑必须幂等

页面每 2.5 秒轮询重绘一次。像 `toggleNoteFull` 这种"重绘后恢复 UI 状态"的函数
**只会被反复调用**，因此只能同步 UI，**绝不能重复做尺寸换算 + 重定位**——
否则每轮固定偏移，表现为画面持续漂移。

判定口诀：**如果画面自己在动，先问是不是周期任务在重复一次不该重复的转换。**
（历史上"白板一直往右下角弹"就是这么来的。）

### 5. `TY.noteUiBusy()` 是轮询的刹车

白板内拖拽、缩放、老师面板打开、输入框聚焦时，它必须返回 `true`，
否则 2.5s 轮询重绘会**清空老师正在写的内容**。新增任何输入 UI 都要接进这个判断。

⚠️ **学生端目前没有接这个刹车**，而是靠"输入区根本没被轮询重建"侥幸安全：
`refreshBoard()` 里重调 `bindInput()` 的分支**只对 `material` 类型生效**，
`notes` 类型的输入区从头到尾没被重绘过。
所以学生端图片状态刻意存在 **`S.imgData`（JS 对象）而不是 DOM 里**，
并用**幂等**的 `S.setImgPreview()` 回填 ——
**如果哪天给 `notes` 也加上轮询重绘，必须保留这个"状态在对象上 + 幂等回填"的结构**，
否则学生正在选的图片会在 3 秒后凭空消失，而且不报任何错。
（详细复盘见 skill 的 `references/pitfalls.md` 坑 20。）

### 6. 凭据永不进仓库

真实 `envId` / `accessKey` 只写 `config.local.js`（已在 `.gitignore`）。
公开的 `config.js` 永远保持空模板。

推代码前自查：

```bash
git grep --cached -I -e "<envId>" -e "<accessKey>"
```

### 7. 脚本加载顺序不能调换

`vendor/cloudbase.full.js` → `config.js` → `config.local.js`
→ `js/store.local.js` → `js/common.js` → `js/db.js`

### 8. 随机点名：抽中的人 / 分组结果只能存在 `App.rc` 上

**任何"当场算出来的结果"都必须先落到 JS 状态对象，再由渲染函数从状态重画。**

`App.rc` = `{ bid, picked, current, rolling, groups, groupMode, groupN, noRepeat }`。
`rcStageHTML()` / `rcGroupsHTML()` 每轮都从它**幂等重画**，绝不从上一轮 DOM 里抄。

为什么（这是本文件里最容易被忘记的一条）：
`renderBoardBody()` 每 2.5 秒被整块重建一次（`body.innerHTML = ...`），
**只写进 DOM 的东西一定会被冲掉**，而且不报任何错 —— 表现就是
「刚抽出来的人 3 秒后自己消失 / 分组结果闪一下就没了」，老师会以为是玄学。

两条配套刹车：

- `refreshBoard(silent)` 开头有 `if (silent && this.rc && this.rc.rolling) return;`
  —— **抽名动画（约 1.7s）期间让轮询直接跳过**，否则动画会被重绘打断。
- `rollCall()` 的动画 step 里判断 `el && el.isConnected`，舞台被换掉就**安静收工、不动状态**，
  不要抛错、也不要把 `rolling` 卡在 `true`（卡住会让轮询永远停摆）。

同理，学生端「填名字」输入区也**只在"报名状态真的翻转"时才 `bindInput()`**，
否则 3 秒轮询会清空学生正在打的名字（见约束 5 的学生端部分）。

### 9. 学生端「交没交」决定哪块 UI 长什么样 —— 轮询只能补，不能覆盖

学生端页面上有**两份看起来一样的东西**（结果区 + 作答区），所以"交没交"必须
同时管住两边，否则学生在手机上看到上下各一份选项，不知道该点哪（2026-09-17 用户反馈）。

- **选择题没交之前只留一份可点的选项**：`renderAllUnits()` 里
  `locked = (u.subType === 'choice') && !this.isSubmitted(i)` → 结果区换成 `S.lockHTML()`，
  卡片加 `.locked`（虚线、无底、隐藏计数）。提交后自动解锁并画出结果。
- **`S.editing`（改选）期间，轮询绝对不能重建输入区** —— 判据是
  `refreshBoard()` 里那句 `if (this.isSubmitted(0) && !this.editing)`。
  少了 `&& !this.editing`，学生正在重选的选择会被 3 秒轮询清掉。
  （同约束 4 的"画面自己在动"，只是换成"选项自己变回去"。）
- **提交后跳转**统一走 `S.jumpToResults(partIdx)`：`scrollIntoView` + `.flash` 高亮。
  它只在提交成功那一刻调用，**不要放进轮询**，否则学生会一直被拽着滚。
- 结果的「我选的那项」标记走 uid：`myRow(partIdx)` → `myPicked(partIdx)` →
  `renderChoice(..., { mySel })` 打 `.mine` + `你的选择` 徽章。
  老师端不传 `mySel`，所以大屏渲染与以前完全一致（common.js 的渲染器加了可选参数，
  不要改成必填）。
- 重新提交 = 先 `removeSubmission(myRow(0).id)` 再 `submit`，**不能只 submit**，
  否则同一题会记两票（票数会凭空多出来）。这一句**故意不加 `if (editing)` 守卫**：
  学生清了浏览器数据后本地「已提交」标记会丢，但库里的行还在，再交一次就会
  让同一个人在大屏上出现两次（实测过：`ty_subbed_*` 被清掉再交，加守卫时 2 行 / 共 6 人，
  去掉守卫后 1 行 / 共 5 人）。

### 10. 从表单读值必须读 `.value` / `.checked`，**不能 `!!getElementById(id)`**

```js
// ❌ 元素对象永远 truthy → multi 恒为 true
const multi = !!document.getElementById('cfg-multi');
// ✅
const multi = !!(document.getElementById('cfg-multi') || {}).checked;
```

这条是**真实事故**（2026-09-17 用户反馈「我没勾多选，结果里却写着（多选）」）：
`createBoard()` 里写成 `!!document.getElementById('cfg-multi')`，于是**只要建选择题就恒存
`config.multi = true`** —— 大屏每个选项都挂「（多选）」，学生端也放开了多选。
**不勾 = 单选**，这是产品约定。

- 同类写法在教师端多处出现（`v(id)` 取 `.value` 那套是对的，照着写）。
- **已建的老互动救不回来**（库里恒 true，分不清老师本来想不想多选），所以大屏加了

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teacheryo-ai/teacheryo-whiteboard](https://github.com/teacheryo-ai/teacheryo-whiteboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
