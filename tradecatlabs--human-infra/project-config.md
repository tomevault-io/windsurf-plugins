---
trigger: always_on
description: Human Infra 是以人类任务执行为中心的基础设施项目。根目录只承载总项目入口、稳定理论文档和子域路由，不直接承载某一个具体产品的全部数据与脚本。
---

# Human Infra 架构说明

Human Infra 是以人类任务执行为中心的基础设施项目。根目录只承载总项目入口、稳定理论文档和子域路由，不直接承载某一个具体产品的全部数据与脚本。

## 目录结构

```text
human_infra/
├── .github/              # GitHub 协作模板与远程质量门禁
│   ├── AGENTS.md         # .github 目录说明
│   ├── ISSUE_TEMPLATE/   # issue 模板
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── workflows/        # GitHub Actions 工作流
├── AGENTS.md              # 记录总项目架构、职责边界和维护规则
├── human-infra-dashboard.html # 静态 Web 看板：展示主体持续性与生命路径预测模型
├── singularity-human-infra.html # 专项 Web 展示页：将《奇点更近》学习资料转译为 Human Infra 价值与预测模型
├── web/                   # Observable Framework 正式 Web 应用：科研叙事、预测模型和交互图表
│   ├── AGENTS.md          # web 目录说明
│   ├── README.md          # 本地运行、构建和页面职责
│   └── src/               # Framework 页面、组件、样式和展示数据
├── tools/                 # 仓库检查脚本和可复用工程工具
│   ├── AGENTS.md          # tools 目录说明
│   ├── check_repository.py
│   ├── arxiv_html_paper_tool.py
│   └── arxiv-html-paper/  # arXiv HTML papers 复用模板包
├── README.md              # Human Infra 定位、边界、目录和变更日志
├── docs/                  # 总理论、域地图、伦理边界和原始资料归档
│   ├── AGENTS.md          # docs 目录说明
│   ├── README.md          # docs 入口和阅读顺序
│   ├── decisions/         # 架构与域边界决策记录
│   ├── explanations/      # 概念解释和理论文章
│   ├── how-to/            # 任务导向操作说明
│   ├── reference/         # 域地图、伦理边界、术语和标准
│   ├── source-notes/      # 最新原始资料的改名归档
│   └── tutorials/         # 学习路径
└── domains/               # 可独立演化的子域
    ├── AGENTS.md          # 子域目录维护规则
    ├── antimicrobial-resilience/ # 抗微生物韧性
    ├── assistive-technology-access/ # 辅助技术与无障碍
    ├── ai-agency-safety/  # AI 代理安全
    ├── attention-executive-control/ # 注意力与执行控制
    ├── biostasis-cryopreservation/ # 生物停滞与冷冻保存
    ├── cancer-control/    # 癌症控制
    ├── cardiovascular-resilience/ # 心血管韧性
    ├── cellular-senescence-clearance/ # 细胞衰老与清除
    ├── cellular-reprogramming/ # 细胞重编程与表观遗传年轻化谱系
    ├── gene-therapy-genome-editing-delivery-safety/ # 基因治疗、基因编辑与递送安全
    ├── biological-age-clocks-biomarker-validation/ # 生物年龄钟与标志物验证
    ├── multiomics-personal-baseline-systems-biology/ # 多组学个人基线与系统生物学
    ├── engineered-cell-therapy-regenerative-platforms/ # 工程细胞疗法与再生平台
    ├── organoids-organ-on-chip-disease-models/ # 类器官、组织芯片与疾病模型
    ├── xenotransplantation-bioengineered-organ-replacement/ # 异种移植与工程器官替换
    ├── causal-inference-target-trial-emulation/ # 因果推断与目标试验模拟
    ├── survival-analysis-healthspan-risk-modeling/ # 生存分析与健康寿命风险建模
    ├── human-digital-twin-life-course-simulation/ # 人类数字孪生与生命历程仿真
    ├── implementation-science-adherence-behavior-change/ # 实施科学、依从性与行为改变
    ├── uncertainty-quantification-model-calibration/ # 不确定性量化与模型校准
    ├── data-quality-missingness-representativeness/ # 数据质量、缺失与代表性
    ├── privacy-preserving-computation-federated-learning/ # 隐私保护计算与联邦学习
    ├── knowledge-graph-ontology-semantic-interoperability/ # 知识图谱、本体与语义互操作
    ├── model-cards-ai-audit-documentation/ # 模型卡与 AI 审计文档
    ├── technology-foresight-horizon-scanning/ # 技术预见与地平线扫描
    ├── research-portfolio-prioritization-funding-governance/ # 研究组合优先级与资金治理
    ├── intellectual-property-technology-transfer-access/ # 知识产权、技术转移与可及性
    ├── trustworthy-scientific-communication-peer-review/ # 可信科研传播与同行评审
    ├── research-participant-consent-community-engagement/ # 研究参与者同意与社区参与
    ├── biobanking-biospecimen-quality-chain-of-custody/ # 生物样本库、样本质量与保管链
    ├── longitudinal-cohort-retention-followup-infrastructure/ # 纵向队列留存与随访基础设施
    ├── registries-real-world-data-governance/ # 登记系统与真实世界数据治理
    ├── scientific-instrumentation-sensor-calibration-maintenance/ # 科研仪器与传感器校准维护
    ├── nanomedicine-targeted-delivery-molecular-repair/ # 纳米医学、靶向递送与分子修复
    ├── ai-drug-discovery-protein-design/ # AI 药物发现与蛋白设计
    ├── epigenetic-editing-gene-regulation-therapeutics/ # 表观遗传编辑与基因调控疗法
    ├── brain-preservation-connectomics-emulation/ # 脑保存、连接组与仿真假设
    ├── post-quantum-cryptography-long-term-data-security/ # 后量子密码与长期数据安全
    ├── safety-critical-software-formal-methods/ # 安全关键软件与形式化方法
    ├── long-term-digital-preservation-format-migration/ # 长期数字保存与格式迁移
    ├── cerebrovascular-stroke-brain-perfusion-resilience/ # 卒中、脑灌注与脑血管韧性
    ├── traumatic-brain-injury-neurotrauma-recovery/ # TBI、脑震荡与神经创伤恢复
    ├── delirium-acute-cognitive-failure-prevention/ # 谵妄与急性认知失败预防
    ├── disorders-of-consciousness-coma-recovery/ # 意识障碍、昏迷与恢复边界
    ├── epilepsy-seizure-network-stability/ # 癫痫、发作与脑网络稳定
    ├── autonomic-nervous-system-homeostasis/ # 自主神经系统稳态
    ├── migraine-headache-effective-time-burden/ # 偏头痛、头痛障碍与有效时间负担
    ├── vision-eye-health-continuity/ # 视觉与眼健康连续性
    ├── hearing-auditory-communication-continuity/ # 听力与听觉沟通连续性
    ├── vestibular-balance-spatial-orientation/ # 前庭、平衡与空间定向
    ├── speech-language-communication-continuity/ # 言语、语言与沟通连续性
    ├── swallowing-dysphagia-aspiration-nutrition/ # 吞咽、误吸与营养连续性
    ├── smell-taste-chemosensory-continuity/ # 嗅味觉与化学感知连续性
    ├── peripheral-neuropathy-somatosensory-continuity/ # 周围神经病变与躯体感觉连续性
    ├── space-weather-geomagnetic-storm-resilience/ # 空间天气与地磁暴韧性
    ├── planetary-defense-near-earth-object-risk/ # 行星防御与近地天体风险
    ├── volcanic-ashfall-geohazard-continuity/ # 火山灰与地质灾害连续性
    ├── earthquake-seismic-risk-built-environment-continuity/ # 地震、抗震与建成环境连续性
    ├── tsunami-warning-coastal-evacuation-continuity/ # 海啸预警与沿海撤离连续性

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tradecatlabs/human_infra](https://github.com/tradecatlabs/human_infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
