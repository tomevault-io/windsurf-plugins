---
trigger: always_on
description: Travel assistant for generating source-checked, visually polished travel guides. Handles three user states: no clear idea yet, partial plan needing completion, and fixed places needing itinerary cleanup. Includes path triage, artifact intake, active research, fact ledger, image asset collection, geography/time validation, destination-specific visual theming, a required HTML/CSS print source, browser-exported editorial PDF output, a trip-wide constraints layer (budget tier, companion structure, t
---


# 旅游攻略 Skill · travel-itinerary

## 一、触发条件

用户提出旅行相关任务时触发，包括：

- 「不知道去哪，帮我推荐/做攻略」
- 「想去 X，但还没定路线/天数/酒店」
- 「这些地点我想去，帮我排成行程」
- 「我已有每天大概安排，帮我整理成能旅行时看的文件」

**触发后第一件事**：执行 Stage 0.0 情况评估（见下方），确定走哪条流程路径。

## 二、流程路径（四界面管道 + 三条成熟度分支）

**核心方案（v3.0）：围绕「地图 + 点位」做规划布局，再一键导出渲染成攻略。问卷表格方案（Q1/Q2）已彻底退役，归档在 `assets/_deprecated/`，不再使用。**

四个界面顺序串联，形成「发现 → 规划 → 补充 → 出图」的数据管道：

```
discovery     planner             supplement        trip-print-v2
发现选点   →  地图排点位+分天  →  勾选备选       →  8模板渲染出图
（勾必去/  →  （Leaflet拖拽   →  （备选景点/    →  （导出 PDF）
 感兴趣）      排序+锁定）         美食/特产）
```

**多城（≥2 城）走同一条管道，不另造分支**：开局先在 Stage 0 定下「城市顺序 + 每城天数 + 城际交通段」骨架（`meta.city_plan`，见第六章）。planner 地图按当前天的点位自动 fit，按天切换自然聚焦当天的城——所以**多城用单个 planner 文件即可，不拆文件、不分图、不分次跑**。跨城游产出粒度就是「天」：每天归属它所在的城，城际那天出 T2 转场页，由现有「按天出页」原生覆盖。

### 路径决定走哪几个界面（由 checklist 9.0 路径分诊门判定主路径）

| 路径 | 触发 | 走哪几个界面 |
|---|---|---|
| **A · Full** | 没想法 / 认知低 | discovery → planner → supplement → trip-print-v2（全程） |
| **B · Constraint** | 有部分计划 | 跳过 discovery；planner 直接载入「用户已有点 + Claude 补研究的点」→ supplement → trip-print-v2 |
| **C · Express** | 行程已定 | 跳过 discovery + supplement；planner 仅核实/排序确认 → trip-print-v2 |

分段成熟度仍生效（见 9.0.2）：`locked` 段只核实、`open` 段主动研究——路径只决定界面起点，研究力度按段走。

### 三个数据交接断裂点（铁律 · 必须 Claude 手工转换）

界面间用 JSON 导出/导入衔接，但有三处对不齐，Claude 必须补：

- **断裂 A（discovery → planner）**：discovery 导出的选择列表**无坐标**。进 planner 前，Claude 必须为每个点补 `coords:[lat,lng]`（web_search 地图/高德/OSM 查），注入 planner 的 PLACES 池（注入时按实际给 planner type，方言对照见 `references/type-vocabulary.md`）。**没坐标 planner 地图无法标点。**
- **断裂 B（supplement 独立）**：supplement 与 planner 行程**完全解耦**，只导出补充项的 id+name。最终在构建 TRIP 时**并行合并**进 `extras`，不从 planner 拼接；**合并前按 POI id/name 去重——已进 planner timeline 的点不再重复进 extras**（见 checklist I-1③）。
- **断裂 C（planner+supplement → trip-print-v2）**：两者导出 ≠ TRIP 结构，**不能直接拼**。先按 `references/type-vocabulary.md`（链路通用语言总表）做 type 归一化+细化再组装。Claude 必须：① **type 归一化**——planner/discovery 方言全部翻成标准 type，`spot/experience` 粗分按实际细化（自然=sight_nature·寺庙=sight_cultural·徒步=experience_outdoor）；② **合成转场段**——跨城/跨岛那天造 `{type:'transit',intent:'intercity',from,to,coords,duration}`（planner 不产出转场段，漏了选不出 T2）；**城际段对账**：按 `meta.city_plan.intercity` 逐段核对——骨架里有 N 段城际交通，最终 timeline 就必须有 N 条 `{type:'transit',intent:'intercity'}`，逐段比对 from/to，缺一段即不通过（防三城两段漏造中间一段）；段数 + from/to 对账之外，**还要过一遍距离常识**——反常识超长段（跨大区/跨气候带，如"吉林→腾冲 3000km"）回 9.0.0 事实预检消歧，不静默通过；③ **标徒步段**——整天/半天徒步主段标 `experience_outdoor`+必填 `duration`，视情填 `on_trail_food`（否则选不出 T3）；③.5 **自驾沿途景观点不丢**（病灶⑮）——自驾 / 自驾环线转场日，沿途经过的景观点（如五彩滩、魔鬼城）**必须造成正常的 `sight` / `sight_nature` 点位**，**且整体排在该日 timeline 的 transit 段之后**（先把当天所有 intercity transit 段连排，再排沿途点 + 落地活动），**绝不把景观点夹在多段 transit 之间**。原因：选页器只提取「末段跨城 transit 之后」的落地点（`landing = timeline.slice(lastCrossIdx+1)`），夹在多段 transit 中间的景观点会被 slice **彻底切掉、静默消失**；排在 transit 段之后才保得住。**已知代价（L1 · 校准措辞）**：当沿途点 ≥3 个时，它们会被选页器**独立渲染成到达城的「城内游」页、地图归到达城名下**，可能与点的实际位置有偏差（不只是「被当成到达点的活动」，偏差比这更明显）；**点不丢**（满足止血目标），但**归属语义有偏**——这是 L1 已知且接受的妥协（不丢内容优先于语义精确），根治留待 L2。④ **回填 maturity**——每个 day 按 9.0.2 标（A 路径缺省 open·B/C 缺省 partial）；⑤ `planner.days[].places[]` 转成 `TRIP.days[].timeline[]`，补 `meta`（日期/人数/预算/交通/酒店）、`global`（紧急/清单/票据）、`days[].meals`/`reminders`/`weekday`/`theme`；⑥ 丰富每个 card 的 `emoji/intro/ph/price`（intro 2-4 段，按 cards schema）；⑦ 跑 `selectPagesForDay` 拆页，**验收：点位无消失·徒步日出 T3·转场日出 T2·每天 maturity 已落·`meta.themes[].anchors` 每个（已过 9.0.0④ 核查、且在计划城市内的）点都在某天 timeline 找得到对应 card（anchor 保入对账，防主题点被稀释丢失，见 checklist 9.0.5⑥②）**；⑧ **约束统一出口**——`meta.constraints` **全部由「出行须知」区渲染**（含 `scope` 为 region slug / `'day:N'` 的约束，渲染层自动在文字前标注适用范围：`day:N`→`【第N天】`、region→`【region名】`，无中文名映射时降级显示 slug 原文）。region/day 级约束**可选**额外落进对应 `day.reminders` 作当天上下文增强，但「出行须知」区才是保证不丢的**唯一主出口**——别再依赖各页型渲染 reminders 来承载约束。**跨境行程额外**：把 `meta.city_plan.entry_exit.border_notes` 每条转写成 `meta.constraints`（`type:'transport'`、`scope:'trip'`）再走上述出口，`inbound/outbound` 的口岸→市区交通落进抵达/离境日 T2；`entry_exit` 自身不出图（病灶⑫：填了不转写=不出图=白填）。**主题行程额外**：把 `meta.themes[]` 每条转写成 `meta.constraints`（`type:'theme'`、scope 沿用，text 点明「本趟贯穿 X 主题、已优先安排取景/朝圣点」）再走上述「出行须知」出口；themes 自身不出图，只填不转写=用户看不到主题=病灶⑬没修（与 `border_notes` 同一转写模式）。⑨ **跨夜长途转场**（夜船/红眼/夜卧跨午夜）按 9.0.6 分支②单独造一个 `day`（timeline = 该 transit 段 ＋ 一个 `type:'free'` 锚点；纯 transit 无 free 的日会被 `selectPagesForDay` 判成 T8 休整页、出不了 T2），让 `selectPagesForDay` 出 T2，到达城首日从次日另起。

### 文档职责边界（维护规则）

- `SKILL.md` 只放触发条件、路径选择、资产导航和最高优先级原则。
- `references/v2.4-checklist.md` 虽保留旧文件名，但内容按当前版本维护；所有执行细则、自检项、事实核查规则以它为准。
- `references/fact_ledger.template.md` 是事实台账模板；新 case 优先复制它，不要每次重新发明格式。
- `references/asset_ledger.template.md` 是图片资产台账模板；PDF 配图必须用它追踪来源和状态。
- `references/pdf-output-rules.md` 是最终画册交付规则；它规定 HTML/CSS 是唯一视觉排版源，PDF 必须由浏览器从 HTML 导出。
- `references/design-rules.md` 只管视觉方向、色板和 CSS 变量映射。
- 若同一条执行规则在 `SKILL.md` 和 checklist 同时出现，以 checklist 为准，并在下次维护时删掉 `SKILL.md` 中的重复细节。

### Stage 0.A · 输入物识别（用户给了文件/截图时必做）

如果用户提供图片、PDF、地图、备忘录截图、票据截图、聊天记录或散乱文档，先判断输入物类型，再选流程。不要默认 PDF 一定有文字，也不要默认截图只是参考图。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [logic0512/travel-itinerary](https://github.com/logic0512/travel-itinerary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
