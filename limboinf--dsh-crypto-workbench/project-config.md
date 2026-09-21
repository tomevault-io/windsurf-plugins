---
trigger: always_on
description: 合约实验室：主流币 USDT 永续的 AI 辅助交易工作台（受控执行，2026-09-11 用户拍板开放下单），以插件形式挂载到 DeepSeek Harness（dsh），
---

# dsh-crypto-workbench —— Agent 工作指引

合约实验室：主流币 USDT 永续的 AI 辅助交易工作台（受控执行，2026-09-11 用户拍板开放下单），以插件形式挂载到 DeepSeek Harness（dsh），
**不改动 dsh 核心代码**。技术方案：`docs/合约实验室技术实现方案-20260911.md`（实现即按它落地）；
产品边界：`docs/加密货币工作台可行性与MVP方案-20260911.md` §8.1–8.3。

- dsh 源码仓库（本机 clone，路径自行替换；需先 `pnpm install` + `pnpm run build:lib`）
- 插件开发官方文档：https://deepseek-harness.github.io/deepseek-harness/develop/basic
- 仓库根 README.md 有完整的接入/验证命令，改动部署相关内容时先同步读它

## 常用命令

```bash
npm run typecheck   # tsc --noEmit
npm test            # node --import tsx --test test/*.test.ts（node:test，无 vitest/jest）
npm run live-check  # 联网走 OKX 公共 REST：BTC/ETH 行情 + USDT-CNY 汇率（账户/消息需在 dsh 里验）
npm run build       # tsdown → lib/index.js（host）+ lib/client.js（client）
```

## 目录结构

```
src/index.ts             插件入口（export name/inject/apply）：装配传输（dsh MCP 主 + REST 兜底）、
                         OkxReadonlyClient、NewsAdapter、SnapshotBuilder、CryptoStore，再 registerCryptoTools
src/crypto-dto.ts        面板数据契约（host ⇄ client 唯一载荷）：纯类型 + 纯函数、零 import。
                         EXPERIMENT_PROFILE 固定实验参数；<!--crypto:kind …--> 标签编解码
src/risk.ts              风险引擎纯函数：指标公式、闸门阈值（GATE_THRESHOLDS）、合约规格
                         （CONTRACT_SPECS）、仓位计算 buildTradePlan（executable=风控是否通过）、planToInput 执行前重算
src/crypto-normalize.ts  OKX/消息原始 JSON → DTO：信封解析、账户/仓位/行情规范化、消息分类/等级/ID
src/okx-adapter.ts       适配层：ToolTransport（dsh ctx.tools.execute / 公共 REST / 兜底组合）、
                         Result<T,SourceError>、READONLY_TOOLS + OkxReadonlyClient、WRITE_TOOLS + OkxTradeClient
src/execute.ts           执行链路：三道门复核 → 设杠杆 → 限价单附止损/止盈 → 回执 → 成交 import → 执行日志；平仓
src/news-adapter.ts      消息源（OKX news / OKX 日历 / CRYPTO_NEWS_TOOL 外部源）+ 去重 + 时间线
src/opportunity.ts       机会雷达（只读扫描面）：24h/3d/7d 三窗全部同向才给方向，四因子评分
                         （趋势强度 35/动量质量 20/资金费健康度 25/区间位置 20）≥70 才显示；
                         扫描面宽（对话可配：crypto_radar_config → radar.json，允许全集
                         RADAR_ALLOWED_COINS，观察币种上限 8）执行面=同一份白名单（2026-09-12 拍板放开，BTC/ETH 固定在列）；
                         RADAR_REFRESH_MS=60s 独立计时，不随 5s 快照轮询重取（2026-09-12 重做）
src/jev-service.ts       Jev 决策建议层（2026-09-18 接入）：OpenRouter ~typesafe/jev-latest 的 choice
                         客户端（fetch+超时+fail-open）+ 5s 行情环形缓冲（内存不落盘）+ 语义桶
                         （趋势按 2σ 噪声带宽判档——数字只算一次，不喂裸价格序列）+ watch 状态机
                         （tick fire-and-forget、陈旧响应丢弃、conf<0.5 一律观望）；决策动作转变时
                         写 decision-logs（kind=note，tags 带 jev）；纯建议层，不进执行链
src/jin10-mcp.ts         金十标准 MCP HTTP 客户端：Bearer、握手/发现、session、SSE/JSON、structuredContent
src/jin10-service.ts     quote://codes 与报价/快讯/资讯/日历的聚合和字段规范化
src/jin10-dto.ts         金十 tab 独立载荷契约（零 import）与 <!--jin10:payload …--> 编解码
src/jin10-tools.ts       5 个 crypto_jin10_* 工具 + /jin10 影子命令
src/snapshot.ts          SnapshotBuilder：并行取数 + 分段 TTL 缓存 + 失败保留上一份 + 闸门叠加（快照只在内存流转，不落盘）
src/store.ts             本地仓储：experiment.json / daily / plans / fills / decision-logs，
                         原子写 + 记录校验 + JSONL fail-closed
src/crypto-render.ts     payload → markdown（模型/人读）；数字只在这里格式化一次
src/crypto-tools.ts      13 个工具（OKX/交易 + crypto_radar_config 雷达配置 + crypto_jev_watch Jev
                         监控开关，含 crypto_execute_plan / crypto_cancel_plan_order /
                         crypto_close_position） + /crypto（含 jev / jevchart 子命令）、
                         /crypto-activity 命令（裸 JSON-Schema）
src/env.ts               数据目录与 env 文件解析（含金十 URL / Bearer Token / 默认报价品种 / OpenRouter Key）
src/client/index.ts      client 入口：shell.overlay 面板、tool.call.toolview 工具行、input.left 快捷按钮
src/client/channel.ts    影子会话取数（executeCommandText）+ 对话联动（linkSend）+ ctx 最小类型
src/client/side-panel.ts 右侧停靠面板壳（拖拽调宽、开合记忆、对话列让位 CSS）
src/client/crypto-panel.ts 面板体七区（总览/仓位/委托/行情/闸门/消息/计划） + 消息线 OKX/金十双 tab
                         + 币种详情视图（点行情卡片进入：SVG 走势+决策点、语义桶、Jev 决策卡、
                         决策流水；Esc/返回退出；走势种子走 /crypto jevchart、监控开关走 /crypto jev）
src/client/jin10-panel.ts 金十 tab：切入时按需调用 /jin10，不参与 5s 轮询
skills/crypto-briefing/  简报规范（事实/推断/行动分段，数字只抄工具）
skills/crypto-risk/      风险审查规范（第二道闸：可否决，不可推翻引擎）
test/                    node:test；fixtures.ts 是 OKX 1.4.6 信封样本，helpers.ts 是可编程假传输
scripts/live-check.ts    REST 兜底链路联网自检
cordis.dev.patch.yml     开发模式：OKX MCP（market,account,news,swap）+ 源码直载（command/profile 是占位符；
                         本机实际值放 cordis.dev.local.patch.yml，已 gitignore，启动用它）
cordis.patch.yml         安装模式：只插本插件，MCP 由用户 profile patch 另挂
```

## 设计纪律（改代码前必读，违反即破坏本项目核心原则）

1. **零运行时依赖**：不得 import 任何 `@deepseek-ai/*` 运行时符号。工具用裸 JSON-Schema 注册，
   ctx 类型在 `src/index.ts` / `src/okx-adapter.ts` / `src/client/channel.ts` 里自声明（刻意宽松）。
2. **执行三道门**：写操作只走 `OkxTradeClient`（WRITE_TOOLS：永续设杠杆/下单/撤单/平仓/查单/查成交，无现货/期权/划转）；
   `crypto_execute_plan` 必须同时满足：用当前快照重算的 blockers 为空、计划 status=confirmed、`confirm=true`，
   且最新价偏离入场参考价 ≤1%、计划未过期。用户只说「确认」不等于「执行」，模型不得替用户说这两句。
   `test/execute.test.ts` 锁死了这个顺序，别绕。模型直连 `mcp__okx__*`：写动词（isOkxWriteTool）全局 guard 拒绝、
   读工具放行（查单/成交/行情核验回执用）；未知形态 fail-closed。想退回只读观察：patch 加 `--read-only`，执行工具自动报通道未启用。
3. **数字只算一次**：所有金额/杠杆/回撤/强平距离/张数在 host 确定性层算完进 DTO；client 只格式化，
   模型只解读。闸门由 `risk.ts` 算死，模型与前端不能覆盖。
4. **不猜数**：除数 ≤ 0 返回 null；数据过期可以展示但 `blocked`；账户 `partialFailure` 直接不出净值；
   「无消息」不是信号；成交只能人工录入或从 OKX 成交流水 import（按 tradeId 幂等）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [limboinf/dsh-crypto-workbench](https://github.com/limboinf/dsh-crypto-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
