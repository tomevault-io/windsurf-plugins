---
trigger: always_on
description: LoopDeck is a PHP 8.1+ ThinkPHP 8 cloud-task panel (NetEase Cloud Music, Bilibili, Epic, etc. daily/level tasks). Code lives in `app/`: `index/` serves users, `admin/` provides administration, `cron/` runs scheduled work, `install/` handles first-run setup, and `service/`, `middleware/`, and `command/` hold shared behavior. Platform adapters are under `extend/`. Configuration belongs in `config/`; templates sit in each app's `view/`; browser assets and the front controller are in `public/`. Cont
---

# Repository Guidelines

## Project Structure & Module Organization

LoopDeck is a PHP 8.1+ ThinkPHP 8 cloud-task panel (NetEase Cloud Music, Bilibili, Epic, etc. daily/level tasks). Code lives in `app/`: `index/` serves users, `admin/` provides administration, `cron/` runs scheduled work, `install/` handles first-run setup, and `service/`, `middleware/`, and `command/` hold shared behavior. Platform adapters are under `extend/`. Configuration belongs in `config/`; templates sit in each app's `view/`; browser assets and the front controller are in `public/`. Container scripts live in `docker/`, and regression checks in `tests/`.

Do not edit generated or local-state directories such as `vendor/` and `runtime/`. Treat `public/static/uploads/` as runtime data.

## Architecture & Security Gotchas

- `extend/` is loaded via composer **classmap** (not the `app\` PSR-4 root) and adapters use their own namespaces (e.g. `namespace netease;`, `bilibili\sdk`). After adding or renaming classes there, run `composer dump-autoload`.
- Scheduling is in-process: `cron/` plus `app\service\AutomaticSchedule` execute task classes directly behind a task-name whitelist. Never reintroduce URL self-invocation that puts cookies, `RUN_KEY`, or other secrets into query strings — that pattern was deliberately removed for security ([decision](.agents/notes/implemented/architecture/2026-09-03-in-process-scheduling.md)).
- `runtime/netease-daka/` holds per-account task state files; deleting an account must remove its state file, and the scheduler prunes orphans after `DAKA_STATE_RETENTION_DAYS` (default 30).
- User-facing templates, copy, and README are Simplified Chinese; keep new UI text consistent.
- Frontend console: new platform task entries join the existing `VIP 功能` nav group in `app/index/view/console/head.html` — never create a new nav group for them ([decision](.agents/notes/rejected/architecture/2026-09-24-daily-checkin-platforms.md)).
- Cron task-log entries have a fixed shape: a `[成功]`/`[重试中]`/`[失败]` prefix derived from `Common::statusTag()` in `app/cron/controller/Common.php`, followed by user-facing detail copy composed through the `app\service\TaskMessage` templates — multi-subtask results must never hand-write whole sentences, and "将自动重试" may only appear for results that genuinely retry in minutes ([decision](.agents/notes/implemented/architecture/2026-09-25-task-message-copy-composer.md)). Route every task result through that helper instead of hand-formatting status prefixes, and map scheduler exceptions that reschedule a job to `[重试中]`.
- `daka_new` completion copy is split by entry state: a verification run that finds the settled counter at target must report the day's real totals (`本次已听歌 X 首，累计听歌总数由 Y 首变更为 Z 首`); 「今日已完成，无需重复打卡」 is reserved for runs that start after the day was already completed, and failure copy is separate ([decision](.agents/notes/implemented/architecture/2026-09-25-daka-settlement-confirmation-copy.md)).
- `DOCKER.md` covers container deployment and the updater flow; consult it before touching `docker/`, `compose.yaml`, or `app/service/SystemUpdater.php`.
- The web process never drives Docker: the admin 「立即检查」 button may only create the empty `auto-updater-state.json.check-request` marker, and the updater keeps choosing sources, images and commands itself — never add a web endpoint that updates, pulls or passes parameters to the updater ([decision](.agents/notes/implemented/architecture/2026-09-28-updater-manual-check-request.md)).
- Never report a run that accomplished nothing as success: when zero of N actions land (e.g. `已投币 0/3`), return failure with the upstream reason, and treat per-item refusals such as Bilibili 34005 (video already coined) as skip-and-continue, not end-of-run ([decision](.agents/notes/implemented/architecture/2026-09-28-bilibili-coin-skip-capped-videos.md)).

- Do not reintroduce payment gateways, online purchases, balances or pricing. Entitlements come from registration gifts, administrator settings and administrator-issued redemption codes ([decision](.agents/notes/implemented/architecture/2026-09-28-admin-managed-entitlements.md)).
- New redemption codes combine VIP days with one cross-platform account total; zero means permanent VIP / unlimited accounts. Preserve higher and unlimited benefits on renewal; user-table quota=0 still means no slots. Only legacy quota codes add slots. Serialize redemption and account creation by the same user row lock ([decision](.agents/notes/implemented/architecture/2026-09-29-combined-redemption-account-limit.md)).

## Build, Test, and Development Commands

- `composer install` installs locked PHP dependencies and refreshes autoloading.
- `php think run` starts the ThinkPHP development server (after configuring the database).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [7788dev/loopdeck](https://github.com/7788dev/loopdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
