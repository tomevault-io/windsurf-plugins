---
trigger: always_on
description: LLM multi-agent 美股量化交易系统，通过 Alpaca 执行交易（默认 paper trading）。
---

# quant-agent

LLM multi-agent 美股量化交易系统，通过 Alpaca 执行交易（默认 paper trading）。

## 入口 + 测试

```bash
pytest tests/ -v                                    # 全量测试
python main.py --mode morning|midday|evening|live   # 手动跑
python main.py --mode daily                          # P&L 历史 CSV → Telegram（无 LLM 无交易）
```

**Prompt/CoT 改动验证（决策重放 harness，2026-06-07 加）**：prompt 改动这里没法回测,所以容易"听起来对就上"。`scripts/replay_decision.py` 把 `agent_logs` 里某次真实调用的 `input_message` 喂回**当前** prompt+model,结构化 diff 新旧决策(PM 比 per-symbol target 权重/conviction 的增删改),把"我觉得这版更好"变成"它在 N 个真实输入上具体怎么改了决策"。`--no-llm` 只列待重放、不烧 token；要 LLM 时需 `.env` key。核心逻辑在 `src/replay.py`(load/replay/diff,有单测),靠 `base.py:BaseAgent._execute(user_message)` 这个 seam(= `run()` 去掉 build_user_message 的部分)。**这是"先验证再改"的工具——P1-P5/#1-#2 这类软层改动应该用它在历史决策点上看效果,而不是只靠论证**。outcome-aware 评分(对比次日/5日真实走势判好坏)是它之上的下一层,尚未建。

生产路径走 OS-level timer，不走 `--mode live`：
- **Linux（当前部署，2026-05-11 起）**：systemd 用户 timer `~/.config/systemd/user/quant-agent@.timer` + service template，`Linger=yes` 让 user 退出登录后 timer 仍触发。状态：`systemctl --user list-timers 'quant-agent@*'`、日志：`journalctl --user -u 'quant-agent@*.service'`
- **macOS（legacy）**：launchd `~/Library/LaunchAgents/com.quant-agent.*.plist`，配套 `scripts/install_plists.sh` 安装/重载。**2026-05-11 起 Mac 路径已停用**（迁到 Linux server）；plist 文件保留作 fallback，详细注意事项见下面 "macOS Sequoia" 段

## 架构速览

- 8 个日常 LLM agent：tech / news / macro / earnings / portfolio_manager / risk_manager / position_reviewer / evening_analyst。额外一个 **meta_reflector** 每季度末跑一次，负责对 6 个可编辑 agent（tech / news / macro / earnings / PM / evening）**自我画像 + prompt 审计 + 自动修订**。risk_manager 和 position_reviewer 被 **schema + prompt_editor 双层保护**，不允许被 auto-evolved 改动（硬纪律不能被稀释）
- 双层风控：硬规则引擎（cash_only / 仓位 / 暴露 / 日损 / 板块 / 相关性 / earnings-queued） + LLM RiskManager 审核 + `_force_delever()` 硬兜底
- **6 个 session**（ET Mon-Fri）：earnings_preprocess 08:00-09:15（唯一跑 earnings LLM）、morning 09:30-12:00（full team）、intra_check 09:30-16:00 每 30min tick（熔断器，零 LLM）、midday 13:00-14:30（position_reviewer patient）、close 15:30-16:00（position_reviewer act-on-trigger；窗口 ≥ launchd 30min tick 保证任何 phase 都能打中）、evening 20:00-22:00（report + outlook）。**季度末额外一次 meta**：`--mode meta` / `run_quarterly_meta_reflection`，跑 `quarterly_digest` 聚合 90 天事实 + `meta_reflector` LLM 7 步 CoT + `prompt_editor` 4 道保险 apply。**另有 daily CSV 导出**（2026-06-10 PR #99）：`--mode daily` 把全量 NAV/SPY/drawdown 历史发成 Telegram 文档（取代 evening 推送里的 P&L 文本表）；纯数据导出（零 LLM 零交易），**不走** run_if_et_window.sh（wrapper 会拒掉 daily 模式），由独立 timer `quant-agent-daily.timer`（Mon-Fri 09:00 ET）经 `scripts/run_daily_export.sh` 触发，units 在 `scripts/systemd/`
- 数据源：yfinance、FRED、RSS、SEC EDGAR
- 配置：`config/settings.yaml` + `.env`；按 agent 独立选 OpenAI / Anthropic / DeepSeek 模型。**2026-06-04 起所有 9 个 agent 用 OpenAI `gpt-5.5`**（5-11 曾切 claude-opus-4-7 应对 OpenAI quota，6-04 又切回 OpenAI 并升到 5.5；切 provider 一条 `sed` 命令）。Provider 路由按 model name 前缀判断：`deepseek-` 走 DeepSeek，`gpt-` / `o1-` / `o3-` / `o4-` 走 OpenAI，其它走 Anthropic（`src/agents/base.py` 的 `_DEEPSEEK_PREFIXES` / `_OPENAI_PREFIXES`）
- **DeepSeek（OpenAI-compatible，2026-06-05 加）**：走 openai SDK + `base_url=https://api.deepseek.com` + DeepSeek key（`_call_deepseek`）。三个坑都已处理（研究自 api-docs.deepseek.com）：(1) DeepSeek 只认 **`max_tokens`** 不认 OpenAI 的 `max_completion_tokens`（发错会被静默丢弃 → 回落 ~4096 默认截断）；(2) DeepSeek **拒绝**（不裁剪）超 ceiling 的 max_tokens，所以按 `_DEEPSEEK_MAX_OUTPUT` per-model 客户端 clamp（v4-flash/pro/chat/reasoner=384K，未知 deepseek-* 保守 8192）；(3) 402「Insufficient Balance」= 不可重试 → 触发 failover，`insufficient_system_resource` finish_reason 记为 truncated。**`deepseek-chat`/`deepseek-reasoner` 2026-07-24 弃用**（现已 alias `deepseek-v4-flash`），新配置直接用 `deepseek-v4-flash`。cost 用**官方** $0.14/$0.28（LiteLLM 的 $0.28/$0.42 是 V4 前旧值，已 **pin** 在 `cost_table._PRICING_PINNED` 防 cache 刷新覆盖）
- **跨 provider 自动 failover**：当**非-Anthropic 主**（OpenAI 或 DeepSeek）调用重试耗尽 / 非可重试错误(quota、DeepSeek 402、死 key、宕机)后，`base.py:run()` 会**自动用 Anthropic 的 `_FALLBACK_MODEL`(=`claude-opus-4-7`)单发一次**(无重试,避免吃穿 session 窗口),成功就用它的结果继续(`AgentResult.model` 记实际用的模型,cost 按实际模型算)、失败就抛出原始错误。只在「主=OpenAI/DeepSeek 且 `.env` 有 ANTHROPIC_API_KEY」时触发;主已是 Claude 则 no-op(同 provider 无意义,且构造时不会因此报错)。截断(max_tokens)不触发 failover。pipeline 给 9 个 agent 都传 `fallback_api_key=config.api_keys.anthropic`。`src/config.py` 的 LLMConfig 默认(settings.yaml 漏配时的兜底)也已从过时的 `*-4-6` 更到 `claude-opus-4-7`
- **Telegram 推送**：开/关由 `.env` 控制，缺 `TELEGRAM_BOT_TOKEN` 或 `TELEGRAM_CHAT_ID` 时 notifier 静默 no-op，trading 不受影响。每个 session 在 `main.py` finally 块里调一次 `notifier.send(format_session_result(...))`；噪声策略：morning/midday/close/evening 总推；earnings_preprocess 只在真分析了 filing 时推；intra_check 只在 emergency 触发时推（14 次/天 OK tick 静默）；meta 只在真季末跑时推；daily 的 `sent` 静默（CSV 文档本身就是送达确认，status text 只在 error/skipped 时推且带原因）；**任何 session 抛异常都强制推**绕过噪声策略。文档见 `src/notifier.py` docstring 和 README "Optional env vars" 段

### Agent CoT 结构（schema-enforced 必填字段数；违反 → ValidationError）
| Agent | CoT 步数 | 备注 |
|---|---|---|
| tech_analyst | 5 | trend / momentum / volatility / volume / S&R |
| news_analyst | ❌ 无 | 故意；结构化输出 `state_changes` + `stock_news` 本身就是它的思维结构 |
| macro_analyst | 6 | vol / yield curve / monetary / inflation+labor+credit / cross-signal / sector |
| earnings_analyst | 5 | 嵌在 `investment_implications.reasoning_chain` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yebof/quant-agent](https://github.com/yebof/quant-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
