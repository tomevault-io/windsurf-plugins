---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## О проекте

Анализатор мастерства пилота для iRacing: читает бинарные файлы телеметрии
`.ibt`, режет сессию на круги, находит повороты трассы автоматически, считает
метрики круга/поворота и сравнивает каждый круг с эталоном (лучшим чистым
кругом), выдавая текстовые подсказки о том, где и почему теряется время.
Есть CLI для импорта/анализа и FastAPI-бэкенд, отдающий JSON дашборду
(маршрут `/analysis`). Сайт открывается входом: на `/` —
страница входа по макету Claude Design. Дальше развилка `/main`:
живая телеметрия (`/live`) и разбор прошлых заездов (`/analysis`). Вход не
обязателен — на развилку ведёт и ссылка «без входа», потому что локальный
анализ учётки не требует.

Требует Python >= 3.14, менеджер пакетов — `uv`. Фронтенд — Vite + Vue 3 SFC + TypeScript + Pinia + vue-router в `frontend/`.

## Команды

```bash
uv sync                                    # установить Python зависимости
cd frontend && npm install                 # установить frontend зависимости
cd frontend && npm run build               # собрать фронтенд в web/dist
cd frontend && npm test                    # запустить Vitest тесты фронтенда
uv run pytest                              # все Python тесты
uv run pytest tests/test_laps.py -v        # один файл тестов
uv run pytest tests/test_laps.py::test_name -v   # один тест
uv run iracing-coach import [путь]         # импорт .ibt (файл или папка), по умолчанию Documents/iRacing/telemetry
uv run iracing-coach analyse               # пересчитать повороты/эталоны/подсказки без нового импорта
uv run iracing-coach stats                 # сводка по базе
uv run iracing-coach sync                   # синхронизировать онлайн-кэш iRacing (нужна учётка)
uv run iracing-coach gen-types              # перегенерировать web/api-types.d.ts из pydantic Response-моделей
uv run pyright web/app                      # проверка типов фронта (strict; pyright ставится через uv, без npm)
uv run uvicorn iracing_coach.api:app --reload --port 8712   # HTTP API + дашборд (см. .claude/launch.json)
```

Реальных `.ibt` в репозитории нет (десятки МБ); юнит-тесты собирают
синтетический файл той же двоичной раскладки через `build_ibt`/`synth_ibt` в
[tests/conftest.py](tests/conftest.py). Тесты, помеченные
`requires_real_telemetry`, пропускаются, если в `~/Documents/iRacing/telemetry`
нет настоящих файлов.

## Архитектура

### Конвейер импорта — два независимых прохода ([ingest.py](iracing_coach/ingest.py))

Это ключевой инвариант, который нельзя нарушать:

- **`import_file`** — разбор одного `.ibt`: круги, метрики круга. Зависит
  только от самого файла. Идемпотентен по `file_hash` (blake2b от содержимого).
- **`analyse`** — разметка поворотов трассы, выбор эталонного круга,
  пер-поворотные метрики, подсказки. Зависит от *всей* накопленной базы —
  эталон улучшается с каждой новой сессией.

Смешивать проходы нельзя: первый же импорт разметил бы повороты по случайному
кругу, и все последующие сравнения унаследовали бы эту случайность.
`ensure_turns` при этом никогда не переписывает уже размеченные повороты —
как только пилот (`source='manual'`, см. `PATCH .../turns/{turn_no}`) или прошлый
прогон их зафиксировал, автоматика их не трогает. Повороты размечаются на
**связку трасса+машина** (`turns.car_id`), а не только на трассу: границы и точка
торможения зависят от машины. Поэтому разметка и `_turn_ids` идут внутри цикла по
машинам в `analyse`.

### Хранилище: SQLite + Parquet ([db.py](iracing_coach/db.py), [traces.py](iracing_coach/traces.py))

- **SQLite** (`data/iracing.db`) — только метаданные и метрики: `tracks`,
  `cars`, `sessions`, `laps`, `turns`, `refs`, `findings`. Метрики (`lap_metrics`,
  `turn_metrics`) хранятся в длинном формате ключ-значение, а не колонками —
  набор метрик растёт с каждой идеей, и не хочется гонять миграции схемы под
  каждую.
- **Parquet** (`data/laps/<session_id>/<lap_no>.parquet`) — сама трассировка
  круга, приведённая к сетке из 1000 точек по дистанции. Круг всегда читается
  целиком и колонками, поэтому здесь Parquet эффективнее строк в БД.

### Версия схемы и миграции ([db.py](iracing_coach/db.py))

Форма схемы пронумерована, номер лежит в самом файле базы (`PRAGMA
user_version`). Причина: `CREATE TABLE IF NOT EXISTS` создаёт недостающие
таблицы, но ничего не знает о недостающих *колонках* — база, созданная до
появления `turns.car_id`, молча оставалась старой формы, и запрос к
несуществующей колонке падал уже в API, а фронт показывал это как «нет
эталона».

Правила:

- `SCHEMA` — форма **текущей** версии, применяется только к пустому файлу.
  Существующие базы доводятся до неё шагами из `MIGRATIONS`.
- Новая колонка = новый шаг в `MIGRATIONS` **и** +1 к `SCHEMA_VERSION`. Правка
  одной только `SCHEMA` до существующих баз не доедет; это стережёт тест
  `test_migrations_end_at_schema_version`.
- Снимки формы внутри шага (`_TURNS_V2` и т.п.) скопированы, а не собраны из
  `SCHEMA`: миграция описывает переход между двумя зафиксированными формами, и
  следующая правка `SCHEMA` не должна задним числом менять смысл старого шага.
- Пилотское не пересоздаётся: `turns.source='manual'` и `turn_notes` миграция

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wanabe91/IRacinProject](https://github.com/Wanabe91/IRacinProject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
