---
trigger: always_on
description: 女装电商运营驾驶舱 — 给电商品牌主的飞书 CLI 数字化方案。包含 38 大能力（经营/商品/销售/供应链/管理 5 大板块）+ 飞书 CLI 12 个深度集成 + 14 个独家创新（自演进/精准传达/评论迭代/经验沉淀/双级审批/智能反哺/客观贡献度/妙记自动剪辑/货盘梳理/直播话术分析/虚拟试穿/直播日报/库存 GMV 智能匹配/浏览器自动抓数据）。覆盖经营全流程+组织学习闭环+文档协作闭环+多模态视频/试穿+直播自动化抓数据。当用户提到女装/电商运营/搭配推荐/新品下单/任务追踪/手机指挥/AI 自演进/上传下达/根据评论改文档/经验沉淀/员工贡献度/剪个高光视频/会议金句剪辑/货盘梳理/直播话术分析/虚拟试穿/产品上身/今日直播总结/库存 GMV/浏览器自动抓数据/抖音视频号小红书自动化/初始化系统等场景时使用。
---


# lark-fashion-cockpit — 女装电商运营驾驶舱

> **🎯 核心定位：** 给做女装的人用的（适用所有服装电商品类，化妆品/家居/食品改改字段也能跑）
>
> **📦 集成深度：** 飞书 CLI 23 个原生 skill 中调用 13 个 + 多 CLI 套娃编排（飞书 CLI + douyin-monitor Python CLI）
>
> **🚀 一键搭建：** 用户首次说「初始化 / 帮我搭建系统 / 装这个 Skill」时，跳到下方 [初始化流程](#-初始化流程一键搭建整套数据中枢) 章节。

## 🚦 路由总览（用户意图 → 子 skill）

读完本路由表，根据用户意图跳到对应的 `skills/<name>/SKILL.md` 读详细工作流。

### 🅰️ 公司经营（7 能力，含 1 个 meta-skill）

| 用户意图关键词 | 跳到子 skill |
|---|---|
| "今天怎么样 / 经营异常 / 健康度 / 老板早报" | [`morning-report`](skills/morning-report/SKILL.md) |
| "利润 / 哪些款赚钱 / 投放 ROI / 成本结构" | [`profit-analysis`](skills/profit-analysis/SKILL.md) |
| **"上新任务下发 / 给团队分任务 / 协同跟进"** ⭐ | [`task-collaboration`](skills/task-collaboration/SKILL.md) |
| **"巡检任务 / 看下逾期 / 任务追踪 / 任务复盘 / 为啥老延期"** ⭐ | [`task-lifecycle`](skills/task-lifecycle/SKILL.md) |
| **"飞书消息触发 / 手机指挥 / 离开电脑跑 skill / 套娃模式"** 🔥 | [`event-router`](skills/event-router/SKILL.md) |
| **"AI 审稿 / 设计稿评论 / 自动找改进点"** | scripts/design-review-comments |
| **"审批分流 / 自动批 / 大额升级老板"** | scripts/approval-router.ps1 |
| **"agent 越用越懂我 / 长期记忆 / 自动学习"** | scripts/memory-evolve.ps1 |
| **"会议总结分发 / 按岗位定制纪要 / 让员工不懵 / 上传下达"** 🔥 | [`meeting-broadcaster`](skills/meeting-broadcaster/SKILL.md) |

### 🅱️ 商品中心（8 能力）

| 用户意图关键词 | 跳到子 skill |
|---|---|
| **"产品库 / 产品详情 / SKU / AI 产品分析 / 产品关系图"** ⭐ | [`product-library`](skills/product-library/SKILL.md) |
| "上新波段 / 商品企划 / 开发款式 / 上新节奏" | [`new-launch-planning`](skills/new-launch-planning/SKILL.md) |
| "库存 / 补货 / 缺货 / 滞销 / 平台库存分配" | [`stock-replenishment`](skills/stock-replenishment/SKILL.md) |
| "退货原因 / 评价反馈 / 商品反馈" | [`feedback-returns`](skills/feedback-returns/SKILL.md) |
| "竞品 / 爆款 / 趋势 / 同行" | [`competitor-monitor`](skills/competitor-monitor/SKILL.md) |
| **"XX 配什么好 / 搭配推荐 / 主图穿搭 / 直播搭配 / 老库存清仓"** ⭐ | [`product-matching`](skills/product-matching/SKILL.md) |
| **"新品下多少件 / 备货建议 / 翻单决策 / 尺码颜色占比 / 面料备多少"** ⭐ | [`launch-decision`](skills/launch-decision/SKILL.md) |

### 🅲 销售增长（4 能力）

| 用户意图关键词 | 跳到子 skill |
|---|---|
| "销售数据 / 4 平台对比 / 销售趋势" | [`platform-analytics`](skills/platform-analytics/SKILL.md) |
| **"选题 / 文案 / 内容创作 / 拍摄 / 剪辑 / 发布 5 阶段"** ⭐ | [`content-pipeline`](skills/content-pipeline/SKILL.md) |
| "直播 / 主播 / 排期 / 活动 / 投放 ROI" | [`live-streaming`](skills/live-streaming/SKILL.md) |
| "私域 / 会员 / 客户分层 / 客服" | [`private-domain`](skills/private-domain/SKILL.md) |

### 🅳 供应链履约（2 能力）

| 用户意图关键词 | 跳到子 skill |
|---|---|
| "生产 / 供应商 / 工厂 / 打样 / 交期" | [`production-supplier`](skills/production-supplier/SKILL.md) |

### 🅴 公司管理（11 能力 — 飞书 CLI 增值层 + 组织学习闭环 + 视频剪辑）

| 用户意图关键词 | 跳到子 skill |
|---|---|
| "知识库 / SOP / 客服话术 / 培训资料" | [`knowledge-base`](skills/knowledge-base/SKILL.md) |
| **"会议总结分发 / 按岗位定制纪要 / 上传下达不损耗"** 🔥 | [`meeting-broadcaster`](skills/meeting-broadcaster/SKILL.md) |
| **"提交经验 / 失败教训 / 改进建议 / 经验沉淀"** 🔥 | scripts/experience-capture.ps1 |
| **"经验审批 / 知识库归档 / 复用部门"** 🔥 | scripts/experience-approval.ps1 |
| **"AI 反哺 / 别人犯过吗 / 有人用过吗 / 找谁请教"** 🔥 | scripts/knowledge-feedback.ps1 |
| **"员工贡献度 / 客观评估 / 奖金参考 / 年终述职"** 🔥 | scripts/contribution-tracker.ps1 |
| **"剪个高光视频 / 妙记自动剪辑 / 会议金句 / 自动加字幕"** 🔥 | [`meeting-clip-extractor`](skills/meeting-clip-extractor/SKILL.md) |
| **"货盘梳理 / 选品 / 冲销售/清库存/测款/提利润"** 🔥 | scripts/merchandise-curation.ps1 |
| **"直播话术分析 / 主播评分 / 5 维度复盘"** 🔥 | scripts/livestream-script-analyzer.ps1 |
| **"虚拟试穿 / 模特换装 / 产品上身预览"** 🔥 | scripts/virtual-tryon-mock.py |
| **"今日直播总结 / 直播日报 / 早晚场对比"** 🔥 | scripts/livestream-daily-report.ps1 |
| **"库存 GMV / 库存款销售 / 库存消化"** 🔥 | scripts/inventory-gmv-matcher.ps1 |
| **"浏览器自动抓数据 / 直播平台自动化 / 抖音视频号小红书爬数据"** 🔥 | scripts/livestream-scraper.py |
| **"抓直播 / 拉直播数据 / 自动抓数据"** 🔥 | scripts/livestream-fetch-by-record.ps1 |
| "OKR / 目标拆解 / 部门指标" | [`okr-cascade`](skills/okr-cascade/SKILL.md) |
| "审批 / 预算 / 采购单 / 退货特批" | [`approval-flow`](skills/approval-flow/SKILL.md) |

## 🔥 4 大杀手锏（贯穿全局，不单独成 skill）

| 杀手锏 | 描述 | 在哪用 |
|---|---|---|
| **1. 飞书套娃远程指挥** | event 长连接订阅，老板手机一句话 → agent 调全部 skill | 全局入口（参考 `examples/larkchannel-pattern.md`）|
| **2. AI 产品分析助手** | 自然语言问业务问题 → 读多维表 → 生成飞书文档 | 嵌入 `product-library` |
| **3. 产品关系图** | whiteboard 自动生成"产品×SKU×内容×投流×直播×素材×元素"图 | 嵌入 `product-library` |
| **4. 多 CLI 套娃编排** | 飞书 CLI + douyin-monitor Python CLI + 即梦 CLI + ffmpeg 串联 | 架构层 |

## 📊 数据中枢：14 张飞书多维表

详见 [`lib/base-schema/README.md`](lib/base-schema/README.md)。所有 19 能力共享同一个飞书多维表 App。

| # | 表名 | 服务能力 |
|---|---|---|
| 1 | 产品库 | 5, 7, 14 |
| 2 | 4 平台销售 | 1, 2, 10 |
| 3 | 库存预警 | 7 |
| 4 | 上新波段 | 4, 6 |
| 5 | 任务清单 | 4, 6, 11, 14 |
| 6 | 选题池 | 9, 11 |
| 7 | 文案库 | 11 |
| 8 | 直播排期 | 12 |
| 9 | 生产档案 | 14 |
| 10 | 客户分层 | 13 |
| 11 | 退货反馈 | 8 |
| 12 | 竞品库 | 9 |
| 13 | OKR | 18 |
| 14 | 审批记录 | 19 |

## 🧑‍🤝‍🧑 9 个角色 + 10 步上新流程

任务下发的标准模板（`task-collaboration` 用）：

```
选款 → 设计稿 → 打版 → 打样 → 生产 → 摄影 → 详情页 → 种草内容 → 直播预热 → 上架投流 → 客服培训
```

| 角色 | 负责步骤 |
|---|---|
| 设计师 | 设计稿 |
| 打版师 | 打版 / 打样 |
| 生产主管 ⭐ | 跟工厂 / 进度 / 交期 |
| 摄影师 | 产品图 + 模特图 |
| 模特 | 拍摄配合 |
| 运营 | 详情页 / 上架 / 投流 |
| 内容编辑 | 种草视频 / 图文 |
| 主播 ⭐ | 直播预热 / 直播讲解 |
| 客服 | 话术更新 / 团队培训 |

## 🚀 安装与使用

### 路径 1：完全零代码（推荐普通老板娘）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DontGrowUpAlone/lark-fashion-cockpit](https://github.com/DontGrowUpAlone/lark-fashion-cockpit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
