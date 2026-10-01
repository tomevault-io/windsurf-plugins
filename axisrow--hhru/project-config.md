---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## О проекте

CLI-инструмент на Playwright для поиска вакансий, откликов и поднятия резюме на hh.ru.
Запускается **только вручную** из терминала. Каждая команда логирует свои действия,
поддерживает `--dry-run` и ограничена дневными лимитами + случайными паузами, чтобы
не выглядеть как подозрительная автоматизация для анти-фрод системы hh.ru. При работе
с этим кодом сохраняй этот принцип: не добавляй фоновые/скрытые режимы и не убирай
троттлинг/лимиты.

**Интерфейс — только CLI, вывод только текст/ASCII-таблицы, без эмодзи.** Эталон набора
команд и формата вывода (сигнатуры, примеры, природа READ/WRITE) — `docs/cli-spec.md`
(дизайн-документ ишью #21); актуальные сигнатуры генерируются в `docs/cli-reference.md`.
Новую команду добавляй по образцу из спеки, её вывод должен
соответствовать зафиксированным там форматам (префиксы `[OK]`/`[INFO]`/`[FAIL]`/`[DRY-RUN]`/`[skip]`,
ASCII-таблицы через `report._ascii_table`).

## Команды

```bash
# Установка
pip3 install -r requirements.txt
python3 -m playwright install chromium

# Настройка (вся папка data/ в .gitignore — коммитить не нужно)
./scripts/run.sh account create default

# Все команды запускаются через обёртку run.sh (вызывает установленный entry point `hhru`)
./scripts/run.sh --account default login                                    # ручной вход, сохраняет сессию
./scripts/run.sh --account default search --resume <id> --dry-run           # поиск без откликов
./scripts/run.sh --account default apply  --resume <id> --dry-run --limit 5 # план откликов
./scripts/run.sh --account default apply  --resume <id> --limit 5           # боевой отклик
./scripts/run.sh --account default bump   --resume <id>                     # поднять резюме (не чаще 1 раза в 4 часа)
./scripts/run.sh --account default run                                       # apply + bump для всех резюме

# Общие флаги: --headless, --verbose, --config <path>, --history <path>, --max-pages <n>
# --resume опционален — без него команда идёт по всем резюме из конфига
# --max-pages по умолчанию адаптивный; --limit apply — целевое число УСПЕШНЫХ откликов (#441)
```

Система сборки — `pip install -e .` (editable install, entry point `hhru`), линтер — `ruff`
(`check` + `format --check`), тесты — `pytest`; все три гоняются в CI (см. «Тестирование и TDD»).

## Архитектура

Поток данных — цепочка ответственностей, не файлов: **сбор вакансий**
(`search_vacancies`, живёт в `search.py`) → **фильтрация/отсев** (`filter_candidates`
+ pre-LLM-префильтр #85 + таблица skipped #87, живёт в `search.py`/`history.py`) →
**планирование** (`run_apply_for_resume` — ранжирование/скоринг #74 + дневной лимит,
живёт в `commands/_common.py`) → **действие** (`apply_to_vacancy` в `apply/pipeline.py`,
`bump_resume` в `bump.py`) → **запись результата** (`history`, живёт в `history.py`). Каждая
ответственность — отдельный модуль с чистыми переиспользуемыми функциями (см. `apply/`
ниже); имена файлов — это «где живёт», а не суть шага. Все браузерные модули
используют **синхронный** Playwright API (`playwright.sync_api`).

### Ключевые архитектурные решения (неочевидны из кода одного файла)

1. **Поиск и фильтрация намеренно разделены.** `search_vacancies()` возвращает ВСЕ
   карточки без применения `exclude_employers`/`exclude_keywords` и без учёта истории.
   Отсев делает отдельная чистая функция `filter_candidates()` — так её причины отказа
   логируются и она тестируема без браузера. Не сливай эти два шага обратно.

2. **Дедупликация откликов не зависит от разметки hh.ru.** «Уже откликались» определяется
   по локальной SQLite-истории (`history.py`), а не по маркеру на странице (анонимному
   запросу hh.ru его не показывает). В `history.py` есть частичный UNIQUE-индекс по
   `(resume_id, vacancy_id)` для `action='apply'` со статусом `success`/`dry_run` —
   `has_applied()` опирается на него. **Важно:** `dry_run`-отклики тоже пишутся в историю
   и считаются «уже откликались», поэтому повторный `--dry-run` по той же вакансии её
   отсеет. `count_last_24h()`/`last_action_at()` для лимитов считают `status='success'` и
   `status='uncertain'` (#176: действие могло выполниться при упавшем посреди клика
   Playwright — fail-closed, `uncertain` тоже дедуплицируется `has_applied()`).

3. **Двухуровневый троттлинг** в `throttle.py`:
   - Дневные лимиты (`daily_apply_limit`, `daily_bump_limit`) — проверяются перед каждым
     действием, при `dry_run` не применяются. Счётчик — скользящее окно 24ч
     (`count_last_24h`, #1142), как реальный лимит hh.ru: календарный сброс в полночь
     позволял CLI обгонять отказ hh.ru в день сброса; окно `[now-24h, now]` — надмножество
     календарного, так что rolling строго консервативнее.
   - Кулдаун поднятия резюме: жёстко `BUMP_COOLDOWN = 4 часа` (`can_bump_now()`), сверх
     дневного лимита.
   - `throttle.wait()` — случайная пауза `min_delay..max_delay` секунд после каждого
     РЕАЛЬНОГО действия на hh.ru — клика поднятия/submit отклика (`BumpResult.acted`/
     `ApplyResult.acted`, #163). Ранние выходы до действия (плейсхолдер в конфиге,
     форма входа, hint «рано», dry-run) не ждут паузу и не пишут `failed` в actions:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axisrow/hhru](https://github.com/axisrow/hhru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
