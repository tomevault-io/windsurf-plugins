---
trigger: always_on
description: Plan trips and build day-by-day travel itineraries, delivered as one polished, standalone HTML page. Use this skill PROACTIVELY for any travel-planning request — trigger on "帮我规划行程 / 旅游攻略 / X日游 / 安排去XX玩 / 做旅行计划 / 行程安排 / 帮我排个行程", on "plan a trip / make an itinerary / X-day trip to a city / things to do in a place", and whenever the user gives a destination plus dates or a duration (e.g. "6月去东京玩5天", "国庆想去成都4天", "trip to Kyoto next month", "下周末杭州两日游"). Always gather requirements first (never jump s
---


# Trip Planner Skill

Produces a detailed, realistic, **day-by-day travel plan** as a single polished HTML file —
geographically sensible routing, honest time budgets, verified opening hours and prices, an embedded
interactive map, and a pre-trip checklist. The visual design reuses the `study-notes` HTML output
spec (collapsible blocks, hierarchical nav, colored callouts, dark-mode theme, scroll-position
memory), retargeted to travel and documented in **`references/design-system.md`**.

**Audience assumption**: the reader is the traveler. Write so they can follow the plan on their phone
during the trip — concrete times, addresses, prices, "how to get there", and what to do if it rains.

**Environment (Windows / Claude Code).** There is **no** `places_search`, `places_map_display_v0`, or
`weather_fetch` tool here. Do research with **`web_search`**; render the map by embedding
**Leaflet + OpenStreetMap** in the output HTML (OSM tiles need no API key). For requirements
gathering use the **`AskUserQuestion`** structured-question tool if available, otherwise ask in plain
text — batched by dependency layer, each question carrying a recommended answer (see Step 1).

**Browser scraping — read the DOM with `javascript_tool`, don't screenshot-and-eyeball.** 查机票/酒店价、
携程/高德评论时，**优先用 `javascript_tool` 抓 DOM 取结构化 JSON**（省 token，且绕开高德 canvas 截图报错——
评价在 HTML 面板里，JS 抓得到）。探测脚本 + 各站点已复验选择器模板见 **`references/scraping-method.md`**；
只在确认加载/需要视觉/canvas 内容时才截图。**保存的选择器只是"自检缓存"、会过期——取到 0 条就按该文档
§八 自愈降级（探测脚本 → 锚定内容模式 ¥/HH:MM/日期 → 语义兜底），别干等、别放弃；自愈失败仍取不到，就如实
标「未读到 · 以官网为准」，绝不编。**

**Output path.** Write the final HTML to the current working directory (or a directory the user
names). Filename: `<目的地><N>日游行程.html` (e.g. `东京5日游行程.html`).

---

## 数据诚信契约（DATA INTEGRITY — 凌驾本文件一切其它规则）

**这个 skill 唯一不能破的底线：页面上每一个具体数字/事实，要么本会话真的查到了，要么就别写成具体值。
宁可不写，绝不编。** 一次编造的营业时间能把游客送到闭馆的门口，一张编造的"4.8 分 3,201 条点评（已核）"
能让整份行程的可信度归零。这条契约的违反，比任何排版/组件缺失都严重。

**浏览器优先，不是浏览器可选。** 机票、酒店（价格+评论）都是**强制要主动尝试浏览器**的项——
先去连/去查，连不上或用户跳过才退到估价。**不许因为"这趟没机票"或图省事，跳过尝试直接走 web_search 估价。**

**两类来源，泾渭分明：**

- **实查（可写具体值）**：本会话真的用浏览器（携程/飞猪/航司官网/高德）或 `web_search` 读到了，
  且**就近标了来源**——浏览器读的标「实查于 YYYY-MM-DD」/「浏览器实读点评区」/「高德实测」，
  web_search 读的标「web_search · 未浏览器核实」，火车标「以 12306 为准」。
  **注意：`web_search` 只能给价格区间和事实概要，给不了真实评论区——评论结论与评分只有浏览器读得到，
  web_search 写评论=编造。**
- **没查到（禁止写具体值）**：没读到就**不许填一个看起来合理的数**。用不含杜撰细节的 hedge 占位，
  例如「营业时间以官网为准」「评分以 App 为准」「打车费用以高德为准」「车次时刻以 12306 实时为准」。

**受控『已验证』词表——只有真查到才能用：**
`已核 / 已核实 / 可信 / 实测 / 实测数据 / 浏览器实读 / 浏览器实查 / 高德实测 / 分布正常 / 不像刷分 /
好评集中在X / 口碑稳 / 活跃真实`。这些词**只有当本会话确实读到对应内容、且同一处带了来源+日期标记**时
才允许出现。**同一张卡/同一段里，价格标了「未核实/估价」，就绝不能在别处出现这些词**——那是自相矛盾，
是最恶劣的"伪装成已核实"。也不要用同义词（口碑扎实/水分不大/可放心订/评分稳）绕开——**任何关于评论区
的结论性判断，没有就近来源标记，一律不写。**

**按面分项的硬规矩（每一面都管，不只是酒店）：**

| 数据 | 实查到 → 可写 | 没查到 → 只能写 |
|---|---|---|
| 酒店评分/点评数 | 具体分+条数+「实查于 日期」 | 留空，或「评分以 App 为准」（**禁止**任何分数/条数） |
| 评论体检结论 | 读到的真实复发主题+来源标记 | 只写"为什么选它/订前自查"这类连锁常识推荐理由 |
| 酒店/机票比价 | 现役平台（携程/飞猪/官网）真实价 | 现役平台估价并标「（估）」；**已废平台（去哪儿/美团/同程/艺龙）禁止出现**；估价行不得打「✓最低」 |
| 门票¥/营业时间/地址 | 实查值 | 就近标「以官网为准」（**禁止**编逐项联票价并做"单买超¥X"的派生算术） |
| 车次号/分钟级时刻/票价 | 实查值+「以12306为准」 | 不写具体车次号与到发分钟，只写「高铁直达约 Nh · 班次票价以12306实时为准」 |
| 预约配额/放号时刻 | 实查值 | 「名额有限 · 配额与放号以官方公众号当日为准」（**禁止**编"每日2000人/8:00放号"） |
| 打车¥/分钟/里程 | 高德路线规划值+「（高德实测）」 | 「打车可达 · 费用时长以高德为准」（短步行腿 ≤2km 可按 70–80 m/min 估距离+步行 min，但一旦出现¥或"打车"就必须高德来源；**禁止**"步行或打车 N min"这种混写偷渡打车分钟） |
| 步数 | 由实测里程换算（≈1,300 步/km） | 不标具体步数 |
| 天气（>14 天外） | —— | 「X月气候典型 约A°/B°（非当日预报，出行前再查）」，**不得对多日复制同一具体温度串当各日预报** |
| 地图坐标 | 知名点的已知坐标 | 定位不到就降到城区级并注明近似，**禁止**给定位不到的点编 4 位小数精确坐标 |
| 预算派生 | 各项来自上面的实查/估价 | 不得把编造的单价滚成"精确"日小计/总计；header 总价与预算表必须一致 |

**逃生门（"你直接安排就行"）只放宽提问，不放宽这条契约**：缺的偏好按推荐值填、写进顶部假设 callout；
但任何**事实**仍然遵守上表——没查到就 hedge，绝不因为是非交互模式就开始编数字。

`scripts/check_html.py` 的第 8–10 检查是这条契约的**兜底**（抓最常见的编造：无凭据的酒店评分/评论结论、
假"最低"比价、复制粘贴的假天气）。但校验器只是结构兜底、挡不住刻意绕过——**真正的防线是你自己守住"宁可
不写绝不编"**。

---

## 工作流总览（plan → research → verify → assemble）

Don't free-write a plan in one pass. The quality comes from the structure:

1. **Plan** — gather requirements (Step 1), then sketch the day count, geographic clusters, and which
   stops need booking. Nothing is researched yet; this is the skeleton.
2. **Research (fan-out)** — for each day/cluster, look up transport, weather, opening hours, closure
   days, and prices with `web_search` (Steps 2–3). For a big trip you may run parallel subagents, one
   per day or per city.
3. **Verify (the decisive step)** — enforce the **数据诚信契约 above**: every specific number/fact is
   either actually verified this session (with a source marker) or written as a hedge that carries **no
   invented specifics** — never a plausible-looking number. This applies to **every surface**, not just
   hotels: 票价/营业时间/地址/车次/时刻/船票/打车费/里程/步数/预约配额/坐标/天气. Re-check that no stop
   is scheduled on its closing day and that intercity connections leave realistic buffers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eric6286/trip-planner](https://github.com/Eric6286/trip-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
