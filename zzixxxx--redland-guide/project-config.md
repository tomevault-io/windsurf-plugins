---
trigger: always_on
description: 后续所有开发以本文件为准。改了约定要同步改这里。
---

# CLAUDE.md — redland-guide 开发约定

后续所有开发以本文件为准。改了约定要同步改这里。

## 1. 项目是什么

- 小红书 **RED LAND 2026** 线下活动（2026-10-02 ~ 10-06，上海杨浦复兴岛船台 PARK）的**手机端快速查询攻略**。
- 线上地址：https://zzixxxx.github.io/redland-guide/ （GitHub Pages，push `main` 自动部署）
- 仓库：https://github.com/zzixxxx/redland-guide
- 四个底部 Tab：
  1. **展位攻略**（`/booths`）：每日时刻横条 → 主线玩法折叠卡（含本机打卡进度）→ 官方场馆平面图卡 → A/B/C 区 IP 展位列表（搜索 / 区域筹选 / 有攻略 / 已打卡）。点展位进 **展位详情**（`/booth/:id`）：官方一句话、展会信息、展台活动、舞台活动、展台任务、奖励一览、官方笔记原图、来源链接；无详情时给小红书搜索关键词。
  2. **花车巡礼**（`/parade`）：打卡 / 巡游时间 → 官方花车巡礼路线图卡 → DAY1–5 切换的头号花车出场角色 → IP 主题花车（7 台专属花车：效果图 + 按当前 DAY 的出席嘉宾名单 + 笔记图）→ 主角方阵。
  3. **月光舞台**（`/stage`）：DAY1–5 切换 → 主题横幅 → 节目单（歌手 / 曲目 / 来源 IP）→ 营地说明 → 五日主题总览。
  4. **PIN 图鉴**（`/pins`）：三区拼图进度 → 区域 / 夜间 NPC 老玩家 / 拼图筹选 + 「只看已公布」→ 2 列图鉴卡（抠出的单枚 PIN 缩略图、占位编号、获取方式、跳展位、本机「已收集」）。未公布 PIN 的展位按 `booths.js` 自动生成「?」占位卡。

## 2. 用户硬性口径（不要违反）

- **不收录购票 / 票价 / 答题送票**相关内容。只保留场馆地址与入场规则类信息。
- **主要目标用户是手机端**：一切布局先保证 375–430px 宽度可用，桌面端只要不坏即可（`.page` 已限 max-width 640）。
- **视觉参考 RED LAND 官方活动页的像素海岛风**：天蓝格纹底、奶白像素描边卡、红色编号标签、黄色星标、导航深蓝描边。不要改成通用 Material / iOS 风。
- **数据只来自官方**：小红书 RED LAND 官方活动页、各 IP 官方账号的「RED LAND2026 | XX展台活动详情」笔记、官方新闻稿。不编造、不猜测；未公布的写「待补充 / 待确认」。
  - 唯一例外：首页「场馆平面图」卡下半部分保留 **2025 年**网友整理的参考图（`rules.js venueMapRef`，用户 9/10 决定放、9/11 官方图出来后决定继续保留），UI 与数据必须标明「2025 年」「非官方」。
- **场馆平面图官方图已接入**（`rules.js venueMap`，RED LAND 官方号 9/11 笔记）：放在首页「场馆平面图」卡上半部分（原 LOADING 占位处），5 张图在 `public/img/rules/map-2026/`。**全图必须原像素**（用户 9/11）：缩略图 `00.jpg` 3200 宽，灯箱打开加载 `00-full.jpg` 14412×5854（q88 约 5.5MB），走 `Lightbox` 的 `item.full`。给每个展位挂坐标（点展位在图上定位）仍未做——官方图没有网格，要逐个量展位框中心点。
- 多步任务默认直接推进，不逐步确认；只在需要业务口径 / 方案取舍时停下问。

## 3. 技术栈与目录

- Vite 8 + Vue 3（`<script setup>`）+ vue-router 4（hash 模式）。无 Pinia、无 UI 库、无 TS。Node 22。
- `vite.config.js` 的 `base: '/redland-guide/'`，所有 public 资源在代码里用 `import.meta.env.BASE_URL + 'img/...'` 拼路径。

```
src/
  main.js / App.vue           入口；App 里 keep-alive 三个列表页（按组件 name 匹配）
  router/index.js             /booths  /booth/:id  /parade  /stage
  style.css                   全部样式（设计 token 在 :root；像素组件类见 §5）
  composables/useStore.js     useChecked（展位打卡，localStorage rl26.checked）/ useDay（花车与舞台共享的当前 DAY，rl26.day）/ useCollected（PIN 已收集，rl26.pins）
  components/                 TabBar（底栏）PageHeader（顶栏，back 模式）DayChips（DAY1–5）Lightbox（多图灯箱：左右滑动翻页、长图按宽铺满上下滚动、← → Esc）StepList（活动 / 任务的分步列表）PostCopyBtn（带话题任务的「复制发帖文案」）
  utils/xhs.js                小红书链接：主页 / 搜索 URL；openProfile 在手机端先唤起 App（xhsdiscover://user/<uid>，Android Chrome 走 intent://），未安装 / 取消再退回网页，PC 不拦截
  views/                      BoothsPage / BoothDetailPage / ParadePage / StagePage / PinsPage
  data/                       所有内容数据，纯 JS 模块，见 §4（含 roaming.js：无固定展位、场内游荡分发物料的 IP）
public/img/booths/<展位id>/   各 IP 笔记原图（810px 宽 JPEG，无水印版）+ note.json（抓取原始数据，含 fileIds / keptIndex）
public/img/pins/              从笔记图抠出的单枚 PIN 缩略图（<pin id>.jpg，最长边 320px）+ zone-A/B/C 通用占位软盘
public/img/roaming/<id>/      游荡 IP 的笔记图 + note.json（id 用拼音，如 gongyongbingxiang）
public/img/parade/<id>/       花车巡礼：route/ 官方路线图（3118 宽原图）+ 7 台 IP 专属花车笔记图（eggy / nishuihan / xindong / yimo / naraka / valorant / yuewen，810 宽）+ note.json
scripts/fetch-note.mjs        抓小红书笔记正文 + 图片（按 fileId 拉无水印原图），打印 boothDetails 骨架
scripts/refetch-clean.mjs     把已抓的带水印图按 note.json 的 fileIds 重拉成无水印版（历史目录一次性用过，新目录不需要）
scripts/crop-pins.py          按裁切框从笔记图抠单枚 PIN 缩略图到 public/img/pins/
scripts/fetch-ditto.mjs       抓 ditto 专题页（目录页 + --sub 子页）全部图片、热区跳转、关注组件 uid → ditto.json
scripts/xhs-profile.mjs       按 uid 读小红书主页公开信息（昵称 / 小红书号 / 认证类型 2=官方 / 粉丝），`--all` 核对 booths.js 里已填账号；有风控，连续约 3 个后要等
docs/                         总资料底稿：REDLAND2026_信息汇总.md + assets/（官方页面图、各 IP 笔记归档）+ raw/（DSL JSON、逐图转录、KOL id）；不参与构建，见 §9
.github/workflows/deploy.yml  push main → build → GitHub Pages
```

## 4. 数据约定（`src/data/`）

| 文件 | 内容 | 来源 |
|---|---|---|
| `booths.js` | `booths[]`：`{ id, zone, no, ip, blurb, alias? }`；`zones[]`；`boothMap` | 官方「冒险者攻略 · IP展位一览」 |
| `boothDetails.js` | `{ [boothId]: detail }`，schema 见下 | 各 IP 官方账号「展台活动详情」笔记 |
| `indie.js` | C04 独立游戏试玩名单：`indieGames[{ letter, games[{ name, en?, xhs?: { uid, name }, url? }] }]` + `indieSource`；有 `xhs` 详情页渲染 📕 跳主页，有 `url` 名字可点开攻略（目前都待补） | 官方「独立游戏聚合页」ditto 93f070416d60405ea59f29bb691a0df3（9/9 版 84 款，页上无关注组件 / 热区） |
| `parade.js` | `paradeInfo` / `paradeRoute`（官方路线图：`image` / `desc` / `route` 按图理解的走法 / `legend` / `encounters` / `source`）/ `paradeDays[{ day, date, entries[{ ip, chars[] }] }]` / `themeFloats[{ id, ip, desc, look, intro, images[], guests[{ day: 1–5 \| 'all', label?, chars[] }], guestNote?, source }]` / `playerSquad` | 官方「花车巡礼」半层 + RED LAND 官方号 9/11 路线图笔记 + 9/5 七条「前方高能！XX 专属花车准备发车！」笔记 |
| `stage.js` | `campInfo` / `stageDays[{ day, date, theme, hint, items[{ performer, songs[], ip?, note? }] }]` | 官方「冒险者营地」半层 |
| `roaming.js` | `roaming[]`：`{ id, name, chars, xhs, days[], dateText, where, items[{ name, how }], note, image, source }` 无固定展位的游荡 IP，首页展位列表下方「自由游荡的 IP」卡 | 该 IP 官方账号笔记（已收录：公用冰箱里有什么 / 鼠记私房菜，10/4） |
| `rules.js` | `event`（含 `days[]`）/ `mainline`（含 `regions[].pin/color`、`nightPin`、`images`、`source`）/ `eggs` / `places` / `dailySchedule` / `venueMap`（官方平面图：`images` / `routes` 路线图例 / `routeTip` / `tips` 交通与点位 / `facilities` 回血点位 / `source`）/ `venueMapRef`（2025 年参考图 + 交通要点，非官方，保留） | 官方「冒险者攻略」半层 + 主会场 + RED LAND 官方号 8/25「PIN 收集玩法」、9/11「登岛地图已解锁」笔记；`venueMapRef` 来自网友「星辰大海」2025-08-07 笔记 |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zzixxxx/redland-guide](https://github.com/zzixxxx/redland-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
