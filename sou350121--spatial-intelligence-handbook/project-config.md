---
trigger: always_on
description: 本文件面向在 Spatial-Intelligence-Handbook 仓库中工作的自动化 / AI agent，说明写作与维护规范。姊妹仓库 [VLA-Handbook](https://github.com/sou350121/VLA-Handbook) 的协议为基准，本文档列出**保留 / 收紧 / 新增**三类规则。
---

# AGENTS.md

本文件面向在 Spatial-Intelligence-Handbook 仓库中工作的自动化 / AI agent，说明写作与维护规范。姊妹仓库 [VLA-Handbook](https://github.com/sou350121/VLA-Handbook) 的协议为基准，本文档列出**保留 / 收紧 / 新增**三类规则。

## 目标
- 维护结构化、可检索、**跨 embodiment 可对比**的空间智能知识库
- 严守 `crossing/` 是本仓 USP — 任何会被独立 embodiment 综述写出的内容，都不该独占在某一 `embodiments/<x>/` 下
- 工程细节与 sensor 物理优先于综述堆砌（学界综述写不出 SWaP-C 工程账）

---

## 仓库快速导航

```
foundations/      # 跨 embodiment 共享底层（3DGS / VGGT / depth / semantic 3D / sensor-physics ★）
embodiments/      # 各 embodiment 应用层（manipulation / humanoid / ground / driving / aerial ★ / marine）
crossing/         # 跨 embodiment 合流 ★★ USP（scale / sensor / SLAM-VIO / representation / failures）
deployment/       # 工程实战（hardware-selection / sync / calibration / compute / failure-modes）
benchmarks/       # geometry / manipulation / driving / aerial / marine / reasoning
bridge-to-vla/    # 与 VLA-Handbook 接口（3D-aware VLA、feature-cloud→action、neural-map-memory）
companies/        # 产业地图
reports/          # weekly + biweekly（Pulsar pipeline 输出）
cheat-sheet/      # timeline + representation-comparison + sensor-budget-matrix
docs/             # 仓库元文档（含 pulsar-integration.md）
```

★ = 维护者深度锚点（写得比其他 embodiment / 主题深 1.5–2×）

---

## Pulsar 写入协议（继承自 VLA-Handbook，路径与权限矩阵已重定向）

> **核心原则不变**：自动 agent 默认只做"追加行 / 创建新文件"。仅在权限矩阵明确允许时才可对自动生成文件做同日 upsert（重跑修复）；**永远不修改、不删除人工内容**。

### 写入权限矩阵

| 文件 / 目录 | 允许操作 | 禁止操作 |
|---|---|---|
| `foundations/*/README.md` · `embodiments/*/README.md` · `crossing/*/README.md` | 追加表格行（条件触发，见下） | 修改既有行、改表头、改子目录结构 |
| `foundations/<lane>/{paper_or_topic}_dissection.md` | 创建新文件 | 修改已存在的文件 |
| `embodiments/<emb>/<axis>/{topic}_dissection.md` | 创建新文件 | 修改已存在的文件 |
| `crossing/<lane>/{topic}.md` | 创建新文件 ★ 高门槛见 §「Crossing 写入门槛」 | 修改已存在的文件 |
| `reports/weekly/{YYYY-MM-DD}.md` | 创建新文件；同日重跑允许覆盖（仅限自动生成） | 修改人工撰写报告、改报告结构 |
| `reports/biweekly/{YYYY-MM-DD}.md` | 同上 | 同上 |
| `reports/{weekly,biweekly}/README.md` | 追加索引行；同日 upsert | 修改其他日期行、改文件结构 |
| `CHANGELOG.md` | 顶部追加条目 | 修改既有条目 |
| `companies/{vendor}.md` | 仅在文末 `## 🤖 Moltbot Updates` 区追加 | 修改既有正文、改结构 |
| `cheat-sheet/timeline.md` | 追加行（按时间倒序最新一行） | 修改既有行 |
| `cheat-sheet/sensor-budget-matrix.md` | **禁止自动追加** — 该文件需要人工 SWaP-C 核算 | — |
| `docs/*.md` · `AGENTS.md` · `CONTRIBUTING.md` · `LICENSE` · `README.md` | **❌ 不可触碰** | — |
| **其他所有文件** | **❌ 默认不可触碰** | — |

### Crossing 写入门槛（高严格度）

`crossing/` 是本仓的核心 USP。自动写入 `crossing/<lane>/<topic>.md` 必须同时满足：

1. **跨 ≥3 embodiment** — 内容明确比较 ≥3 个 embodiment 上的同一问题（不只 manipulation vs driving）
2. **每 embodiment 都附论文 / 一手来源** — 不允许"manipulation 部分跑通了"这种无来源叙述
3. **工程数字门槛** — 至少 1 个 cell 含具体 SWaP-C / 延迟 / 范围数字（哪怕标 `UNVERIFIED`）
4. **boundary 清晰** — 文末必须有「Boundary」段，指明 per-method 拆解去 `foundations/`、per-embodiment 实战去 `embodiments/`

不满足以上任意一条，**跳过自动写入，推送告警**让维护者补足。

### 条件触发：sensor 物理 / 硬件文档同步

`foundations/sensor-physics/*.md` 和 `deployment/hardware-selection/*.md` 不接受自动追加。这些文档需要人工核对数据手册数字，Moltbot 仅允许在文末 `## 🤖 Moltbot Updates` 段以"日期 + 一句话事件 + 一手来源 URL"格式追加发布动态。

### Commit Message 规范

emoji 前缀，便于 `git log` 区分人工与自动：

| 来源任务 | 格式 | 示例 |
|---|---|---|
| 每日论文 | `📄 daily papers: {日期} (+N papers)` | `📄 daily papers: 2026-05-21 (+4 papers)` |
| SOTA 追踪 | `📈 benchmark: {model} on {benchmark}` | `📈 benchmark: VGGT on ScanNet++` |
| Release 追踪 | `🔧 release: {source} — {event}` | `🔧 release: NVIDIA Cosmos — Cosmos-1.1 released` |
| 周报 | `📊 weekly: {起} → {止}` | `📊 weekly: 2026-05-15 → 2026-05-21` |
| 双周报 | `📊 biweekly: {起} → {止}` | `📊 biweekly: 2026-05-08 → 2026-05-21` |
| 代码分析 | `📝 code analysis: {项目}` | `📝 code analysis: VGGT-distilled` |
| 索引更新 | `📋 update index: {文件}` | `📋 update index: reports/weekly/README.md` |
| Cross-embodiment 综合 | `🔭 crossing: {topic}` | `🔭 crossing: depth-foundation-across-scales` |

人工提交不使用以上前缀。

### 失败处理

- GitHub API 非 2xx：不重试、记录、不影响主任务
- 409 conflict：放弃本次写入、推送告警
- 401/403：推送告警「GitHub Token 可能已过期」
- 格式异常（表格列数不匹配）：跳过、推送告警

---

## 标注系统

继承 VLA-Handbook 三级 + 方向标记。**新增 sensor / wedge 标记**：

### 重要性标注

| 标记 | 含义 | 入选条件 |
|---|---|---|
| ⚡ | 战略级 | 知名团队 + 明显方法论创新 / benchmark SOTA / 跨 embodiment 范式转移 |
| 🔧 | 可操作 | 有代码 / 数据 / 协议可复现 |
| 📖 | 值得了解 | 有参考价值但不需立即行动 |
| ❌ | 不收录 | 学术堆砌 / 与本仓边界无关 / 无 spatial 角度 |

### Spatial 专用方向标记

| 标记 | 含义 |
|---|---|
| 🛰️ | 跨 embodiment 论文（在 ≥2 embodiment 上做实验或显式讨论跨 embodiment 迁移）— `crossing/` 候选 |
| 🌬️ | 维护者锚定方向（drone / aerial），同 VLA 的 🎯 |
| `[3DGS]` `[VGGT]` `[VIO]` `[Sensor]` `[BEV]` `[WorldModel]` | team 方向标签 |
| `📡 sensor-physics` | sensor 物理 / 硬件类内容（hardware-selection 候选）|

🛰️ 与 ⚡/🔧/📖 组合使用，例如 `🛰️🔧 [VGGT] vggt_vs_drone_vio dissection`。

### 内容来源标注

| 标记 | 含义 |
|---|---|
| ⚙️ 本文由 Moltbot 自动生成 \| {日期} | 纯自动生成 |
| ⚙️ 初稿由 Moltbot 自动生成 \| {日期} \| 经人工编辑 | 自动 + 人工修订 |
| ✍️ | 人工撰写（Moltbot 不触碰） |

---

## 内容与格式规范

- **语言**（2026-05-21 update）：中文 = **简体中文**（不接受繁体）。
  - `foundations/` `embodiments/` `deployment/` `companies/` `cheat-sheet/` → **偏简体中文 narrative**（technical terms / paper names / model names / arXiv IDs / GitHub URLs / 公式符号 全部保留英文原样）
  - `crossing/` → **优先英文**（学术圈通用 USP，国际读者也读得懂）
  - `bridge-to-vla/` → 与 VLA-Handbook 接口，mixed OK，保持单文档语言一致
  - `benchmarks/` → mixed 视既有风格
  - 所有文档**双语标题**：`# Chinese Title (English Title)` 永远必须
- **文件名**：`snake_case.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sou350121/Spatial-Intelligence-Handbook](https://github.com/sou350121/Spatial-Intelligence-Handbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
