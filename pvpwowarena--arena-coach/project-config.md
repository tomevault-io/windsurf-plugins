---
trigger: always_on
description: > Последнее обновление: 2026-07-30 (Phase 4.19 — аддон 0.4.0, мост 0.9.2)
---

# Arena Coach — CLAUDE.md (контекст проекта для новых чатов)

> Последнее обновление: 2026-07-30 (Phase 4.19 — аддон 0.4.0, мост 0.9.2)
> Читай этот файл в начале каждого нового чата перед любой работой.
> **Живые цифры KB** (драфты/гипотезы/покрытие) — в `docs/NEXT-SESSION-HANDOFF.md` и `docs/COVERAGE.md`, не здесь (этот файл — про архитектуру/инфру).

---

## Что это за проект

**WoW Arena Assistant** — система реал-тайм подсказок для 2v2/3v3 арены в **WoW: Burning Crusade Classic Anniversary** (клиент 2.4.3).

Игроки из вайтлиста получают советы в Discord DM во время боя. Источник знаний — KB (база матчапов в Markdown), наполненная вручную из стримов/гайдов.

**Репо:** https://github.com/pvpwowarena/arena-coach — **публичный** (с 2026-05-15).

---

## Архитектура (актуальная)

```
[WoW client + ArenaCoach addon (Lua 2.4.3)]
        │ пишет события в chat-frame с префиксом [AC|TYPE|f1|f2|...]
        │ (v0.2.0: ARENA_START шлёт и союзников; форс-флаш чат-лога LoggingChat toggle)
        ▼
[arena-bridge (Windows .exe + macOS arm64, PyInstaller onefile)]
        │ tail WoW Logs/WoWChatLog.txt ИЛИ Chat-YYYY-MM-DD.txt (автодетект, v0.3.0)
        │ → нормализация → HTTPS POST /v1/events, Bearer-токен аутентификация
        ▼
[Backend VPS: pvpwowarena.surprise4you.dev]
        ├── FastAPI (uvicorn, 127.0.0.1:8000) — systemd arena-coach-api
        ├── Discord bot (discord.py)           — systemd arena-coach-bot
        ├── KB store (Markdown matchups + in-memory KBIndex)
        ├── Whitelist + Audit log (SQLite + Fernet шифрование, append-only JSONL)
        └── LLM orchestrator (Anthropic API — опционально, сейчас заглушка)
        ▼
[Nginx 1.18 + TLS (Let's Encrypt)]
        ├── /              → /var/www/arena-coach/index.html
        ├── /download      → download.html (аддон + arena-bridge.exe)
        ├── /how-it-works  → how-it-works.html
        ├── /v1/           → FastAPI
        └── /health        → FastAPI
        ▼
[Discord DM — текстовые подсказки игрокам]
```

**Realtime-канал (Phase 4.2, 2026-07-23):** bridge читает **COMBAT-лог**
(`Logs/WoWCombatLog-MMDDYY_HHMMSS.txt`, автодетект свежего) и сам собирает события:
ARENA_START по ауре Arena Preparation (32727/32728), классы игроков из кастов
(re-emit при уточнении), TRINKET/ABILITY по spell id. Причина: **chat-лог в
Anniversary-клиенте не флашится до полного выхода из игры** (буфер ~48КБ,
LoggingChat(false→true) — no-op; проверено живым тестом), combat-лог флашится в бою.
Аддон 0.2.2 включает LoggingCombat при логине. Chat-frame канал `[AC#...]`
(ADR-0003, разделитель `#` с 0.2.1 — «|» запрещён клиентом) остаётся легаси-режимом
(`--no-combat-log`); SavedVariables как realtime отвергнуты (только /reload и logout).

**Имя chat-лога (v0.3.0):** стандартно `/chatlog` пишет в `Logs/WoWChatLog.txt` (не `Chat-YYYY-MM-DD.txt`, как предполагал ADR-0003). Bridge следит за обоими кандидатами и выбирает растущий. Клиент буферизует запись → аддон форс-флашит `LoggingChat(false→true)` после критических событий (отключается `/ac flush off`).

---

## VPS

| Параметр | Значение |
|---|---|
| IP | 77.239.120.150 |
| Домен | pvpwowarena.surprise4you.dev |
| ОС | Ubuntu 22.04 LTS |
| Python на VPS | **3.10** (не 3.11!) |
| Systemd сервисы | `arena-coach-api` (uvicorn :8000) + `arena-coach-bot` |
| Nginx | 1.18.0 — `listen 443 ssl;` без `http2 on;` |
| TLS | Let's Encrypt, certbot --nginx, автообновление через certbot.timer |
| Данные | `/var/lib/arena-coach/coach.db` (SQLite) |
| Конфиг | `/etc/arena-coach/api.env` (секреты) |
| Репо на VPS | `/opt/arena-coach/` |
| Venv | `/opt/arena-coach/.venv/` |
| Статика nginx | `/var/www/arena-coach/` (index.html, download.html, how-it-works.html, arena-bridge.exe) |
| Webmin | порт 10000, правило UFW открыто |

### Проверка работоспособности
```bash
curl -s https://pvpwowarena.surprise4you.dev/health
# → {"status":"ok","uptime_s":...}

systemctl status arena-coach-api arena-coach-bot --no-pager
```

### Деплой
```bash
# Через Webmin terminal на VPS:
cd /opt/arena-coach
sudo -u arenacoach git pull --ff-only
cp ops/nginx/html/*.html /var/www/arena-coach/   # если статика менялась
cd backend && sudo -u arenacoach /opt/arena-coach/.venv/bin/alembic -c alembic.ini upgrade head
sudo systemctl restart arena-coach-api arena-coach-bot
```

Альтернатива с локальной машины:
```bash
ARENA_VPS_HOST=root@77.239.120.150 ./ops/scripts/deploy.sh
```

### api.env на VPS
```
DISCORD_BOT_TOKEN=...
DISCORD_GUILD_ID=...
ARENA_COACH_OWNER_DISCORD_IDS=...
ANTHROPIC_API_KEY=sk-ant-...          # боевой ключ активирует LLM (Phase 4.7):
                                      # разбор незнакомых сетапов + постматч.
                                      # Пусто/нет ключа = чистый детерминизм (в бою LLM не нужен).
ANTHROPIC_MODEL_SYNTH=claude-sonnet-4-6         # постматч-разбор (качество, 1 вызов/матч)
ANTHROPIC_MODEL_CLASSIFY=claude-haiku-4-5-...   # (зарезервирована под классификацию)
ANTHROPIC_MODEL_ADVICE=claude-haiku-4-5-...      # разбор незнакомого сетапа (фон+кэш);
                                                 # можно Sonnet — генерим раз, отдаём дёшево навсегда
ARENA_COACH_FERNET_KEY=...
BRIDGE_BEARER_TOKEN=...
DATABASE_URL=sqlite+aiosqlite:////var/lib/arena-coach/coach.db
KB_PATH=/opt/arena-coach/kb

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pvpwowarena/arena-coach](https://github.com/pvpwowarena/arena-coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
