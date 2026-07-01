---
trigger: always_on
description: Pure transforms from data structures to wire format. Output is a string/bytes meant for Telegram or Telegraph.
---

# rendering/ — data → user-visible output

Pure transforms from data structures to wire format. Output is a string/bytes meant for Telegram or Telegraph.

See root `CLAUDE.md` Invariant #4 for the MarkdownV2/HTML carve-out — this subpackage owns the HTML side.

`email_client.py` mirrors `telegraph_client.py`'s shape — a single file owning both the HTML template build AND the network send (in `asyncio.to_thread`). They live in `rendering/` because their output is "string/bytes meant for external delivery" — same membership rule as the existing Telegraph publish path. Email delivery is **opt-in per user** (digest schema's `email` field) and **mirror-only**: failures NEVER break the Telegram path (the email_client swallows internally; `run_user_digest` also wraps the call in another try/except for defense in depth). `is_email_configured()` is the env-var gate (`RESEND_API_KEY` + `RESEND_FROM` both required). The HTML body uses inline-style attributes only — Gmail and Outlook both strip `<style>` blocks and ignore `<link rel=stylesheet>`. Per-ticker `.md` attachments are built lazily via `load_historical_state` (the same disk path `/history` reads), keeping email building decoupled from the digest's `status` dict (which only carries the signal + Telegraph URL). A missing log file (ephemeral Docker FS without persistent `TRADINGAGENTS_RESULTS_DIR`) skips just that ticker's attachment with a log line, never the whole email.

`send_digest_email` returns `EmailSendResult(ok, recipient, message_id, error)` (frozen dataclass). `ok=True` only on Resend-accepted send with a message id; otherwise `error` is `"open_mode"` / `"not_configured"` / `"malformed_response"` / `type(e).__name__`. Never raises — Telegram-mirror invariant unchanged. **Open-mode backstop (M3):** the first check refuses the send (`error="open_mode"`, before the env check and before any `import resend`) when `Config.ALLOWED_USER_IDS` is empty — the mirror relays through the operator's verified domain to a user-supplied recipient, so an open bot would be a spam relay. This single gate covers the daily mirror AND the immediate `/email test` / `diagnose` sends; the command layer refuses earlier with a clearer message (see `handlers/CLAUDE.md`). The footer-rendering caller lives in `handlers/CLAUDE.md`. `check_resend_domain(domain) -> (status, error)` lives here too: any Resend SDK call (send, domain check) goes through `email_client.py` so all SDK access is in one module (parity with `telegraph_client.py` owning every Telegraph SDK call). `/email diagnose` is the only caller today; future health-check or admin commands reuse it.

**Membership-rule inflection point at N=3 delivery channels.** This subpackage was originally "string/bytes meant for Telegram or Telegraph" (PR #56 origin). PR #73 broadened it to "string/bytes meant for external delivery" so `email_client.py` fits cleanly. Safe at N=2. If a third delivery channel ever lands (Slack webhook, Discord, SMS, etc.), the new author MUST consciously re-evaluate whether `rendering/` is still the right home or whether a `delivery/` subpackage is warranted — otherwise the rule silently degrades to a catch-all. The decision needs to be intentional, not invisible. Until then, two delivery clients side-by-side (telegraph_client + email_client) is the documented shape.

## Key contracts

- **Caption rendering (HTML).** `format_short_message` builds the post-analysis caption as **HTML**: signal emoji (`BUY` 🟢, `OVERWEIGHT` 🟩, `HOLD` 🟡, `UNDERWEIGHT` 🟥, `SELL` 🔴), `<b>ticker</b>`, `<b>signal verb</b>`, summary wrapped in `<blockquote expandable>`, then `<i>Generated …</i>` + `<i>via provider · …</i>`. Summary sources from **`final_trade_decision`** (post-risk-debate synthesis — matches the badge by construction) via `formatters.caption_summary`, which prefers the **`**Executive Summary**` section** body when present (the structured TL;DR with concrete price levels + stops + re-entry zones) and falls back to a strip-header + 700-char clip for older or non-conforming outputs. Both `**Executive Summary**:` and `**Executive Summary:**` header variants are accepted; section terminates at the next `\n\n**` paragraph header. **No inline Telegraph link** — `_full_report_keyboard(ticker, date, telegraph_url)` emits a two-button row (📰 Instant View URL → Telegraph IV, may truncate sections under the 40K budget · 📥 Download Full Report callback → unbounded `.md`, all 7 sections). Label asymmetry advertises the completeness guarantee. On publish failure (`telegraph_url=None`), the IV button is omitted AND the caption surfaces `⚠️ Instant View unavailable`. Summary runs through `markdown_to_telegram_html`; tables and `<img>` are dropped by the sanitizer to fit the 1024-char caption budget.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvanWng97/TradingAgents-Telegram](https://github.com/IvanWng97/TradingAgents-Telegram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
