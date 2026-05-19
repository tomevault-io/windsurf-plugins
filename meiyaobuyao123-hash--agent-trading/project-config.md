---
trigger: always_on
description: 立即读取以下所有 topic 文件，不得跳过：
---

# Project Memory — Agent-Trading (CLAUDE.md — 仓库版，无凭证)
# 凭证见本地 credentials.md，不在此文件中

## 🔴 每次会话开始必须执行（强制）
立即读取以下所有 topic 文件，不得跳过：
- `CLAUDE.md`（本文件）— 索引 + 规则 + 速查
- `architecture.md` — 架构、数据流、启动命令
- `pitfalls.md` — 踩坑记录
- `rules.md` — 工作规则详细版
- `sessions-log.md` — 历史会话记录 + 讨论结论 + 被否定方案

## ⚠️ 工作规则摘要
**记忆更新**: 发现新信息 → 立即更新，不等任务结束
**双份同步**: 本地 topic 文件 + 仓库 CLAUDE.md 必须同时更新，内容完全一致
**数据源**: EVM聪明钱用 `web3.okx.com`，SOL用 Helius WS，禁止 Etherscan，禁止 www.okx.com
**实现规则**: 讨论完先验证API → 实现后grep验证 → 不得悄悄换数据源
**诚实原则**: 没做就说没做，做了必须有证据同步用户，不得虚报完成
**用户偏好**: 中文输出，真实数据，不估时间，不过度工程化

---

## 快速速查

### 线上地址
- Portal: http://43.156.207.26（服务器部署，nginx反代:3000）
- Portal (Vercel备): https://agent-trading-portal.vercel.app/hot
- Backend API: http://43.156.207.26（nginx分流到:8000）
- GitHub: https://github.com/meiyaobuyao123-hash/Agent-Trading

### 本地路径
- 后端: `services/pump-scanner/`
- Flutter: `apps/app/`
- Portal: `apps/web/`

### 服务器
- IP: `43.156.207.26`（腾讯云轻量，新加坡，到期 2026-05-16）
- SSH: `ssh ubuntu@43.156.207.26`（密码见本地 credentials.md）

### Flutter 启动
```bash
flutter run -d DBC925B5-7657-4410-B770-F21E4605A9D6 \
  --dart-define=API_BASE_URL=http://43.156.207.26 \
  --dart-define=HELIUS_API_KEY=<见credentials.md>
```

---

## 当前功能状态
| 模块 | 状态 | 备注 |
|------|------|------|
| pump.fun 采集 | ✅ 线上 | 三阶段+实时信号池：score>=55动态进出，APP 30s轮询 |
| 热币扫描 | ✅ 线上 | OKX+GeckoTerminal 双源，毫秒级打分，进出榜单 |
| 聪明钱追踪 | ✅ 线上 | SOL ~400ms / EVM ~2.5s，三层供给（miner+TopHolders+Dune），1506地址 |
| KOL 舆情 | ✅ 线上 | 212 KOL，_evaluate_accuracy TODO |
| Agent 交易 | ✅ 线上 | Claude LLM + OKX DEX，SOL+EVM |
| Flutter App | ✅ 运行 | 模拟器 iPhone 17 Pro Max，i18n 4语言 |
| Portal | ✅ 线上 | 服务器部署(systemd+nginx)，Vercel备用 |
| i18n 国际化 | ✅ 完成 | zh/en/ja/ko，275+ 本地化字符串，语言切换器 |
| 合规 | ✅ | 免责声明Gate + CN IP屏蔽 + 推送限流 |
| XGBoost ML | ⏸ 待训练 | 管线就绪，3/27 提醒 |
| App Store 上架 | 🔄 进行中 | Build 2 已上传，待 Build 3（含新icon）+ 截图 + 提交审核 |
| Firebase 推送 | ⏸ 待配置 | 需创建 Firebase 项目 |
| docs/agent-pm/00-16 | 📄 仅产出 | Agent v1 优化 PM 设计文档（17 篇），**从未实施**，讨论时不要当 baseline。详见 `docs/memory/project_agent_pm_docs_status.md` |

## 待执行（手动）
- [ ] Supabase Dashboard 执行 `migrations/017_user_api_quota.sql`（如未执行）
- [ ] Firebase 项目创建 + 下载 google-services.json / GoogleService-Info.plist

---

## 系统架构摘要

### 热币数据源分层
- **发现** (10min): OKX toplist 多时间帧（4时间帧×2排序=8次/链），4链并行
- **刷新** (30s): DexScreener 批量
- **安全**: GoPlus；**SOL持仓**: Helius RPC；**打分**: M+Q+P 三维

### 聪明钱追踪
- SOL: Helius `accountSubscribe` WebSocket，~400ms
- ETH/BSC/Base: `GET https://web3.okx.com/api/v5/wallet/post-transaction/transactions-by-address`
  - 参数 `chains=`（List，不是chainIndex=），需 `User-Agent: Mozilla/5.0`
  - chainIndex: ETH=1 / BSC=56 / Base=8453，响应 `data[].transactionList[]`
  - 5s轮询，~2.5s感知

### Agent 交易链路
Claude LLM → 策略DSL → 规则引擎 → 风控 → OKX DEX（quote→swap→sign→broadcast→record）

### Supabase 主要表
- `hot_coins` / `hot_daily_picks` / `pump_daily_report`
- `smart_money_signals` / `smart_money_txns` / `smart_wallets`
  - `smart_wallets` 列名是 `wallet`（非 `address`），无 `chain` 列
- `strategies` / `strategy_executions` / `user_api_quota`
- 共 18 个 Migration

---

## 踩坑速查（详见 pitfalls.md）
- OKX Wallet API 必须用 `web3.okx.com`（www.okx.com 返回403）
- `transactions-by-address` 参数是 `chains=`（List），不是 `chainIndex=`
- 响应结构 `data[].transactionList[]`，不是 `tokenTransferDetails[]`
- EVM nonce 必须动态获取，hardcode 0 只对首笔有效
- Supabase DDL 只能 Dashboard SQL Editor 手动执行
- Flutter `|| null` 误杀零值，改用 `?? null`
- Python 3.9 不支持 `X | None`，用 `Optional[X]`
- ON CONFLICT 重复行：同批次去重用 `seen_keys: Set[tuple]`

---

## 2026-03-17 本次会话
- ✅ 聪明钱升级为实时：SOL Helius WS ~400ms + EVM OKX 5s轮询（commit c8fcad7）
- ✅ 修复 OKX base URL / endpoint / 参数 / 响应解析
- ✅ 修复 smart_money_txns 批次去重、OKX toplist 429
- ✅ 记忆文件重构：双份同步机制、topic文件拆分、sessions-log.md 新建
- ✅ pump采集三阶段架构（commit bac2a06）：WS全量捕获→交易追踪(20k)→按需enrich(Sem20)
- ✅ Portal 部署到服务器：systemd portal.service + nginx 反代分流（FastAPI:8000 / Next.js:3000）
- ✅ 实时信号池（commit 2c2e227）：替代每日推荐，score>=55 且 BC 3-35% 动态进出
- ✅ Flutter PicksScreen 重写：30s 轮询 /api/pump/signals 实时显示
- ✅ nginx 新增 /api/pump/ 路由
- ✅ **i18n 国际化**（commit fc4740a + 1158d63）：zh/en/ja/ko，275+ 字符串，语言切换器，QA 修复 80+ 遗漏
- ✅ **App Store 上传**（Build 2 成功）：手动签名 + ATS 修复 + App 名称 AiTrading Pro
- ✅ **App Icon 替换**：AI 大脑+K线图 icon（替换 Flutter 默认 logo）
- ✅ **Build 3 + App Store 已提交审核**
- ✅ **热币实时管理器**（commit 9f5d6fe + 332eb6d + 1cca6e0）：
  - 毫秒级打分 + GeckoTerminal 发现 + 退出机制
  - 表现追踪：发现瞬间价格 + D0~D30 涨幅 + 退出原因写入 DB
  - Portal 修复：daily_highs key 映射 + D0 列 + hot_live source
- ✅ **Hot Coin Optimizer Agent**（commit b775038）：AI 自动分析+优化热币推荐算法
- ✅ **监控口径修正**（commit bdec08d）：7天窗口/1h涨幅/退出后3天追踪/D3命中率
- ✅ **内盘 Bug 修复**（commit 533d43e）：isoformat crash + 零交易内存泄漏 + enrich 竞态
- ✅ **聪明钱地址供给系统**（commits 988b51a + e28c4cc + 16a4102）：
  - 三层供给：miner(+12) + TopHolders(实时) + Dune(+493)，总数 1506
  - v3 五维度评估：胜率/PNL/规模/活跃度/时效性，总分100，2h评估
  - 实时bot检测 + 14天降级/28天移除
  - Dune 4链 Query: SOL(6850812) ETH(6858638) BSC(6858633) Base(6858622)
  - DEX 程序级监控（毫秒级）：SOL logsSubscribe 5 DEX + EVM eth_subscribe Swap 3链
  - HashSet 匹配：SOL 5222 + EVM 10540 地址，O(1) 23ns/lookup
  - 分页加载修复：Supabase limit 1000 → 全量 15710
  - 内盘 pump 评分自动联动全量地址库
- ✅ **Agent 事件驱动升级**（commit ce12609）：
  - event_listener.py：EventBus 订阅毫秒级策略评估（替代 30s 轮询）
  - performance_analytics.py：胜率/PNL/最大回撤/夏普率
  - backtester.py：策略回测引擎（7 天历史数据）
  - risk_manager.py：+2 风控检查（BTC 大盘 + 同链集中度），总 15 项
  - 4 新 API：performance/portfolio/daily-pnl/backtest

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meiyaobuyao123-hash/Agent-Trading](https://github.com/meiyaobuyao123-hash/Agent-Trading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
