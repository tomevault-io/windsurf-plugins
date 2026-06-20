---
trigger: always_on
description: You are building a plugin for **Teleton**, a Telegram AI agent on TON. Ask the user what plugin or tools they want to build, then follow this workflow.
---

# Teleton Plugin Builder

You are building a plugin for **Teleton**, a Telegram AI agent on TON. Ask the user what plugin or tools they want to build, then follow this workflow.

## Reference documentation

Before building, read the relevant reference files from the teleton-plugins repo:

- **Full rules & SDK reference**: `CONTRIBUTING.md` — complete guide with tool definition, SDK API tables, error handling, lifecycle, best practices, testing
- **Simple plugin example**: `plugins/example/index.js` — Pattern A (array of tools, no SDK)
- **SDK plugin example**: `plugins/example-sdk/index.js` — Pattern B (tools(sdk) with database, TON balance, Telegram messaging)
- **Advanced SDK plugin**: `plugins/casino/index.js` — real-world SDK plugin with TON payments, payment verification, isolated database, payout logic
- **Registry**: `registry.json` — list of all existing plugins (check for name conflicts)
- **README.md** — project overview, plugin list, SDK section

Read at least `CONTRIBUTING.md` and the relevant example before building.

---

## Workflow

1. **Ask** the user what they want (plugin name, what it does, which API or bot)
2. **Decide** — determine if the plugin needs the SDK (see decision tree below)
3. **Plan** — present a structured plan and ask for validation
4. **Build** — create all files once the user approves
5. **Install** — copy to `~/.teleton/plugins/` and restart

---

## Step 1 — Understand the request

Determine:

- **Plugin name** — short, lowercase folder name (e.g. `pic`, `deezer`, `weather`)
- **Plugin type**:
  - **Inline bot** — wraps a Telegram inline bot (@pic, @vid, @gif, @DeezerMusicBot…)
  - **Public API** — calls an external REST API, no auth
  - **Auth API** — external API with Telegram WebApp auth
  - **Local logic** — pure JavaScript, no external calls
- **Tools** — list of tool names, what each does, parameters
- **Does it need GramJS?** — yes for inline bots and WebApp auth
- **Does it need the SDK?** — use the decision tree below

---

## SDK Decision Tree

The Plugin SDK (`tools(sdk)`) gives high-level access to TON, Telegram, database, logging, and config. Use it **only when needed** — simpler plugins should use Pattern A.

**Use `tools(sdk)` (Pattern B) if ANY of these apply:**

| Need | SDK namespace | Example |
|------|--------------|---------|
| Check TON balance or wallet address | `sdk.ton.getBalance()`, `sdk.ton.getAddress()` | Casino checking balance before payout |
| Send TON or verify payments | `sdk.ton.sendTON()`, `sdk.ton.verifyPayment()` | Casino auto-payout, paid services |
| Get TON price or transactions | `sdk.ton.getPrice()`, `sdk.ton.getTransactions()` | Portfolio tracker |
| Query jetton balances or metadata | `sdk.ton.getJettonBalances()`, `sdk.ton.getJettonInfo()` | Token portfolio, DEX tools |
| Send jettons (TEP-74 transfers) | `sdk.ton.sendJetton()` | Token payments, swaps |
| Query NFT items or metadata | `sdk.ton.getNftItems()`, `sdk.ton.getNftInfo()` | NFT gallery, collection tools |
| Convert TON units or validate addresses | `sdk.ton.toNano()`, `sdk.ton.fromNano()`, `sdk.ton.validateAddress()` | Any TON plugin |
| Send Telegram messages programmatically | `sdk.telegram.sendMessage()` | Announcements, notifications |
| Edit messages or send reactions | `sdk.telegram.editMessage()`, `sdk.telegram.sendReaction()` | Interactive UIs |
| Send dice/slot animations | `sdk.telegram.sendDice()` | Casino dice game |
| Send media (photos, videos, files, stickers) | `sdk.telegram.sendPhoto()`, `sdk.telegram.sendFile()` | Media bots, file sharing |
| Delete, forward, pin, or search messages | `sdk.telegram.deleteMessage()`, `sdk.telegram.pinMessage()` | Moderation, archival |
| Schedule messages | `sdk.telegram.scheduleMessage()` | Reminders, timed announcements |
| Create polls or quizzes | `sdk.telegram.createPoll()`, `sdk.telegram.createQuiz()` | Engagement, trivia bots |
| Moderate users (ban/mute/unban) | `sdk.telegram.banUser()`, `sdk.telegram.muteUser()` | Group moderation |
| Stars & gifts (balance, send, buy) | `sdk.telegram.getStarsBalance()`, `sdk.telegram.sendGift()` | Gift trading, rewards |
| Post stories | `sdk.telegram.sendStory()` | Promotional content |
| Lookup users, chats, or participants | `sdk.telegram.getUserInfo()`, `sdk.telegram.getChatInfo()` | Analytics, admin tools |
| Need an isolated database | `sdk.db` (requires `export function migrate(db)`) | Tracking user scores, history, state |
| Key-value storage with TTL | `sdk.storage` (auto-created, no migrate needed) | Caching, rate limits, temp state |
| Manage API keys or secrets | `sdk.secrets` | Plugins that call authenticated APIs |
| Plugin-specific config with defaults | `sdk.pluginConfig` + `manifest.defaultConfig` | Customizable thresholds, modes |
| Structured logging | `sdk.log.info()`, `sdk.log.error()` | Debug, monitoring |
| Swap tokens on DEX | `sdk.ton.dex.quote()`, `sdk.ton.dex.swap()` | DEX aggregator, trading bots |
| Compare DEX prices (STON.fi vs DeDust) | `sdk.ton.dex.quoteSTONfi()`, `sdk.ton.dex.quoteDeDust()` | Arbitrage, price comparison |
| Check/register .ton domains | `sdk.ton.dns.check()`, `sdk.ton.dns.resolve()` | Domain tools, DNS management |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TONresistor/teleton-plugins](https://github.com/TONresistor/teleton-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
