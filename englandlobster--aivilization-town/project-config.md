---
trigger: always_on
description: 本文件适用于整个仓库。所有 AI Agent 和开发者在分析、设计、修改、测试或评审代码前，必须先遵守本文件。子目录如存在更具体的 `AGENTS.md`，其规则可以补充本文件；发生冲突时，以离目标文件最近的规则为准，但不得破坏这里定义的全局架构边界和数据一致性要求。
---

# AIvilization Town 工程与架构约束

本文件适用于整个仓库。所有 AI Agent 和开发者在分析、设计、修改、测试或评审代码前，必须先遵守本文件。子目录如存在更具体的 `AGENTS.md`，其规则可以补充本文件；发生冲突时，以离目标文件最近的规则为准，但不得破坏这里定义的全局架构边界和数据一致性要求。

## 1. 核心目标

本项目是一个长期演进、事件驱动、AI-native 的社会与经济模拟系统。架构决策必须优先保障：

1. **正确性**：领域不变量、资金守恒、事件回放、模拟时间语义必须可证明。
2. **可维护性**：一个业务概念应有明确且唯一的归属，避免规则散落。
3. **可扩展性**：新增企业、市场、金融、治理、社会机制时，应通过稳定边界扩展，而不是继续扩大中心 switch 或共享状态。
4. **高内聚、低耦合**：同一领域规则聚合在一起；跨领域只通过明确的端口、命令、领域事件或集成事件协作。
5. **确定性与可回放性**：相同初始状态、事件和策略版本必须得到相同结果。
6. **兼容性**：已有事件流、快照、manifest 和公开 API 不得被无意破坏。
7. **可观测性**：重要决策、拒绝原因、策略版本和跨账户流动必须可审计。

不要把“快速把功能塞进去”视为完成。一个功能只有在领域归属、依赖方向、不变量、回放和测试都成立时才算完成。

## 2. DDD 使用原则

DDD 在本项目中用于保护复杂业务规则，不是为了制造目录、类或样板代码。

### 2.1 必须先确定业务归属

开始实现前，必须回答：

- 这个概念属于哪个 bounded context？
- 哪个聚合拥有它的状态？
- 哪个聚合负责维护它的不变量？
- 这是领域规则、应用编排、基础设施还是只读投影？
- 它产生领域事件还是跨上下文集成事件？
- 策略变化是否会改变历史事件的重放结果？

如果无法明确回答，不要直接向 `world`, `worker` 或大型 projection switch 增加逻辑。先定义边界。

### 2.2 聚合规则

- 聚合是事务一致性和业务不变量边界，不是任意数据集合。
- 聚合之外的代码不得直接发明聚合状态转换。
- 聚合对外优先暴露纯决策函数：`state + command + policy -> domain events | rejection`。
- 状态更新由统一的 domain event applier/reducer 完成。
- 一个命令需要修改多个聚合时，由应用层编排；不得为了方便把多个领域合并成巨型聚合。
- 聚合之间通过 ID 引用。除只读快照外，不应持有另一个聚合的完整可变对象。
- 聚合拒绝必须给出稳定、可测试、可审计的原因。

### 2.3 领域模型不能贫血化

如果一个实体存在状态机或重要不变量，例如企业偿付能力、股权、债务、招聘、住房等级，则相关规则必须位于所属领域模块，而不是全部堆在 handler 或 projection 中。

可以使用 TypeScript 数据类型和纯函数，不强制面向对象类。但必须保证：

- 状态结构、策略、决策函数和事件应用函数在同一领域边界内高内聚。
- 应用层只做授权、跨聚合读取、流程编排和事件映射。
- projection 只重建状态，不重新计算业务决策。

## 3. 当前 bounded contexts

### `@aivilization/sim-core`

拥有通用模拟原语：ID、命令/事件 envelope、时间、分区、事件存储基础能力。

- 不得依赖任何上层领域包。
- 不应包含企业、税收、住房等具体业务规则。
- 全局命令/事件类型注册可以放这里，但 payload 和业务语义属于对应领域或 `world` 集成层。

### `@aivilization/economy`

拥有经济基础原语和算法：

- 账户、复式交易和货币供给分类；
- Inventory 数学；
- AMM、价格、估值；
- 生产、配方与生产链；
- 外部市场流动性算法；
- 镇外贸易定价（滚动净出口平衡 + √balance 冲击，注入/销毁由 world 结算）。

这里不拥有 Agent、企业生命周期、政府或世界调度。API 应保持纯函数、无存储依赖、无 `world` 依赖。

### `@aivilization/enterprise`

拥有企业聚合及其生命周期：

- 企业现金、库存和经营累计；
- 所有权份额与留存收益；
- 员工成员关系和容量；
- 偿付能力、宽限期、破产、关闭；
- 可分配利润和分红规则。

企业领域不得依赖 `world`、worker、API、文件系统或数据库。业务决定返回领域事件，由 `world` 映射成持久化集成事件。

新增贷款时，不要默认把整个银行系统塞入 Enterprise。若存在贷款人、还款计划、抵押品和违约生命周期，应建立独立 credit/finance bounded context；Enterprise 只保留影响自身偿付能力的引用或义务。

### `@aivilization/credit`

拥有单一权威"镇银行"聚合及其信贷生命周期：

- 银行现金账户（流通账户）、存款台账（负债）与贷款账簿；
- 存款利率支付、贷款日结息、摊还自动扣款（先息后本）、missed-payment 计数与违约；
- 按信用记录演化的贷款额度（违约减半、还清加成、上下限 clamp）；
- 并发贷款上限与准备金约束（放款后银行现金 ≥ 存款总额 × reserveRatio）。

credit 领域为纯决策函数与领域事件，不得依赖 `world`、worker、API、文件系统或数据库。一切资金变动都是银行与 Agent 流通账户之间的 transfer，不得改变 moneySupply；银行初始准备金来自 scenario 种子 `initialBankReserves` 并计入种子 moneySupply。跨账户扣款（借款人余额读取与记账）由 `world` 应用层编排。

### `@aivilization/society`

拥有社会与家庭侧规则：生理、消费、生活方式、教育、职业资格、招聘、工资政策、税收、福利、住房（含区域地价指数定价）和公共预算决策。

- 纯规则放在这里。
- 不读取 WorldProjection，不负责事件持久化。
- 与企业或市场交互时返回决定或金额，由应用层完成账户转移。

### `@aivilization/memory`、`@aivilization/agent-runtime`、`@aivilization/llm`

- `memory` 拥有认知记录、检索、反思和长期画像。
- `agent-runtime` 拥有目标、计划、action synthesis、修复和 Agent 决策过程。
- `llm` 是模型提供方和结构化生成基础设施。
- Agent 的“想做什么”不能绕过 World/Domain 的权威校验。
- 决策上下文是只读模型，不得成为领域真相来源。

### `@aivilization/world`

`world` 是服务器权威的应用与集成边界，负责：

- 命令分派和跨聚合编排；
- 身份、位置、忙碌状态等世界级授权；
- 领域事件到 durable world event 的映射；
- simulation time cadence 调度；
- 事件回放和组合 read projection。

`world` 不应继续吸收可以独立表达的领域算法。新增复杂领域时，应先创建/扩展领域包，然后在 `world` 建立 adapter。

### `@aivilization/observability` 与 apps

- `observability` 只负责指标、轨迹、报告和验证，不反向控制领域真相。
- `apps/worker` 负责运行时组装、调度、策略解析、Agent context adapter。
- `apps/api`、`apps/server`、`apps/web` 是输入输出或部署 adapter。
- apps 不得复制领域公式或通过 UI/LLM 直接修改 projection。

## 4. 依赖方向

依赖必须从外层指向内层。当前允许的核心方向：

```text
sim-core
├── content
├── credit
├── economy
├── llm
├── observability
└── society (also depends on content)

economy + sim-core
└── enterprise

credit + economy + enterprise + memory + sim-core + society
└── world

llm + memory + sim-core + society
└── agent-runtime

domain packages + world + agent-runtime
└── worker / server / API / web adapters
```

强制要求：

- 禁止 `economy`、`enterprise`、`credit`、`society` 依赖 `world`。
- 禁止任何 domain package 依赖 apps。
- 禁止通过相对路径跨 package 导入内部文件；使用公开 package export。
- 新依赖必须说明为什么现有端口无法满足，且不得形成循环依赖。
- 为共享类型新增依赖前，先判断类型应归属 `sim-core`、领域包还是集成层；不要建立无归属的 `common` 大杂烩。

## 5. 分层职责

### Domain 层

负责：

- 值对象和聚合状态；
- 领域策略和不变量；
- 决策函数；
- 领域事件；
- 领域状态机。

不得负责：文件 I/O、网络、定时器、全局 projection、事件序号、UI、LLM prompt。

### Application 层

负责：

- 查找参与命令的聚合；
- 调用领域决策；
- 跨聚合授权和流程协调；
- 将 domain event 映射成 integration event；
- 保证一个命令产生的事件序列是原子的。

不得重新实现领域公式或偷偷修正领域状态。

### Infrastructure / Adapter 层

负责：事件存储、快照、文件、HTTP、worker、LLM provider、进程与部署。

不得把存储格式或供应商 SDK 类型泄漏进领域模型。

### Projection / Read Model

负责从事件确定性地重建查询状态。

- reducer 不得调用随机数、当前系统时间或外部服务。
- reducer 不得决定“是否应该发生”；这个决定必须已记录在事件中。
- 大型 projection 必须按 bounded context 拆为 reducer，并由组合入口调用。
- read model 可以为查询冗余数据，但不得成为写模型的不变量来源。

## 6. 命令、事件与时间语义

- Command 表示意图，Event 表示已经发生的事实。
- Event 名称使用过去式，payload 包含回放所需的最终事实。
- 历史事件不可改变语义；若新规则影响重放结果，应增加 policy/event schema 版本或兼容分支。
- 新字段优先设计为可兼容的 optional 字段，并为旧快照提供明确 normalization。
- 不得依赖 `Date.now()` 决定模拟领域结果；使用 simulation clock。
- 随机决策必须使用确定性种子，种子输入需要稳定并可审计。
- 跨越多个 cadence 的时间推进必须逐个补算非线性、概率性或有状态效果。
- 只有严格可加的效果才允许合并计算；必须有等价性测试。
- cadence scheduler 只负责枚举边界并调用领域决策，不承载领域公式。

## 7. 经济与会计不变量

任何余额变化必须属于以下之一：

1. 国内账户之间的转移；
2. monetary authority 注入；
3. 向非流通账户销毁/移出；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnglandLobster/aivilization-town](https://github.com/EnglandLobster/aivilization-town) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
