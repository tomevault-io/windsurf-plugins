---
trigger: always_on
description: CCStockWorkEnv is a Claude Code environment for multi-market stock research & financial analysis, accessed via Claude Telegram Bot (ctb). Supports US, Taiwan, and China markets.
---

# CCStockWorkEnv — 股票研究分析環境

## 專案概述

CCStockWorkEnv is a Claude Code environment for multi-market stock research & financial analysis, accessed via Claude Telegram Bot (ctb). Supports US, Taiwan, and China markets.

**Launch:** `ctb /Users/wanghsuanchung/Projects/CCStockWorkEnv`

## 核心原則

1. **Fail-fast** — No overprotective try-except. Let errors propagate with clear messages.
2. **Direct dict access** — Use `dict["key"]` by default. Only `.get()` for truly optional fields.
3. **No workarounds** — Investigate root causes, don't mask bugs with fallbacks.
4. **No over-engineering** — Only build what's needed now.

## 回應方式 — 優先產生網頁報告

When the user's request involves **multi-stock analysis, financial data, company research, industry overview, or any response that would exceed ~10 lines of text**, Claude MUST generate an HTML report on the web server instead of sending a long Telegram message.

### Rules

1. **Always reply in the original chat** — Respond directly in the conversation. The ctb bot delivers your reply to the original Telegram chat/group automatically. Do NOT call `send_message.py` or `send_mail.py` unless the user explicitly asks to send a message/email, or in scheduled tasks that run without a conversation. Never use email or Telegram send tools as a substitute for replying.
2. **Web report over Telegram text** — If the answer contains tables, financial data, multiple stocks, or detailed analysis, always produce an HTML report in `output/` and reply with the web server URL. Never dump walls of text into Telegram.
3. **Embed interactive charts** — For any stock mentioned in the report, embed a K-line chart via iframe. **MUST use relative path** (starts with `/`), NEVER use `http://localhost:8800` or any absolute URL — mobile users cannot reach localhost.
   ```html
   ✅ <iframe src="/charts/TICKER/?market=XX&period=1y&embed=1" ...>
   ❌ <iframe src="http://localhost:8800/charts/TICKER/..." ...>
   ❌ <iframe src="http://<EXTERNAL_IP>/charts/TICKER/..." ...>
   ```
   Full tag: `<iframe src="/charts/TICKER/?market=XX&period=1y&embed=1" style="width:100%;height:420px;border:1px solid #e0e0e0;border-radius:6px;" loading="lazy"></iframe>`
4. **Provide evidence** — Back up claims with data. Include: stock price charts (embedded iframes), key financial ratios in tables, data source attribution, and relevant metrics (P/E, P/B, ROE, etc.).
5. **Telegram reply should be short** — Only reply with a brief summary (2-3 sentences) + the report URL.
   **Report URL construction:**
   - Read `config.json` → `web_server.fixed_ip` and `web_server.external_port`
   - If `fixed_ip` is set (non-empty, not null): use `http://<fixed_ip>:<external_port>/reports/SLUG/`
   - If `fixed_ip` is missing/empty: fallback to `http://localhost:<internal_port>/reports/SLUG/` and **warn the user** that the URL is only accessible on the local network, not from mobile devices outside the LAN
   - Example: `📊 騰訊 vs 阿里巴巴個股分析報告已產生，包含財務數據、K線圖及關鍵指標。\n🔗 http://<FIXED_IP>:<EXTERNAL_PORT>/reports/SLUG/`
6. **Short answers stay in Telegram** — Simple queries like "台積電現在股價多少?" can be answered directly in Telegram without generating a report.

### Examples of when to generate a report

- "幫我查詢騰訊和阿里巴巴的個股資訊及財務數據" → HTML report with company info tables, financial data, embedded K-line charts
- "找美國上市的光通訊公司，檢查去年財務狀況" → HTML report with screened companies, financial health analysis, charts
- "分析半導體產業趨勢" → HTML report with industry overview, key players, charts
- "比較 AAPL 和 MSFT" → HTML report with side-by-side comparison, charts

### Examples of when Telegram text is OK

- "台積電現在多少錢?" → Short price quote in Telegram
- "MU 的 P/E 是多少?" → One-liner answer in Telegram

## 語言慣例

- **Code & docs**: English
- **Claude 回應**: 繁體中文 (Traditional Chinese) — Claude MUST respond in Traditional Chinese at all times
- **User-facing output**: 繁體中文 (Traditional Chinese)
- **Commands 描述**: 繁體中文 (shown in Telegram)

## 專案結構

```
CCStockWorkEnv/
├── CLAUDE.md                    # This file
├── .claude/commands/            # CTB slash commands (14)
├── .claude/skills/              # Domain knowledge (8)
├── .claude/agents/              # Specialized agents (2)
├── tool_scripts/
│   ├── send_telegram/           # Telegram messaging
│   ├── send_mail/               # Email via Mailgun
│   ├── market_data/             # Market data API abstraction
│   ├── financial_calc/          # Z-Score, F-Score, screener
│   ├── db_ops/                  # SQLite operations
│   ├── report_gen/              # Report & chart generation
│   └── web_server/              # Django report viewer (RWD)
├── schedules/                   # Scheduled task scripts
├── data/                        # SQLite DB, exports, charts, logs (gitignored)
├── output/                      # Timestamped reports (gitignored)
└── prompts/                     # YYYYMMDD_N_description.md
```

## Python 執行模式

All tool_scripts run via uv:
```bash
cd tool_scripts/<subfolder> && uv run python <script>.py [args]
```

## prompts/ 命名

Format: `YYYYMMDD_N_description.md` where N is sequential for the day.

## 資料庫


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoatWang/CCStockWorkEnv](https://github.com/GoatWang/CCStockWorkEnv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
