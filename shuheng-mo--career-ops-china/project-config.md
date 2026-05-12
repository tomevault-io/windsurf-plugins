---
trigger: always_on
description: This system was built and used by [santifer](https://santifer.io) to evaluate 740+ job offers, generate 100+ tailored CVs, and land a Head of Applied AI role.
---

# Career-Ops -- AI Job Search Pipeline（中国大陆版）

## Origin

This system was built and used by [santifer](https://santifer.io) to evaluate 740+ job offers, generate 100+ tailored CVs, and land a Head of Applied AI role.

**This fork has been deeply customized for the China mainland job market** — Chinese language by default, archetypes adapted for 数据工程 / 数据治理 / 数据仓库 / 大模型 / AI Infra / 后端 / 平台 roles, salary research sources switched to 看准网/脉脉/OfferShow/知乎/一亩三分地, portals.yml populated with Chinese tech giants and AI unicorns, and login-wall handling for Boss直聘/拉勾/猎聘.

The portfolio that goes with the original system is also open source: [cv-santiago](https://github.com/santifer/cv-santiago).

**It will work out of the box, but it's designed to be made yours.** If the archetypes don't match your career, the scoring doesn't fit your priorities -- just ask. You (Claude) can edit any file in this system. 用户说「改 archetype 到 后端 + 推荐算法」你就直接改。这就是这个系统的目的。

## What is career-ops

AI-powered job search automation built on Claude Code: pipeline tracking, offer evaluation, CV generation, portal scanning, batch processing.

### Main Files

| File | Function |
|------|----------|
| `data/applications.md` | Application tracker |
| `data/pipeline.md` | Inbox of pending URLs |
| `data/scan-history.tsv` | Scanner dedup history |
| `portals.yml` | Query and company config |
| `templates/cv-template.html` | HTML template for CVs |
| `tools/generate-pdf.mjs` | Puppeteer: HTML to PDF |
| `article-digest.md` | Compact proof points from portfolio (optional) |
| `interview-prep/story-bank.md` | Accumulated STAR+R stories across evaluations |
| `reports/` | Evaluation reports (format: `{###}-{company-slug}-{YYYY-MM-DD}.md`) |

### First Run — Onboarding（重要）

**做任何事之前，先检查系统是否已配置。** 每个 session 开始都静默执行下面这些检查：

1. `cv.md` 是否存在？
2. `config/profile.yml` 是否存在（不只是 profile.example.yml）？
3. `portals.yml` 是否存在？（中国大陆版默认已存在，不需要重新创建）

**如果 cv.md 或 profile.yml 缺失，进入 onboarding 模式。** 在基础文件齐全之前，**不要** 跑评估、扫描或任何其他 mode。一步步引导用户：

#### Step 1: CV（必须）
如果 `cv.md` 不存在，问：
> "还没有你的简历。你可以：
> 1. 直接把简历内容贴过来，我帮你转成 markdown
> 2. 给我你的 LinkedIn 或 个人主页 URL，我提取关键信息
> 3. 告诉我你的经历（学校、工作、项目），我帮你起草一份
>
> 你想怎么做？"

不管他们给什么，创建 `cv.md`。用干净的 markdown，标准段落：摘要、工作经历、项目经历、教育、技能。**注意中文 CV 的特殊约定**（看 `modes/pdf.md` 的"中国大陆 CV 的额外约定"段）。

#### Step 2: Profile（必须）
如果 `config/profile.yml` 不存在，从 `config/profile.example.yml` 复制，然后问：
> "我需要几个信息来个性化这个系统：
> - 你的姓名 + 邮箱 + 微信（不会写到生成的内容里，只本地存）
> - Base 城市 + 是否能搬迁
> - 目标岗位（数据工程师 / 数据仓库专家 / 大模型应用工程师 / 等）
> - 期望薪资区间（包不包含年终奖、股票）
> - 现在状态：在职 / 离职 / 在看
>
> 我帮你填好。"

把答案写进 `config/profile.yml`。把目标岗位映射到最接近的 archetype（在 `modes/_shared.md` 中），如果不匹配就改 `_shared.md`。

#### Step 3: Portals（可选 — 中国大陆版默认已配好）
中国大陆版的 `portals.yml` 已经预配置好了 50+ 公司：
- 一线大厂：字节、阿里、腾讯、美团、百度、京东、拼多多、快手、小红书、B站、网易、滴滴
- 大模型独角兽：DeepSeek、Moonshot、智谱、MiniMax、百川、零一、阶跃星辰、面壁
- 数据创业：PingCAP、StarRocks、神策、Kyligence、滴普
- AI Infra：硅基流动、潞晨、无问芯穹

问候选人：
> "portals.yml 已经包含了主流公司。想加自己关注的公司吗？或者想去掉哪些不感兴趣的？"

#### Step 4: Tracker
如果 `data/applications.md` 不存在，创建：
```markdown
# Applications Tracker

| # | Date | Company | Role | Score | Status | PDF | Report | Notes |
|---|------|---------|------|-------|--------|-----|--------|-------|
```

#### Step 5: 了解用户（决定评估质量的关键步骤）

基础就绪后，主动问更多上下文。**你了解候选人越多，评估越准。**

> "基础已经搞定。但这个系统最大的杠杆是『了解你』。能多告诉我一些吗：
> - 你的『独门武器』是什么？同岗位的其他人没有的能力？
> - 什么样的工作让你兴奋？什么让你疲惫？
> - 有没有 deal-breaker？（如：不去 996 / 不去南方 / 不做 ToC / 不去外包）
> - 最让你自豪的一段项目经历，面试时你会主讲的那个？
> - 有没有公开的项目、文章、知乎回答、GitHub 仓库？
>
> 给我越多上下文，我就能过滤得越准。把我当成一个新来的猎头 — 第一周需要了解你，之后就会变得很有价值。"

把候选人分享的信息存到 `config/profile.yml`（narrative 段）或 `article-digest.md`（如果是 proof points）。如果他们描述的方向和默认 archetype 不一致，更新 `modes/_shared.md`。

**每次评估后都要学习。** 如果用户说"这个分太高了，我不会投" 或 "你漏掉了我有 X 的经验"，更新你的理解。调整 `_shared.md` 的 framing 或在 `profile.yml` 加备注。**系统应该每次互动都变得更聪明。**

#### Step 6: 准备就绪
所有文件都齐了之后，告诉用户：
> "你已经全部准备好了！现在可以：
> - 贴一个岗位 URL 让我评估
> - 跑 `/career-ops scan` 扫描门户
> - 跑 `/career-ops` 看所有命令
>
> 一切都可以定制 — 想改什么直接告诉我。
>
> 提示：有个个人作品集 / 技术博客对求职帮助很大。如果还没有，可以参考原作者的开源项目：github.com/santifer/cv-santiago，fork 一份改成自己的。"

然后建议自动化：
> "要不要我定期帮你扫描新岗位？我可以设置每几天自动扫一次。说一句『每 3 天扫一次』我就配好。"

如果用户接受，用 `/loop` 或 `/schedule` skill（如果有）配置周期性 `/career-ops scan`。如果没有这些 skill，建议加 cron 或者提醒用户定期手动跑。

### 中国大陆求职市场的特殊提醒

候选人在 onboarding 或评估过程中如果出现以下情况，主动给提醒：

| 情况 | 提醒内容 |
|------|---------|
| 候选人 33+ 岁 | 互联网 35 岁红线是真实存在的。优先投独角兽 / 中小厂 / 外企，大厂社招对资深 P7+ 容忍度更高 |
| 候选人离职找工作（gap > 3 个月） | gap 在国内 HR 眼里是负面信号。准备好解释（创业 / 学习 / 家庭 / 健康 — 选最不影响匹配度的） |
| 候选人想投远程岗 | 国内远程岗几乎不存在。海外华人公司（HuggingFace、Replicate 等）有但门槛高 |
| 候选人没有"大厂背景" | 双非 / 二本 / 没大厂经历 → 用真实项目和数据补 |
| 候选人在评估 996 公司 | 在 Block D 明确写出真实工时，让候选人自己决定 |
| 候选人想跳到大模型方向 | 强调"端到端落地经验"比"会调 LangChain"重要 — 让面试官看到 Eval / Observability / 成本控制 / 业务影响 |
| 候选人简历提了"在某大厂做过外包" | 别隐瞒，但用项目而不是 title 来 hook |

### Personalization

This system is designed to be customized by YOU (Claude). 用户让你改 archetype、调评分、加公司、改谈判话术 — 直接改。你能读到同样的文件，所以你知道要改哪里。

**常见定制请求：**
- "改 archetype 到 [后端/前端/算法/SRE]" → 改 `modes/_shared.md`
- "把 modes 翻成英文" → 改 `modes/` 下所有文件
- "把这些公司加到 portals" → 改 `portals.yml`
- "更新我的 profile" → 改 `config/profile.yml`
- "改 CV 模板设计" → 改 `templates/cv-template.html`
- "调评分权重" → 改 `modes/_shared.md` 和 `batch/batch-prompt.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuheng-mo/career-ops-china](https://github.com/shuheng-mo/career-ops-china) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
