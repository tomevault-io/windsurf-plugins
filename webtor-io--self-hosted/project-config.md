---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Что это

`self-hosted` — all-in-one Docker-образ Webtor (`ghcr.io/webtor-io/self-hosted`): 13 сервисов платформы (включая `vault` и `content-prober`) + nginx + embedded PostgreSQL + Redis + embedded S3-хранилище (versitygw, не webtor-компонент) + событийная шина NATS (nats-server/nats-box, тоже не webtor-компонент) в одном контейнере под супервизором s6-overlay v3. **Собственного Go/JS-кода здесь нет** — репозиторий состоит из Dockerfile, s6-описаний сервисов и шаблонов конфигов. Ничего не компилируется: Dockerfile копирует готовые бинарники и ассеты из прекомпилированных образов компонентов, опубликованных CI каждого сервисного репозитория (`ghcr.io/webtor-io/<svc>`), закреплённых по тегу и дайджесту.

Общий контекст платформы (архитектура сервисов, matryoshka chaining и т.д.) — в `../CLAUDE.md`.

## Команды

```bash
# Сборка (быстрая: копирует готовые артефакты из образов компонентов; замеры на amd64:
# ~49s холодная, ~31s тёплая — против 40+ минут у старого компилирующего Dockerfile)
docker build -t webtor-self-hosted:assembly .
# Под конкретную архитектуру: docker build --platform linux/arm64 -t webtor-self-hosted:arm64 .

# Запуск и проверка
docker run -d -p 8080:8080 -v data:/data -v pgdata:/pgdata --name webtor webtor-self-hosted:assembly
curl http://localhost:8080
docker logs webtor        # логи всех сервисов с префиксами [service-name] через s6-log

# End-to-end смоук-сьют: boot, DDL, zip-архив, HLS (nginx-vod), транскодирование
# (session API content-transcoder), субтитры, персистентность после рестарта,
# embedded S3 (подписанный round-trip объекта через 127.0.0.1:8099 изнутри контейнера),
# vault (пледж торрента, событие через NATS, файл в /storage/vault, cron-джобы)
tests/run.sh webtor-self-hosted:assembly
```

`tests/run.sh [image]` без аргумента по умолчанию тянет `ghcr.io/webtor-io/self-hosted:latest`. На момент написания этот тег ещё не содержит фикс подписи export-ссылок rest-api (`fix: sign rest-api export urls so torrent-http-proxy accepts them`), поэтому голый прогон падает на сценариях, завязанных на export (архив, HLS, субтитры). До выхода релиза с этим фиксом гонять сьют нужно на локально собранном образе — соберите его командой выше и передайте `tests/run.sh` явным аргументом.

**Релиз:** пуш тега `v*` запускает GitHub Actions (`.github/workflows/docker-image.yml`), который делегирует сборку и публикацию multi-arch-манифеста (amd64+arm64) переиспользуемому workflow `webtor-io/.github/.github/workflows/docker-multiarch.yml`. Все 13 компонентных репозиториев используют тот же workflow и публикуют свои образы под обе архитектуры. PR-гейт (`.github/workflows/test.yml`) собирает образ и гоняет `tests/run.sh` нативно на amd64- и arm64-раннерах; обе ноги обязательны.

Порт хоста для тестов задаётся `WEBTOR_HOST_PORT` (по умолчанию 8080) — пригодится, когда 8080 занят локальным дев-сервером: `WEBTOR_HOST_PORT=18080 tests/run.sh <image>`.

## Как обновить версию сервиса

Компоненты пинятся в Dockerfile по тегу и дайджесту: `FROM ghcr.io/webtor-io/<svc>:<tag>@sha256:<digest> AS <svc>`. Файлов `*.commit` в репозитории больше нет — provenance каждого компонента полностью описывается этой строкой в Dockerfile.

Штатный путь — Renovate (`renovate.json`): следит за `ghcr.io/webtor-io/**`, при появлении нового дайджеста под тем же тегом открывает отдельный PR на компонент (намеренно не групповой — так по упавшей смоук-джобе из `.github/workflows/test.yml` сразу видно, какой компонент виноват).

Не-webtor стейджи Dockerfile — `versitygw` (сторонний S3-гейтвей за `/storage`), `nats`/`natsbox` (событийная шина и её CLI, образы `nats:alpine`/`natsio/nats-box`), базовый `alpine`, плюс `nginx:alpine` в тестовом compose — Renovate тоже видит (через `config:recommended`), но ведёт по другому правилу (`renovate.json`, группа «base images»): не-major бампы собираются в один PR, ждут 7 дней после релиза апстрима и автомержатся по зелёным smoke; major-бампы (например alpine 3.x → 3.y — это смена musl/openssl для всех бинарников образа) требуют одобрения на dependency dashboard и мержатся только вручную. Логика отличия от webtor-компонентов: те к моменту бампа уже отработали в проде webtor.io, а base-образы нигде у нас не обкатывались — отсюда выдержка.

Ручной бамп — то же самое руками: узнать новый дайджест образа (например, `docker buildx imagetools inspect ghcr.io/webtor-io/<svc>:<tag>`) и заменить `@sha256:...` в соответствующей строке `FROM`. Тег (`master`/`main`, в зависимости от компонента — см. Dockerfile) обычно не трогают.

## Архитектура образа

### Сборка (Dockerfile)

Multi-stage, но ничего не компилируется. Каждый `FROM ghcr.io/webtor-io/<svc>:<tag>@sha256:<digest> AS <svc>` — это уже готовый образ, собранный CI соответствующего сервисного репозитория (тег и дайджест зафиксированы вместе, см. «Как обновить версию сервиса»). Финальный стейдж (`FROM alpine:${ALPINE_VER}`) вытаскивает артефакты через `COPY --from=<svc> <src> <dst>`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webtor-io/self-hosted](https://github.com/webtor-io/self-hosted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
