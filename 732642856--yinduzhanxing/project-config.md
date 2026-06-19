---
trigger: always_on
description: 印度占星（Jyotish）专业解盘与推运系统。核心能力：PDF星盘输入→严谨解盘→精确推运应期输出。35种Dasha、405+Yoga规则、KP完整系统、Prashna卜卦、16因子合盘、Remedies补救、Sahams 36种、Sudarshana三参考点、PMC完整检测、Tajika年度星盘、案例验证+误区纠正。触发词：印度占星、吠陀占星、Jyotish、解盘、推运、星盘分析、Dasha、Transit、Nakshatra、Yoga。GitHub: https://github.com/732642856/yinduzhanxing
---


# 印度占星专业解盘与推运系统

> **版本**：v6.9.14 | **详细变更**：`CHANGELOG.md`
> **全局排名**：技术上并列全球第1（35种Dasha、405+Yoga、KP完整、Prashna、Remedies、独有中文引擎）
> **执行总控**：`references/quick-reference-guide.md`
> **严格路由**：`references/strict-workflow-router.md`（涉及事业/婚恋/财务/应期/技法验证时必须优先读取）
> **机器注册表**：`references/technique_registry.json` + `scripts/audit_capabilities.py`

## v6.9.14 核心能力

| 维度 | 数据 |
|------|:--:|
| Dasha系统 | 35种（含Vimshottari/Chara/Kalachakra/Narayana/Yogini等） |
| Yoga规则 | 405+条（BPHS数据驱动架构，Yoga精度Benchmark 100%） |
| 分盘 | D1-D144 + D2/D3变体 + 复合D-m×n + 自定义D-N(2-300) |
| Bhava Chalit | Sripati/Porphyry/Equal/Whole Sign/Placidus/Koch 不等宫位调整 |
| Sudarshana | Asc/Moon/Sun 三参考点盘 + 宫位收敛分析 |
| Shadbala | 1200/1200 Virupas校准（6维力量评估） |
| Ashtakavarga | BAV+SAV+PAV（展开式）+Sodhita（净化式） |
| KP系统 | Sublord+Subsublord+ABCD Significator |
| 合盘 | 16因子36分制（Ashtakoot+Kuta） |
| 补救 | 5类（宝石/咒语/捐赠/斋戒/Dosha专项） |
| 自动化测试 | 475个 pytest 用例全通过 + run_all 100项 |
| Git commits | v6.1.12→v6.9.14 持续推进 |

**独有能力**：中文AI解读引擎、Career/Love结构化分析、验前事反推管道、误区自动纠正、名人+普通人案例双轨验证。

## Yoga 逻辑验证指标

| 指标 | v6.0.45（旧基线） | v6.9.14（当前） |
|---|---:|---:|
| Precision | 83.26% | **96.48%** |
| Recall | 91.52% | **93.99%** |
| **F1 Score** | 87.19% | **95.22%** |
| 规则库 | 82条 | **405条** |
| Yoga精度Benchmark | — | **100%** (8/8) |

---

## ⚠️ 核心定位

**三种输入 → 严谨解盘 → 精确推运应期输出**

| 路径 | 用户输入 | AI行为 |
|------|---------|--------|
| **A：精准出生信息** | 日期+时间+地点 | `full-reading` 引擎全链路计算 |
| **B：PDF/文字星盘** | PDF/详细文字描述 | 提取数据+Quality Gate → `references/pdf-chart-reading-guide.md` |
| **C：时间不明确** | "不知道几点出生" | 互动式出生时间矫正 → 确认后走路径A |

**强制工作流**（完整规范 → `references/ai-reading-workflow-prompt.md` v3.0）：

0. **阶段负一**：问题类型路由（事业/婚恋/财务/应期/历史验证/综合解盘）→ 必须先读 `references/strict-workflow-router.md`，按对应 strict checklist 执行；用户不需要主动点名高级技法。
1. **阶段零**：入口路由（A/B/C自动判断）
2. **阶段一**（仅B）：PDF/图片提取 + Quality Gate
3. **阶段二**：意图识别 → 路由目标宫位（无明确意图→Level 2综合解盘）
4. **阶段三**：静态分析10步（宫位→承诺→Yoga→Argala→逆行→NK→Shadbala→AV→Ketu→分盘）
5. **阶段四**：动态推运7步（Dasha→五系统Convergence→Transit→Double Transit→Jaimini→KP→Varshaphala）
6. **阶段五**：应期输出（五层验证→时间窗口→Actionable Output+案例检索）
7. **阶段六**：补救措施（可选）
8. **阶段七**：现代措辞包装
9. **阶段八**：输出 Technique Audit Table，逐项声明已调用/未调用/部分可用/缺失模块及其对置信度的影响。

---

## ⚠️ 强制规则（与"不跳步"同级）


### 用户隐私与个案资料隔离（v6.0.4-privacy）

**严禁把真实用户个人信息写入 skill 文件或公开仓库。**

包括但不限于：姓名/称呼、出生日期时间地点、星盘度数、人生事件、关系状态、职业经历、项目背景、历史回测结论、当前会话中的个案分析。

允许的资料来源只有三类：
1. 公开 AA 级名人案例；
2. 明确标注为虚构的 smoke test / template；
3. 用户在当前会话中主动提供的数据，但只能在当前会话中使用，不得持久化到 skill、tests、CHANGELOG 或公开仓库。

如需沉淀方法论，只能抽象为通用规则，不得保留可识别个人轨迹的细节。

### Strict Workflow Router（v6.0.1-orchestration）

**凡是用户询问事业、婚恋、财务、事件应期、历史回测或技法可靠性，必须先读取 `references/strict-workflow-router.md`。**

核心要求：
1. 先判断问题类型，再自动选择 `career-timing-strict` / `relationship-timing-strict` / `wealth-timing-strict` / `event-timing-strict` / `event-verification-strict`。
2. 用户不需要知道 Chara Dasha、A10、Argala、Shadbala、Ashtakavarga 等技法名称；AI 必须按问题类型自动调用。
3. 输出末尾必须给出 Technique Audit Table，说明每项高级技法是否调用、结果是什么、缺失会如何降低置信度。
4. 不得把未实现或未调用的技法静默省略；A10/Karma Pada、Pushkara、Vargottama、Dasha Sandhi 已进入 full-reading 输出；Bhava Chalit 与 Sudarshana Chakra 已进入 complete，可正常纳入 Technique Audit Table。

### MEVG 强制外部验证门控（v4.2.0+）

**所有解读结论必须经过外部权威来源验证，禁止仅凭 AI 训练记忆输出。**

| 门控 | 位置 | 职责 |
|------|------|------|
| Step 3.11 | 静态分析后 | 验证 Yoga/尊严/Shadbala/SAV |
| Step 4.10 | 动态推运后 | 验证 Transit/Dasha/天文现象 |
| Step 5.5 | 预测输出前 | 确认每条预测有来源+置信度一致 |

**三步验证法**：V1 构建英文查询词 → V2 web_search ≥3个独立来源 → V3 交叉验证仲裁分歧

→ 完整协议：`references/mandatory-verification-gate-protocol.md`

### Transit Actionable Output（v4.1.0+）

**每条 Transit 预测必须输出三要素**：
1. **时间段**（精确到日/周/月）
2. **具体行动类型**（做什么）
3. **置信度** [A]=已验证 / [B]=高概率(3+维度) / [C]=推断(单一维度)

→ 完整规范：`references/transit-actionable-output-guide.md`

### Rahu/Ketu 节点口径冻结（v6.0.7-node-mode）

**所有 benchmark 与解盘输出必须显式声明 Rahu/Ketu 使用 Mean Node 还是 True Node。**

- 当前 skill 默认：`--node-mode mean`（Swiss Ephemeris Mean Node）。
- 可选：`--node-mode true`（Swiss Ephemeris True Node，用于对齐 PyJHora 默认口径）。
- PyJHora 4.8.6 的 `rasi_chart()` 默认使用 True Node；第三轮 benchmark 的 Rahu/Ketu 差异已由第四轮仲裁确认为 Mean/True Node 口径差异，不应再误判为 D9/D10 计算 bug。
- 输出 `birth_info.node_mode` 与 `node_mode_note` 必须保留，作为参数冻结证据。

### Ashtakavarga 口径冻结（v6.0.8-av-calibration）

**Ashtakavarga 默认使用 BPHS/PVR 书例校准口径，必须保留 SAV=337 与 full SAV=386 不变量。**

- `scripts/ashtakavarga.py` 当前为 v2.1：经第六轮 PyJHora/PVR 公开书例仲裁，校准 Moon/Venus 的 7 个贡献表项。
- 输出 `method` 应显示 `Ashtakavarga八分法（BPHS/PVR书例校准v2.1）`。
- benchmark 若与其他软件不一致，先比较贡献表项和 SAV 总量，不得直接把口径差异判为运行 bug。

### Chara Dasha 能力升级（v6.1.12 benchmark验证通过）

**Chara Dasha KN Rao Method 正式 benchmark 通过（95.83% ≥ 95%），可作为标准应期模块使用。**

- v6.1.12: PyJHora oracle benchmark **10案例×12星座=120对**: Sign 100%, Dur 91.67%, Overall 95.83% ✅ PASS
- v6.1.11: 重写为完整 KN Rao Method（序列基于第9宫方向，时长基于宫主所在宫位+尊贵调整）
- 剩余~4.2%差异: Aquarius/Scorpio 的 Rahu/Ketu 共主动态判定（需复制 PyJHora _stronger_planet_new）
- `jaimini` 输出中的 Chara Karaka、AK/AmK、Karakamsha 继续可用。

### Transit 真实过境冻结（v6.0.10-true-transit）

**full-reading 中的 Transit 多参考点分析必须使用真实过境行星位置，不得复用本命行星位置。**

- `modules.transit_positions` 必须输出 `data_layer: true_transit_positions`、`target_date`、`node_mode` 和 Swiss Ephemeris 计算的过境行星位置。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [732642856/yinduzhanxing](https://github.com/732642856/yinduzhanxing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
