---
trigger: always_on
description: Private automation that grabs Undip Foodtruck coupons via SSO and delivers them via WhatsApp. Three runtimes share one MariaDB: a Node WhatsApp bot, a Python coupon-taker, and a Next.js admin (currently boilerplate).
---

# Claude project instructions

Private automation that grabs Undip Foodtruck coupons via SSO and delivers them via WhatsApp. Three runtimes share one MariaDB: a Node WhatsApp bot, a Python coupon-taker, and a Next.js admin (currently boilerplate).

## Architecture you should know before editing

The Node bot was 1418 lines in `src/chat_bot/bot.js` until commit `508d92b`. Now it's a routing layer:

```
src/chat_bot/
  bot.js              ← whatsapp-web.js client + stealth shim + cron.start()
  router.js           ← single route(msg, client, deps) entry point
  state.js            ← pending_action FSM helpers
  helpers.js          ← location/status names, time predicates, sanitization
  views.js            ← every reply string lives here, as functions returning Indonesian text
  cron.js             ← schedule.scheduleJob registrations + sendCoupons / doLoginAccounts / etc
  commands/
    index.js          ← ordered registry of text commands + admin + image + pending handlers
    ufood.js          ← ufood / commands (single canonical orientation; help/alur/aturan all redirect)
    daftar.js         ← ufood daftar — auto-grants 2x trial on first registration + spawns auto-login
    akun_list.js      ← ufood akun / ufood akun N
    akun_lokasi.js    ← snapshot edits, no `ya` prompt
    akun_submit.js    ← snapshot edits, capacity-checked
    akun_beli.js      ← QRIS image + sets pay_sso_id
    akun_ganti.js     ← snapshot edits, status_login reset to 0
    akun_hapus.js     ← only command that still uses `ya` (besides ping)
    status.js         ← ufood status
    subscribe.js      ← actually toggles wa_messages.subscribed
    ping.js           ← admin handoff, 3h auto-expire
    admin.js          ← !login, !kupon, !kirim, payment confirm (ya N / tidak), ping resolution (sudah)
    image.js          ← refuses uploads without prior `ufood akun N beli` (no admin spam)
```

Each command exports `{ name, match(body, msg), handle({msg, params, client, deps}) }`. Commands that put pending state also export `resolveConfirm({msg, pending, client, deps})`.

`router.js` runs in this order:
1. Block status@broadcast / group messages
2. New user (`waMsgIsBlocked` returns -1) → welcome + create row
3. Currently blocked → drop (admin handles directly)
4. Block just expired (3h passed) → notify + drop this message
5. Image/document → `commands.image.handle`
6. Sender is admin → `commands.admin.handle` first, fall through if it returns null
7. Pending action exists → matching `commands.pending[prefix].resolveConfirm`
8. Text command match (first hit in `commands.text[]`)
9. Fallback → `views.unknownCommand()`
10. Errors anywhere → `errorLogAdd` + `views.commandError()` reply with ❌

## Conventions

- **Replies are always Indonesian, WhatsApp-style.** Use `*bold*`, `_italic_`, `> quote`, `⏳ ✅ ❌` emoji where it fits.
- **Every reply string lives in `views.js`** as a named function. Don't inline strings in handlers.
- **No `ya` confirmation for non-destructive actions.** Snapshot the change directly with `oldVal → newVal`. Only `hapus` and `ping` keep `ya`/`batal`.
- **Pending state format:** `prefix` or `prefix:payload` (e.g. `delete:42`, `ping`). Stored as VARCHAR(64) in `wa_messages.pending_action` with a 5-min TTL via `pending_action_at`.
- **WhatsApp Web version: never override.** Letting `whatsapp-web.js` use its bundled default is the difference between login working and "Try Again" — see [memory: WhatsApp Try Again fix](../../../Users/nanda/.claude/projects/c--Projects-undip-foodtruck-mallocation-com/memory/feedback_whatsapp_web_stealth.md).
- **Stealth shim must stay at the top of `bot.js`** — `puppeteer-extra` + `puppeteer-extra-plugin-stealth` injected into `require.cache["puppeteer"]` before whatsapp-web.js is required.
- **Snap chromium on aarch64.** `CHROME_EXECUTABLE_PATH` and `CHROMIUM_EXECUTABLE_PATH` both point at `/usr/bin/chromium-browser`. The puppeteer-bundled `linux_arm-*` binary is actually x86-64 and won't run on the Oracle Cloud ARM host.
- **Headless mode is `false` on the server,** wrapped by `xvfb-run` via `scripts/start.sh`. WhatsApp Web's frame detached errors go away with a virtual display.

## Database (`sql_undip_foodtruck` on MariaDB 10.11)

Schema lives in **two** places — keep them in sync:
- Node: `src/models/tables.js` (Sequelize)
- Python: `python/database.py` (SQLAlchemy) — only knows `registereds`, `sso_accounts`, `taken_coupons`. The Python side does NOT touch `wa_messages` or `error_logs`.

Tables:
- `registereds` — wa_number → comma-separated sso_ids (max 3) + `pay_sso_id`
- `sso_accounts` — encrypted email/pwd, location, quota, submit toggle, `status_login`
- `wa_messages` — per-user FSM: `subscribed`, `pending_action`, `pending_action_at`, `blocked`, `blocked_at`, `free_trial`, `last_messages`, plus legacy `confirmation`/`rules_accepted` no longer read by new code
- `taken_coupons` — daily attempt log
- `error_logs` — append-only handler errors (chunk-4 telemetry)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raidsmithz/undip-foodtruck](https://github.com/raidsmithz/undip-foodtruck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
